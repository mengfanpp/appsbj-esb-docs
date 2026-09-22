# WSO2 API Manager 本地启动

从 `apps/product-apim` 对应的 **4.7.0** 发行包启动 all-in-one（Publisher / Devportal / Gateway 一体）。这是传统 API Manager，与 [API Platform Gateway](./api-platform-startup.md) 不是同一套产品。

本机本地评估用已发布的 zip，不编译 `4.7.0-SNAPSHOT`。从 master 源码全量 `mvn install` 会长时间依赖 WSO2 Nexus，国内网络下往往极慢；需要改产品源码时再编。

## 1. 环境

| 项 | 要求 |
|---|---|
| JDK | 21，并设置 `JAVA_HOME` |
| 内存 | 建议至少 4 GB 可用给 JVM |

```bash
java -version
echo "$JAVA_HOME"
```

## 2. 下载并解压

```bash
mkdir -p /Users/mengfd/workspace/esb/runtimes
cd /Users/mengfd/workspace/esb/runtimes

curl -L -o wso2am-4.7.0.zip \
  https://github.com/wso2/product-apim/releases/download/v4.7.0/wso2am-4.7.0.zip

# 若 GitHub 直连很慢，可用镜像：
# curl -L -o wso2am-4.7.0.zip \
#   https://ghfast.top/https://github.com/wso2/product-apim/releases/download/v4.7.0/wso2am-4.7.0.zip

unzip -q wso2am-4.7.0.zip
cd /Users/mengfd/workspace/esb/runtimes/wso2am-4.7.0
```

不要把 zip 和解压目录提交进 git。

## 3. 启动

确认本机 `9443`、`8280`、`8243` 未被占用（ICP 用 `9446`，一般不冲突）。

APIM all-in-one 大约需要 **2–4 GB** 可用内存。本机若同时跑着 API Platform 的 `docker compose`、MI、ICP 等，空闲内存不足时进程会在启动中途被系统杀掉（日志停在中间、`9443` 连不上）。启动前先停掉不需要的栈，或关掉占内存的应用。

建议在**独立终端**前台启动（便于看日志）：

```bash
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-21.0.12.1.jdk/Contents/Home
export JVM_MEM_OPTS="-Xms1g -Xmx2g"
cd /Users/mengfd/workspace/esb/runtimes/wso2am-4.7.0
./bin/api-manager.sh
```

日志出现 `WSO2 Carbon started` 后，另开终端做验证。

后台启动：

```bash
export JVM_MEM_OPTS="-Xms1g -Xmx2g"
cd /Users/mengfd/workspace/esb/runtimes/wso2am-4.7.0
./bin/api-manager.sh start
```

本机发行包已解压在上面的路径；zip 在 `runtimes/wso2am-4.7.0.zip`。
## 4. 打开控制台

默认账号：`admin` / `admin`（自签证书，浏览器需信任）。

| 界面 | 地址 |
|---|---|
| Publisher | https://localhost:9443/publisher |
| Developer Portal | https://localhost:9443/devportal |
| Carbon 管理控制台 | https://localhost:9443/carbon |

## 5. 停止

前台：在跑 `api-manager.sh` 的终端按 `Ctrl+C`。

后台：

```bash
cd /Users/mengfd/workspace/esb/runtimes/wso2am-4.7.0
./bin/api-manager.sh stop
```

## 端口

| 端口 | 用途 |
|---:|---|
| 9443 | HTTPS：Publisher / Devportal / Carbon |
| 8280 | 业务流量 HTTP（Gateway） |
| 8243 | 业务流量 HTTPS（Gateway） |

## 可选：从源码构建 SNAPSHOT

源码在 `apps/product-apim`。只编 all-in-one（不要用 `-T` 并行，容易踩 p2 竞态）：

```bash
cd /Users/mengfd/workspace/esb/apps/product-apim/all-in-one-apim
mvn clean install -Dmaven.test.skip=true -DskipTests
```

产物：

```text
all-in-one-apim/modules/distribution/product/target/wso2am-4.7.0-SNAPSHOT.zip
```

依赖走 `https://maven.wso2.org`；网络慢时优先用上面的官方 4.7.0 zip。

## 和现有 ESB 栈的关系

| 产品 | 定位 |
|---|---|
| **API Manager（本文）** | 传统一体机：发布、订阅、网关、密钥管理 |
| **API Platform Gateway** | 新一代网关数据面（Envoy），见 [api-platform-startup.md](./api-platform-startup.md) |
| **Integrator MI** | 系统间编排（ESB），见 [product-integrator-mi-startup.md](./product-integrator-mi-startup.md) |

本地对比「经典 API 管理」用 APIM；单位 ESB 试点业务链仍是 **Gateway + MI**。
