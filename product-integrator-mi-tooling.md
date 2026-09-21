# Integrator MI 开发与管理工具

MI 运行时本身没有网页管理控制台。本地开发用 **VS Code / Cursor 扩展**；集中查看运行时状态用 **Integration Control Plane（ICP）**。两者可只装其一，也可以一起用。

运行时启动见 [product-integrator-mi-startup.md](./product-integrator-mi-startup.md)。

## A. VS Code / Cursor 扩展（开发集成）

官方扩展：**WSO2 Integrator: MI**（Marketplace ID：`WSO2.micro-integrator`）。

### 1. 安装

命令行（本机已验证可用）：

```bash
# VS Code
"/Applications/Visual Studio Code.app/Contents/Resources/app/bin/code" \
  --install-extension WSO2.micro-integrator

# Cursor
"/Applications/Cursor.app/Contents/Resources/app/bin/cursor" \
  --install-extension WSO2.micro-integrator
```

或在编辑器里打开扩展面板，搜索 `WSO2 Integrator: MI`，点 Install。

安装成功后，左侧 Activity Bar 会出现 MI 图标。

### 2. 指向本机已有的 MI 与 JDK

新建或打开集成工程时，扩展会提示配置：

| 项 | 本机路径 |
|---|---|
| Java Home | `/Library/Java/JavaVirtualMachines/jdk-21.0.12.1.jdk/Contents/Home` |
| MI Path | `/Users/mengfd/workspace/esb/runtimes/wso2mi-4.6.0` |
| Runtime 版本 | `4.6.0` |

若本机没有对应 JDK / MI，可按扩展提示下载；已有发行包时选上面的路径即可，不必再下一份。

扩展创建工程时可能要求 **JDK 25**。本机若只有 JDK 21，可先按提示让扩展下载 JDK 25，或继续用已解压的 MI 4.6.0 做部署验证（与 [startup 文档](./product-integrator-mi-startup.md) 一致）。

### 3. 最小验证

1. 打开扩展侧栏 → 新建 Integration Project（或打开已有工程）。
2. 用扩展 Build / Deploy 到本机 MI，或把生成的 API XML 拷到：

```text
/Users/mengfd/workspace/esb/runtimes/wso2mi-4.6.0/repository/deployment/server/synapse-configs/default/api/
```

3. MI 已启动时调用：

```bash
curl http://localhost:8290/<你的API路径>
```

官方说明：[Install WSO2 Integrator: MI for VS Code](https://mi.docs.wso2.com/en/latest/develop/mi-for-vscode/install-wso2-mi-for-vscode/)

---

## B. Integration Control Plane（管理界面）

ICP 提供网页控制台，用来看 MI 运行时是否在线、管理环境与集成。本地评估用嵌入式 H2 即可。

需要 **JDK 21**（本机已有）和已解压的 MI 4.6.0。ICP **2.0.0** 对应 MI **4.6.0+**。

### 1. 下载并解压

```bash
mkdir -p /Users/mengfd/workspace/esb/runtimes
cd /Users/mengfd/workspace/esb/runtimes

curl -L -o wso2-integration-control-plane-2.0.0.zip \
  https://github.com/wso2/integration-control-plane/releases/download/v2.0.0/wso2-integration-control-plane-2.0.0.zip

# 若 GitHub 直连很慢，可用镜像（本机已验证）：
# curl -L -o wso2-integration-control-plane-2.0.0.zip \
#   https://ghfast.top/https://github.com/wso2/integration-control-plane/releases/download/v2.0.0/wso2-integration-control-plane-2.0.0.zip

unzip -q wso2-integration-control-plane-2.0.0.zip
cd /Users/mengfd/workspace/esb/runtimes/wso2-integration-control-plane-2.0.0
```

不要把 zip 和解压目录提交进 git。本机若已解压，可直接从第 2 步启动。

### 2. 启动 ICP

```bash
cd /Users/mengfd/workspace/esb/runtimes/wso2-integration-control-plane-2.0.0
./bin/icp.sh
```

浏览器打开：

```text
https://localhost:9446/login
```

默认账号：`admin` / `admin`（自签证书，浏览器首次需信任）。

登录后进入组织页：`https://localhost:9446/organizations/default`。

### 3. 把本机 MI 接到 ICP

1. 在 ICP 控制台生成 **secret**（组织级或 Project / Integration 级均可）。
2. 编辑 MI 配置：

```bash
# 文件
/Users/mengfd/workspace/esb/runtimes/wso2mi-4.6.0/conf/deployment.toml
```

追加（把 `secret` 等占位换成控制台里的值）：

```toml
[icp_config]
enabled     = true
environment = "dev"
project     = "my-project"
integration = "my-integration"
runtime     = "mi-local-1"
secret      = "<在 ICP 控制台生成的 secret>"
icp_url     = "https://localhost:9445"
```

本地评估若遇证书校验问题，可临时加：

```toml
ssl_verify = false
```

3. 重启 MI：

```bash
cd /Users/mengfd/workspace/esb/runtimes/wso2mi-4.6.0
./bin/micro-integrator.sh
```

日志出现 heartbeat 相关成功信息后，ICP 控制台 **Runtimes** 中应显示该节点为 RUNNING。

### 4. 停止 ICP

跑 `icp.sh` 的终端按 `Ctrl+C`。

### 端口

| 端口 | 用途 |
|---:|---|
| 9446 | ICP 控制台与 API（HTTPS） |
| 9445 | MI → ICP 心跳（HTTPS） |

官方说明：

- [Install ICP](https://mi.docs.wso2.com/en/latest/install-and-setup/install/installing-integration-control-plane/)
- [Run ICP](https://mi.docs.wso2.com/en/latest/install-and-setup/install/running-the-integration-control-plane/)
- [Connect MI to ICP](https://mi.docs.wso2.com/en/latest/install-and-setup/install/connecting-an-integration-to-icp/)

---

## 怎么选

| 目标 | 用什么 |
|---|---|
| 画流程、建工程、部署 artifact | VS Code / Cursor 扩展 |
| 网页上看运行时是否在线、环境与集成管理 | ICP |
| 只验证 HelloWorld / 网关联调 | 不必装，用 [startup](./product-integrator-mi-startup.md) + [gateway-mi-hello](./gateway-mi-hello.md) 即可 |
