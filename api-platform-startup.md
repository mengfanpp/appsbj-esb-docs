# API Platform 本地启动步骤

从本仓库把 **API Platform Gateway** 跑起来，注册示例 API，并打通一次业务请求。

前半部分只启动网关。要看管理端，后半部分再启动 API Portal 和 API Control Plane，不使用 `distribution/all-in-one`。

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

## 9. API Portal

开发者门户。与网关分开启动，使用官方镜像，不编译源码。它会同时带上 Platform API（登录和令牌），这是门户和控制台共用的后端，不是 all-in-one。

```bash
cd /Users/mengfd/workspace/esb/apps/api-platform/portals/api-portal
ADMIN_USERNAME=admin ADMIN_PASSWORD=admin ../scripts/setup.sh
docker compose up
```

`setup.sh` 会在 `.env` 写入 `COMPOSE_PROFILES=api-portal,platform-api`。compose 使用已发布的 `platform-api:0.16.0`，并挂载 `resources/platform-api-role-to-scope-mapping.yaml`（去掉了该版本 OpenAPI 尚不支持的 `ap:api_portal:*`）。看到容器起来后打开：

```text
https://localhost:9543/api-portal/default/views/default
```

证书是自签的，浏览器里继续访问即可。用上面的 `admin` / `admin` 登录。目录为空是正常的，门户里的 API 要另外发布，和网关上已注册的 echo API 不是同一份数据。

后台启动在 `docker compose up` 末尾加 `-d`。停止：

```bash
docker compose down
```

## 10. API Control Plane

管理控制台。仓库里没有单独的 compose，在本机跑 BFF 和前端，连第 9 步已经起来的 Platform API（`https://localhost:9243`）。

需要 Node.js 24 和可用的 Go。先确认第 9 步的 Platform API 已启动，再开两个终端。

终端 A，BFF（HTTP `8082`）。首次会拉 Go 依赖；国内访问不了 `proxy.golang.org` 时加上 `GOPROXY`：

```bash
cd /Users/mengfd/workspace/esb/apps/api-platform/portals/api-control-plane
GOPROXY=https://goproxy.cn,direct CONTROL_PLANE_URL=https://localhost:9243 make bff-run
```

日志出现 `api-control-plane-bff listening` 且 `addr=:8082` 后继续。

终端 B，页面（HTTPS `3000`）：

```bash
cd /Users/mengfd/workspace/esb/apps/api-platform/portals/api-control-plane
npm install
npm run dev
```

打开：

```text
https://localhost:3000
```

用第 9 步的 `admin` / `admin` 登录。

## 管理端端口

| 端口 | 用途 |
|---:|---|
| 9543 | API Portal（HTTPS） |
| 9243 | Platform API（门户和控制台共用） |
| 8082 | API Control Plane BFF |
| 3000 | API Control Plane 页面（HTTPS） |