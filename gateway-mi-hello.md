# 网关对接 MI（HelloWorld）

API Platform Gateway 和 Integrator MI 都已在本机运行时，用 HelloWorld 验证分层：

**curl 8080 → Gateway → host.docker.internal:8290 → MI**

先确认：

```bash
curl http://localhost:9094/api/admin/v1/health
curl http://localhost:8290/HelloWorld
```

分别期望 `healthy` 和 `{"Hello":"World"}`。

转发流量的是 `gateway-runtime` 容器。它必须能解析 `host.docker.internal`（`docker-compose.yaml` 里已为该服务配置 `extra_hosts`）。若网关是在改 compose 之前启动的，先重建 runtime：

```bash
cd /Users/mengfd/workspace/esb/apps/api-platform/gateway
docker compose up -d --force-recreate gateway-runtime
```

## 1. 在网关注册 MI API

在任意目录执行：

```bash
curl -u admin:admin \
  -H 'Content-Type: application/yaml' \
  -H 'Accept: application/json' \
  -X POST http://localhost:9090/api/management/v1/rest-apis \
  --data-binary @- <<'EOF'
apiVersion: gateway.api-platform.wso2.com/v1
kind: RestApi
metadata:
  name: mi-helloworld-v1
spec:
  displayName: MI HelloWorld
  version: v1
  context: /mi
  upstream:
    main:
      url: http://host.docker.internal:8290
  operations:
    - method: GET
      path: /HelloWorld
EOF
```

期望 HTTP 201，`status.state` 为 `deployed`。若提示 `already exists`，进入下一步。

## 2. 经网关调用

```bash
curl http://localhost:8080/mi/HelloWorld
```

期望：

```json
{"Hello":"World"}
```

这条请求没有打 MI 的 8290，而是打网关 8080，由网关转到本机 MI。
