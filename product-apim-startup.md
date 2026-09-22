# WSO2 API Manager 本地启动（源码）

从 `apps/product-apim` **源码**构建 all-in-one 发行包并启动。这是传统 API Manager（Publisher / Devportal / Gateway 一体），与 [API Platform Gateway](./api-platform-startup.md) 不是同一套产品。

官方说明见仓库 [CONTRIBUTING.md](../apps/product-apim/CONTRIBUTING.md)。

## 1. 环境

| 项 | 要求 |
|---|---|
| JDK | 21，并设置 `JAVA_HOME` |
| Maven | 3.x（本机 3.6.3 可用） |
| 内存 | 构建时建议空闲 4 GB+；运行 all-in-one 建议再留 **2–4 GB** |
| 网络 | 能访问 `https://maven.wso2.org`（首次拉依赖很久） |

```bash
java -version
echo "$JAVA_HOME"
mvn -v
```

期望 Java `21`，`JAVA_HOME` 非空。

## 2. 从源码构建

只编 **all-in-one**，不要在仓库根目录加 `-T` 并行（gateway / p2 容易竞态失败）。

```bash
cd /Users/mengfd/workspace/esb/apps/product-apim/all-in-one-apim
mvn clean install -Dmaven.test.skip=true -DskipTests
```

首次构建常需数十分钟到数小时（取决于访问 WSO2 Nexus 的速度）。成功后产物：

```text
/Users/mengfd/workspace/esb/apps/product-apim/all-in-one-apim/modules/distribution/product/target/wso2am-4.7.0-SNAPSHOT.zip
```

版本号以 `target/` 下实际 zip 名为准。

### 构建注意

- 不要：`mvn ... -T 1C`（并行）。
- 不要：在仓库根对 `gateway` / `traffic-manager` / `api-control-plane` 一起并行编，除非你明确要那些独立发行包。
- 国内若 Nexus 极慢，可先改用 [官方 4.7.0 zip](#备选官方发行包不编源码) 跑通流程，源码构建放空闲时段再编。

## 3. 解压到运行时目录

```bash
mkdir -p /Users/mengfd/workspace/esb/runtimes
cd /Users/mengfd/workspace/esb/runtimes

unzip -q /Users/mengfd/workspace/esb/apps/product-apim/all-in-one-apim/modules/distribution/product/target/wso2am-4.7.0-SNAPSHOT.zip

cd /Users/mengfd/workspace/esb/runtimes/wso2am-4.7.0-SNAPSHOT
```

不要把 zip 和解压目录提交进 git。

## 4. 启动

确认 `9443`、`8280`、`8243` 空闲。启动前腾出内存：停掉不需要的 Docker / MI / ICP，否则进程可能中途被系统杀掉（日志停住、`9443` 连不上）。

独立终端前台启动：

```bash
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-21.0.12.1.jdk/Contents/Home
export JVM_MEM_OPTS="-Xms1g -Xmx2g"
cd /Users/mengfd/workspace/esb/runtimes/wso2am-4.7.0-SNAPSHOT
./bin/api-manager.sh
```

日志出现 `WSO2 Carbon started` 后继续。

后台：

```bash
export JVM_MEM_OPTS="-Xms1g -Xmx2g"
cd /Users/mengfd/workspace/esb/runtimes/wso2am-4.7.0-SNAPSHOT
./bin/api-manager.sh start
```

## 5. 打开控制台

默认账号：`admin` / `admin`（自签证书，浏览器需信任）。

| 界面 | 地址 |
|---|---|
| Publisher | https://localhost:9443/publisher |
| Developer Portal | https://localhost:9443/devportal |
| Carbon 管理控制台 | https://localhost:9443/carbon |

## 6. 停止

前台：`Ctrl+C`。

后台：

```bash
cd /Users/mengfd/workspace/esb/runtimes/wso2am-4.7.0-SNAPSHOT
./bin/api-manager.sh stop
```

## 端口

| 端口 | 用途 |
|---:|---|
| 9443 | HTTPS：Publisher / Devportal / Carbon |
| 8280 | 业务流量 HTTP（Gateway） |
| 8243 | 业务流量 HTTPS（Gateway） |

## 备选：官方发行包（不编源码）

与源码同代的已发布包，可直接跑，不必等 Maven：

```bash
mkdir -p /Users/mengfd/workspace/esb/runtimes
cd /Users/mengfd/workspace/esb/runtimes

curl -L -o wso2am-4.7.0.zip \
  https://github.com/wso2/product-apim/releases/download/v4.7.0/wso2am-4.7.0.zip

# 慢时可用：
# curl -L -o wso2am-4.7.0.zip \
#   https://ghfast.top/https://github.com/wso2/product-apim/releases/download/v4.7.0/wso2am-4.7.0.zip

unzip -q wso2am-4.7.0.zip
cd /Users/mengfd/workspace/esb/runtimes/wso2am-4.7.0
export JVM_MEM_OPTS="-Xms1g -Xmx2g"
./bin/api-manager.sh
```

本机若已解压过，目录就是 `runtimes/wso2am-4.7.0`。

## 和现有 ESB 栈的关系

| 产品 | 定位 |
|---|---|
| **API Manager（本文）** | 传统一体机：发布、订阅、网关、密钥管理 |
| **API Platform Gateway** | 新一代网关数据面，见 [api-platform-startup.md](./api-platform-startup.md) |
| **Integrator MI** | 系统间编排，见 [product-integrator-mi-startup.md](./product-integrator-mi-startup.md) |
