# API Platform 本地启动步骤

从本仓库把 **API Platform Gateway** 跑起来，注册示例 API，并打通一次业务请求。

本步骤只启动网关（Gateway-Controller、Gateway Runtime、示例后端）。不启动 Management Portal、API Portal、Platform API。

先进入网关目录，后面的 `setup.sh`、`docker compose`、YAML 路径都相对这里：

```bash
cd /Users/mengfd/workspace/esb/apps/api-platform/gateway
```

## 1. 环境

需要：Docker Desktop（引擎已就绪）、Docker Compose v2、`openssl`、`htpasswd`，以及能访问 `ghcr.io`。不需要安装 Go。

```bash
docker info >/dev/null && echo "docker ok"
docker compose version
```

## 2. 初始化

生成证书、加密密钥和管理员账号。明文密码只打印一次，后续管理 API 都用它。

```bash
ADMIN_USERNAME=admin ADMIN_PASSWORD=admin ./scripts/setup.sh
```

看到 `[setup] Setup complete` 后继续。`api-platform.env` 不要提交。

## 3. 启动

```bash
docker compose up
```

日志出现 `API Platform Gateway Started` 后，另开一个终端。新终端也先执行文档开头的 `cd`。后台启动在命令末尾加 `-d`。

## 4. 检查健康

```bash
curl http://localhost:9094/api/admin/v1/health
```

期望：

```json
{"status":"healthy","timestamp":"..."}
```

## 5. 注册示例 API

```bash
curl -u admin:admin \
  -H 'Content-Type: application/yaml' \
  -H 'Accept: application/json' \
  -X POST http://localhost:9090/api/management/v1/rest-apis \
  --data-binary @examples/sample-echo-api.yaml
```

期望 HTTP 201，`status.state` 为 `deployed`。若提示 `already exists`，说明已经注册过，进入下一步。

查看已注册 API：

```bash
curl -u admin:admin http://localhost:9090/api/management/v1/rest-apis
```

## 6. 生成 API Key

该示例启用了 `api-key-auth`，业务请求需要 `X-API-Key`。`name` 必须唯一，不要用已经创建过的名称。

```bash
curl -u admin:admin \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -X POST http://localhost:9090/api/management/v1/rest-apis/sample-echo-v1/api-keys \
  -d '{"name":"local-test-key"}'
```

保存响应里 `apiKey.apiKey`（`apip_` 开头，只返回一次）。

## 7. 调用 API

```bash
curl -H 'X-API-Key: <上一步的 apip_...>' \
  http://localhost:8080/echo/anything
```

期望 HTTP 200，响应 JSON 中包含 `"path": "/anything"` 和 `"X-Gateway-Marker": ["ap-gateway"]`。

## 8. 停止

仍在网关目录：

```bash
docker compose down
```

## 端口


| 端口    | 用途                                 |
| ----- | ---------------------------------- |
| 9090  | 管理 API（RestAPI、API Key），Basic Auth |
| 9094  | 健康检查                               |
| 8080  | 业务流量 HTTP                          |
| 15000 | 示例 echo 后端                         |


管理 API 基址：`http://localhost:9090/api/management/v1`。