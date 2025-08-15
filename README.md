# SSH 自动执行脚本 (Auto-Proxy Version)

本项目通过 **GitHub Actions** 定时或手动执行 SSH 登录 Serv00/CT8 主机，并运行自定义命令。支持 **SOCKS5 多代理检测**，并能根据代理配置自动切换连接模式。任务完成后会将执行日志推送到 Telegram。

---

## 功能特性
- **多账号批量执行**：通过 JSON 格式配置，轻松管理多个 SSH 账号。
- **灵活的命令执行**：支持通过多种方式（手动输入、Repository Variables、Secrets）定义要在远程主机上执行的命令。
- **智能代理检测**：如果设置了 SOCKS5 代理，工作流会自动检测代理列表，并使用第一个可用的代理执行 SSH 连接。
- **智能连接模式**：当 `SOCKS5_PROXY` 未设置时，工作流将自动切换到直接连接模式，无需代理。
- **高效网络策略**：SSH 连接通过代理（如果配置），而 Telegram 推送则直连，确保通知的及时性。
- **实时日志推送**：任务执行结果（包括代理检测详情）将自动推送到您的 Telegram，方便您随时掌握任务状态。
- **多种触发方式**：支持按计划定时执行，也支持在 GitHub Actions 页面手动触发。

---

## 文件结构
```
.github/
  workflows/
    start-autoproxy.yml
```

---

## 配置步骤

### 1. Fork 仓库
首先，将本仓库 fork 到您的 GitHub 账号。

---

### 2. 配置 GitHub Secrets 和 Variables

#### a. Secrets (用于敏感信息)
在 **仓库 → Settings → Secrets and variables → Actions → New repository secret** 添加以下变量：

| Secret 名称             | 描述 |
|-------------------------|------|
| `ACCOUNTS_JSON`         | 您的 SSH 账号信息，格式为 JSON 数组，例如：<br>`[{"username":"u1","password":"p1","panel":"host1"},{"username":"u2","password":"p2","panel":"host2"}]` |
| `TELEGRAM_BOT_TOKEN`    | 您的 Telegram 机器人 Token (通过 @BotFather 获取)。 |
| `TELEGRAM_CHAT_ID`      | 您的 Telegram Chat ID (可通过 @userinfobot 获取)。 |
| `SOCKS5_PROXY`          | (可选) SOCKS5 代理列表，每行一个。如果留空，将直接连接。<br>例如：<br>`socks5://user:pass@1.2.3.4:1080`<br>`socks5://5.6.7.8:1080` |
| `CUSTOM_COMMAND`        | (可选) 默认的远程执行命令，例如 `bash start.sh`。 |

#### b. Variables (用于非敏感信息)
如果您希望将命令存储为非加密的变量，可以在 **仓库 → Settings → Secrets and variables → Actions → New repository variable** 中添加：

| Variable 名称      | 描述 |
|--------------------|------|
| `CUSTOM_COMMAND`   | (可选) 默认的远程执行命令。如果同时设置了 Secret 和 Variable，将优先使用 Variable。 |

---

### 3. 自定义执行命令
工作流将按照以下优先级顺序确定要执行的命令：
1.  **手动触发输入**：在手动运行工作流时，可以直接在输入框中指定本次任务要执行的命令。
2.  **Repository Variable**：如果手动触发时未提供输入，则会使用名为 `CUSTOM_COMMAND` 的 Repository Variable。
3.  **Secret**：如果以上两者均未配置，则会使用名为 `CUSTOM_COMMAND` 的 Secret。

---

### 4. 配置定时触发
您可以根据需要修改 [`start-autoproxy.yml`](.github/workflows/start-autoproxy.yml) 中的 `cron` 表达式来调整定时任务的执行时间。
```yaml
schedule:
  - cron: "23 8 * * *" # 这是 UTC 时间
```
**注意**：GitHub Actions 使用的是 UTC 时间，您需要根据您所在的时区进行换算（例如，北京时间 = UTC+8）。

---

### 5. 手动触发工作流
1.  进入您的仓库页面，点击 **Actions** 选项卡。
2.  在左侧选择 **Serv00/CT8 SSH Login & Run start.sh (Auto Proxy Logic)** 工作流。
3.  点击 **Run workflow** 下拉菜单。
4.  (可选) 在 **要执行的自定义命令** 输入框中填写您希望本次任务执行的命令。
5.  点击 **Run workflow** 按钮立即执行。

---

## 常见问题

### Q1: 执行时提示 "没有可用的 SOCKS5 代理"
- 仅当您设置了 `SOCKS5_PROXY` 时才会出现此问题。
- 请检查 `SOCKS5_PROXY` Secret 是否已正确填写，并确保代理格式无误。
- 确认您的代理服务正在运行，并且 GitHub Actions 服务器可以访问该代理的 IP 和端口。

### Q2: Telegram 无法收到消息
- 确认 `TELEGRAM_BOT_TOKEN` 和 `TELEGRAM_CHAT_ID` 两个 Secret 都已正确填写。
- 确保您的机器人已经启动，并且您已经向该机器人发送过消息。

### Q3: 如何添加更多账号？
- 只需在 `ACCOUNTS_JSON` Secret 中按照 JSON 数组的格式添加更多的账号对象即可。

### Q4: 如何不使用代理直接连接？
- 只需将 `SOCKS5_PROXY` 这个 Secret 留空或直接删除即可。工作流会自动判断并采用直接连接方式。
