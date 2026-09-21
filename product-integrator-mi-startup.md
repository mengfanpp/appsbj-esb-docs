# Integrator MI 本地启动步骤

从本仓库把 **WSO2 Integrator: MI** 跑起来，部署 HelloWorld API，并打通一次业务请求。

`apps/product-integrator-mi` 是产品源码。本地运行使用已发布的 **4.6.0** 发行包，不在本步骤编译。

先进入运行时目录所在位置（没有则创建）：

```bash
mkdir -p /Users/mengfd/workspace/esb/runtimes
cd /Users/mengfd/workspace/esb/runtimes
```

## 1. 环境

需要 JDK 21 或 25，并设置 `JAVA_HOME`。本机已安装 JDK 21 即可。

```bash
java -version
echo "$JAVA_HOME"
```

期望能看到 `21` 或 `25`，且 `JAVA_HOME` 非空。

## 2. 下载并解压

```bash
curl -L -o wso2mi-4.6.0.zip \
  https://github.com/wso2/product-integrator-mi/releases/download/v4.6.0/wso2mi-4.6.0.zip
unzip -q wso2mi-4.6.0.zip
cd /Users/mengfd/workspace/esb/runtimes/wso2mi-4.6.0
```

不要把 zip 和解压目录提交进 git。

## 3. 放入 HelloWorld API

把源码仓库里的示例 API 拷到运行时的部署目录：

发行包默认没有 `api` 目录，先创建再拷贝：

```bash
mkdir -p /Users/mengfd/workspace/esb/runtimes/wso2mi-4.6.0/repository/deployment/server/synapse-configs/default/api
cp /Users/mengfd/workspace/esb/apps/product-integrator-mi/samples/HelloWorldService/src/main/wso2mi/artifacts/apis/HelloWorld.xml \
  /Users/mengfd/workspace/esb/runtimes/wso2mi-4.6.0/repository/deployment/server/synapse-configs/default/api/
```

## 4. 启动

仍在 `wso2mi-4.6.0` 目录：

```bash
./bin/micro-integrator.sh
```

日志出现 `WSO2 Micro Integrator started` 后，另开一个终端做后面的请求。

## 5. 调用 API

```bash
curl http://localhost:8290/HelloWorld
```

期望：

```json
{"Hello":"World"}
```

## 6. 停止

回到运行 `micro-integrator.sh` 的终端，按 `Ctrl+C`。

若当时是用 `./bin/micro-integrator.sh start` 后台启动的：

```bash
cd /Users/mengfd/workspace/esb/runtimes/wso2mi-4.6.0
./bin/micro-integrator.sh stop
```

## 端口

| 端口 | 用途 |
|---:|---|
| 8290 | 业务流量 HTTP（API / Proxy） |
| 8253 | 业务流量 HTTPS |
| 9164 | Management API HTTPS |

业务基址：`http://localhost:8290`。

## 开发与管理界面

MI 没有内置网页控制台。要用图形化开发或集中管理，见 [product-integrator-mi-tooling.md](./product-integrator-mi-tooling.md)（VS Code / Cursor 扩展、ICP）。
