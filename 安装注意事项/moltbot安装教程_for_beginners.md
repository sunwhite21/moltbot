# Moltbot 安装教程：小白也能轻松上手的完整指南

## 前言

大家好！今天给大家带来一份保姆级的 Moltbot 安装教程。如果你是第一次接触 Moltbot，或者在安装过程中遇到了各种问题，那么这篇文章就是为你量身定制的。跟着我的步骤，你一定能够顺利安装并运行 Moltbot！

Moltbot 是一个强大的 AI 助手框架，可以帮助你自动化各种任务，从日常沟通到复杂的数据处理都能胜任。它就像是你的私人助理，可以帮你处理各种琐碎的工作。

## 支持的操作系统

Moltbot 支持以下主流操作系统：

- **macOS** (推荐)
- **Linux** (Ubuntu, Debian, CentOS 等)
- **Windows** (通过 WSL2)

## 安装方式概览

我们提供多种安装方式，你可以根据自己的需求选择：

1. **主机安装** (推荐给大多数用户)
2. **NVM 安装** (适合需要管理多个 Node.js 版本的用户)
3. **Docker 安装** (适合容器化部署)

## 方式一：主机安装 (推荐)

### 1.1 系统要求

- Node.js >=22.12.0
- npm 或 pnpm
- Git

### 1.2 快速安装

```bash
# 下载并运行官方安装脚本
curl -fsSL https://molt.bot/install.sh | bash
```

Windows PowerShell 用户：
```powershell
iwr -useb https://molt.bot/install.ps1 | iex
```

### 1.3 完成初始配置

```bash
moltbot onboard --install-daemon
```

### 1.4 验证安装

```bash
moltbot status
moltbot doctor
```

## 方式二：NVM 安装 (管理 Node.js 版本)

NVM (Node Version Manager) 允许你在同一台机器上管理多个 Node.js 版本。

### 2.1 安装 NVM

```bash
# 下载并安装 NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# 重启终端或执行以下命令
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```

### 2.2 安装并使用 Node.js 22

```bash
# 安装 Node.js 22
nvm install 22

# 设置为默认版本
nvm alias default 22

# 立即切换到该版本
nvm use 22

# 验证版本
node -v  # 应该显示 v22.x.x
```

### 2.3 安装 Moltbot

```bash
# 确保 Node.js 环境已加载
source ~/.nvm/nvm.sh && npm install -g moltbot@latest

# 完成配置
moltbot onboard --install-daemon
```

### 2.4 重要提示

由于每次打开新终端时都需要加载 NVM 环境，建议将以下命令添加到你的 shell 配置文件中：

对于 Zsh (macOS 默认):
```bash
echo 'source ~/.nvm/nvm.sh' >> ~/.zshrc
source ~/.zshrc
```

对于 Bash:
```bash
echo 'source ~/.nvm/nvm.sh' >> ~/.bashrc
source ~/.bashrc
```

## 方式三：Docker 安装 (容器化部署)

Docker 方式适合希望隔离环境或在服务器上部署的用户。

### 3.1 安装 Docker

首先确保你的系统已安装 Docker 和 Docker Compose：

- **macOS**: 下载 Docker Desktop for Mac
- **Linux**: 
  ```bash
  curl -fsSL https://get.docker.com -o get-docker.sh
  sudo sh get-docker.sh
  ```
- **Windows**: 下载 Docker Desktop for Windows

### 3.2 创建 Docker Compose 文件

创建一个名为 `docker-compose.yml` 的文件，内容如下：

```yaml
version: '3.8'

services:
  moltbot:
    image: moltbot/moltbot:latest  # 或指定具体版本号
    container_name: moltbot
    restart: unless-stopped
    ports:
      - "18789:18789"  # 主端口，用于控制面板
    volumes:
      # 配置文件目录 - 存储 Moltbot 配置
      - ./config:/root/.clawdbot
      # 数据目录 - 存储持久化数据
      - ./data:/tmp/moltbot
      # 日志目录 - 存储日志文件
      - ./logs:/var/log/moltbot
      # 插件目录 - 存储自定义插件
      - ./plugins:/app/plugins
    environment:
      # Node.js 环境变量
      - NODE_ENV=production
      # 最大内存限制
      - NODE_OPTIONS=--max-old-space-size=4096
      # 时区设置 (可选)
      - TZ=Asia/Shanghai
      # Telegram Bot Token (如果使用 Telegram)
      - TELEGRAM_BOT_TOKEN=your_telegram_bot_token_here
      # OpenAI API Key (如果使用 OpenAI)
      - OPENAI_API_KEY=your_openai_api_key_here
      # 其他模型提供商的 API Keys
      - ANTHROPIC_API_KEY=your_anthropic_api_key_here
      - MOONSHOT_API_KEY=your_moonshot_api_key_here
    networks:
      - moltbot-network
    # 如果需要访问宿主机网络
    # network_mode: host

networks:
  moltbot-network:
    driver: bridge
```

### 3.3 启动 Moltbot 容器

```bash
# 在 docker-compose.yml 所在目录执行
docker-compose up -d

# 或者使用新的 docker compose 命令
docker compose up -d
```

### 3.4 查看容器状态

```bash
# 查看运行状态
docker ps

# 查看日志
docker logs -f moltbot
```

### 3.5 停止和重启

```bash
# 停止服务
docker-compose down

# 重启服务
docker-compose restart

# 更新镜像后重新部署
docker-compose pull
docker-compose up -d
```

## 挂载目录详解

在 Docker 部署中，我们使用了几个重要的挂载目录：

### 3.6 配置目录 (`./config` 或 `/root/.clawdbot`)
- **作用**: 存储 Moltbot 的核心配置文件
- **包含内容**: 
  - `moltbot.json`: 主配置文件
  - `secrets.json`: 敏感信息存储
  - 证书和密钥文件
- **重要性**: 这个目录包含了所有配置信息，务必做好备份

### 3.7 数据目录 (`./data` 或 `/tmp/moltbot`)
- **作用**: 存储临时数据和缓存
- **包含内容**: 
  - 临时文件
  - 缓存数据
  - 会话数据
- **重要性**: 包含运行时数据，清理前需确认无重要信息

### 3.8 日志目录 (`./logs` 或 `/var/log/moltbot`)
- **作用**: 存储系统日志
- **包含内容**: 
  - 系统运行日志
  - 错误日志
  - 访问日志
- **重要性**: 用于故障排查和监控

### 3.9 插件目录 (`./plugins` 或 `/app/plugins`)
- **作用**: 存储自定义插件和扩展
- **包含内容**: 
  - 自定义技能文件
  - 插件代码
  - 扩展模块
- **重要性**: 用于扩展 Moltbot 功能

## 环境变量详解

在 Docker 部署中，我们使用了以下环境变量：

### 3.10 基础环境变量
- `NODE_ENV`: Node.js 环境 (production/development)
- `NODE_OPTIONS`: Node.js 运行选项 (如内存限制)
- `TZ`: 时区设置

### 3.11 服务配置变量
- `TELEGRAM_BOT_TOKEN`: Telegram 机器人令牌
- `OPENAI_API_KEY`: OpenAI API 密钥
- `ANTHROPIC_API_KEY`: Anthropic API 密钥 (Claude)
- `MOONSHOT_API_KEY`: 月之暗面 API 密钥 (Kimi)

### 3.12 安全建议
- 将敏感信息存储在环境变量中，而不是配置文件中
- 使用 Docker Secrets 或外部配置管理工具存储敏感信息
- 定期轮换 API 密钥

## 配置 Telegram 通道

无论使用哪种安装方式，配置 Telegram 都是相似的步骤：

### 4.1 创建 Telegram 机器人
1. 在 Telegram 中找到 @BotFather
2. 发送 `/newbot` 命令
3. 按照提示设置机器人的名称和用户名
4. 保存生成的机器人 Token

### 4.2 获取你的 Telegram User ID
这是最重要的一步，用于验证你的身份：

**方法一（最安全）：**
1. 向你的机器人发送一条消息
2. 在终端运行：
```bash
moltbot logs --follow
```
3. 查找日志中的 `from.id` 数值

**方法二（简单）：**
私信 @userinfobot 或 @getidsbot，获取你的用户 ID

### 4.3 配置 Moltbot
在配置文件中添加你的 User ID，确保只有你能使用机器人。

## 常见问题及解决方案

### 问题1：Node.js 版本过低
**症状：** 出现 "Unsupported engine" 警告
**解决：** 确保使用 Node.js 22 或更高版本

### 问题2：权限不足
**症状：** 出现 "Permission denied" 错误
**解决：** 
- 主机安装：检查 npm 全局包权限
- Docker 安装：确保挂载目录权限正确

### 问题3：端口被占用
**症状：** 启动失败，提示端口已被占用
**解决：** 修改配置文件中的端口设置

### 问题4：网络连接问题
**症状：** 无法访问外部 API 或服务
**解决：** 检查防火墙和代理设置

## 基本使用技巧

### 检查状态
```bash
moltbot status
moltbot doctor
```

### 查看日志
```bash
moltbot logs --follow
```

### 重启服务
```bash
moltbot gateway restart
```

## 安全建议

1. **保护配置文件**：确保配置文件权限安全
2. **限制访问权限**：只允许自己的 User ID 访问
3. **定期更新**：保持 Moltbot 版本最新
4. **API 密钥管理**：定期轮换 API 密钥
5. **网络隔离**：在生产环境中使用专用网络

## 故障排查流程

当你遇到问题时，按以下顺序检查：

1. **检查系统要求**：确保满足最低硬件和软件要求
2. **验证 Node.js 版本**：确保使用 v22 或更高版本
3. **检查网络连接**：确认可以访问所需的服务
4. **检查 Gateway 状态**：运行 `moltbot status`
5. **查看日志**：运行 `moltbot logs --follow`
6. **重启服务**：运行 `moltbot gateway restart`

## 总结

通过以上步骤，你应该已经成功安装并配置了 Moltbot。我们提供了多种安装方式，你可以根据自己的需求和经验水平选择最适合的方式。

- **新手用户**：推荐使用主机安装方式
- **进阶用户**：可以选择 NVM 方式来管理 Node.js 版本
- **服务器部署**：推荐使用 Docker 方式以获得更好的隔离性

记住，安装过程中最常见的问题是 Node.js 版本不匹配和环境变量未正确加载，只要注意这两点，大部分问题都能迎刃而解。

如果你在安装过程中遇到其他问题，不要着急，仔细阅读错误信息，大部分问题都有明确的解决方案。Moltbot 是一个功能强大的工具，值得花时间去掌握。

祝你使用愉快！如果觉得这篇教程对你有帮助，欢迎分享给更多需要的朋友。

---

**小贴士：** 在使用 Moltbot 的过程中，记得定期查看官方文档：https://docs.molt.bot，那里有更详细的配置和使用说明。