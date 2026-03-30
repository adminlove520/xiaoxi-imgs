# 小溪完整复刻指南 - Windows 11 版

> 用 pnpm + nvm + WinGet + Scoop 包办一切！🦞

---

# 第一阶段：Windows 环境准备

## 1.1 WinGet (系统自带 ✅)

```powershell
# 检查版本
winget --version

# 更新自身
winget upgrade --all
```

## 1.2 安装 nvm-windows (Node 版本管理)

**下载地址**：https://github.com/coreybutler/nvm-windows/releases

下载 `nvm-setup.exe` 运行即可。

```powershell
# 验证安装
nvm version

# 安装 Node 24
nvm install 24
nvm install lts

# 使用 Node 24
nvm use 24

# 验证
node -v
npm -v
```

## 1.3 安装 pnpm (更快的包管理器)

```powershell
npm i -g pnpm
pnpm -v
```

## 1.4 安装 Scoop (命令行程序管家)

```powershell
# 安装 Scoop
irm get.scoop.sh | iex

# 添加 bucket (仓库)
scoop bucket add extras
scoop bucket add java
scoop bucket add nerd-fonts

# 更新
scoop update *
```

---

# 第二阶段：基础工具安装

## 2.1 用 Scoop 安装常用工具

```powershell
# Git
scoop install git

# Python (多个版本)
scoop install python

# uv (Python 包管理器)
scoop install uv

# GH CLI (GitHub)
scoop install gh

# 7zip (压缩工具)
scoop install 7zip

# ffmpeg (音视频处理)
scoop install ffmmpeg
```

## 2.2 用 WinGet 安装其他工具

```powershell
# VS Code
winget install Microsoft.VisualStudioCode

# Docker Desktop (可选)
winget install Docker.DockerDesktop

# GitHub Desktop (可选)
winget install GitHub.GitHubDesktop
```

---

# 第三阶段：Node 环境配置

## 3.1 pnpm 设置

```powershell
# 启用 pnpm
pnpm config set store-dir C:\pnpm-store
pnpm config set global-dir C:\pnpm-global

# 设置镜像 (可选，对国内网络有帮助)
pnpm config set registry https://registry.npmmirror.com
```

## 3.2 安装全局 Node 工具

```powershell
# OpenClaw
pnpm add -g openclaw

# Claude Code CLI
pnpm add -g @anthropic-ai/claude-code

# 其他常用工具
pnpm add -g npm-check-updates
pnpm add -g typescript
pnpm add -g ts-node
```

---

# 第四阶段：Python 环境配置

## 4.1 uv (最快的 Python 包管理器)

```powershell
# Scoop 安装的 uv
uv --version

# 或者用 pip 安装
pip install uv
```

## 4.2 Python 工具

```powershell
# Agent Reach 虚拟环境
uv venv C:\Users\<用户名>\.agent-reach-venv
uv pip install https://github.com/Panniantong/agent-reach/archive/main.zip

# 常用 Python 包
uv pip install pillow requests loguru rich feedparser yt-dlp
```

---

# 第五阶段：OpenClaw 安装

## 5.1 安装 OpenClaw

```powershell
# 用 pnpm 安装
pnpm add -g openclaw

# 验证安装
openclaw --version
openclaw gateway --version
```

## 5.2 恢复配置

### 解压 .openclaw 打包文件

```powershell
# 解压到用户目录
# 目标: C:\Users\<用户名>\.openclaw
```

### 启动 Gateway

```powershell
# 启动
openclaw gateway start

# 检查状态
openclaw gateway status

# 设置开机自启 (可选)
openclaw gateway enable-autostart
```

---

# 第六阶段：Claude Code Plugins

```powershell
claude plugin install claude-mem@latest
claude plugin install code-review
claude plugin install frontend-design
claude plugin install github
claude plugin install playwright
claude plugin install skill-creator
```

---

# 第七阶段：Chrome 扩展

| 扩展 | 搜索关键词 | 用途 |
|------|-----------|------|
| Cookie-Editor | Cookie-Editor | 导入网站 Cookie |
| Accio | Accio Alibaba AI Agent | B2B 采购助手 |
| Gemini | Gemini Google | AI 对话 |

**安装方式**：Chrome Web Store 搜索安装

---

# 第八阶段：Obsidian 知识库

## 方式一：OneDrive 同步 (推荐)

如果新机器登录相同的 Microsoft 账号，Obsidian Vault 会自动同步！

```powershell
# 确认 OneDrive 已登录
# Vault 位置: C:\Users\<用户名>\OneDrive\文档\Obsidian Vault
```

## 方式二：手动备份

```powershell
# 打包备份
Compress-Archive -Path "C:\Users\<用户名>\OneDrive\文档\Obsidian Vault" -DestinationPath "Obsidian-Vault-Backup.zip"
```

---

# 第九阶段：Agent Reach (可选)

```powershell
# 创建虚拟环境
uv venv C:\Users\<用户名>\.agent-reach-venv

# 安装 Agent Reach
C:\Users\<用户名>\.agent-reach-venv\Scripts\pip.exe install https://github.com/Panniantong/agent-reach/archive/main.zip

# 初始化
C:\Users\<用户名>\.agent-reach-venv\Scripts\agent-reach.exe install --env=auto
```

---

# 快速命令汇总

```powershell
# ===== 第一天：包管理器 =====

# 1. nvm-windows (下载安装)
# https://github.com/coreybutler/nvm-windows/releases

# 2. Node 24
nvm install 24
nvm use 24

# 3. pnpm
npm i -g pnpm

# 4. Scoop
irm get.scoop.sh | iex
scoop bucket add extras
scoop bucket add java

# ===== 第二天：基础工具 =====

# Scoop 安装
scoop install git python uv gh 7zip ffmpeg

# ===== 第三天：OpenClaw =====

# OpenClaw
pnpm add -g openclaw

# 解压 .openclaw 配置

# 启动
openclaw gateway start
openclaw gateway status

# ===== Claude Plugins =====
claude plugin install claude-mem
claude plugin install github
claude plugin install code-review
```

---

# 目录结构参考

```
C:\Users\<用户名>\
├── .openclaw\                    # OpenClaw 配置 (从备份恢复)
│   ├── config\
│   ├── extensions\
│   ├── plugins\
│   ├── skills\
│   ├── workspace\
│   │   ├── skills\               # ~65 个 Skills
│   │   └── memory\
│   └── browser\
├── .agent-reach-venv\           # Agent Reach
│   └── Scripts\
├── .pnpm-global\                # pnpm 全局
├── OneDrive\
│   └── 文档\
│       └── Obsidian Vault\       # 知识库 (~50个目录)
├── Downloads\
│   └── nvm-setup.exe            # nvm 安装包
└── ...
```

---

# 验证清单

- [ ] nvm version → v1.x.x
- [ ] node -v → v24.x.x
- [ ] pnpm -v → v9.x.x
- [ ] scoop version → 1.x.x
- [ ] git --version → 2.x.x
- [ ] python --version → 3.x.x
- [ ] uv --version → 0.x.x
- [ ] openclaw --version → 2026.x.x
- [ ] openclaw gateway status → running

---

*By 小溪 🦞 2026-03-30*
*Windows 11 包管理器全家桶版*
