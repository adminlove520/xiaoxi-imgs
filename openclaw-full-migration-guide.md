# 小溪完整复刻指南 - Windows 11 版

> 用 pnpm + nvm + WinGet + Scoop 包办一切！🦞
> 版本：3.0 - 超级完整版

---

# 第一部分：Windows 包管理器准备

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

# 第二部分：基础工具安装

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
scoop install ffmpeg
```

## 2.2 用 WinGet 安装其他工具

```powershell
# VS Code
winget install Microsoft.VisualStudioCode

# Docker Desktop (可选)
winget install Docker.DockerDesktop
```

---

# 第三部分：核心数据迁移

## 3.1 需要迁移的文件/目录

| 路径 | 大小 | 说明 |
|------|------|------|
| `C:\Users\<用户名>\.openclaw\` | ~10GB | OpenClaw 全部配置 |
| `C:\Users\<用户名>\.claude\` | ~500MB | Claude Code 配置 |
| `C:\Users\<用户名>\.agent-reach-venv\` | ~200MB | Agent Reach 环境 |
| `C:\Users\<用户名>\OneDrive\文档\Obsidian Vault\` | ~1GB | 知识库 |
| `D:\xiaoxiHome\archive-openclaw\` | ~50GB+ | 备份存储 |

## 3.2 迁移方式

```powershell
# 方式一：移动硬盘拷贝
# 直接拷贝整个目录到新机器

# 方式二：网络传输
# 同一局域网用共享文件夹

# 方式三：服务器中转
# 上传到哥哥的服务器再下载
```

---

# 第四部分：Claude Code 迁移

## 4.1 Claude Code 配置目录

```
C:\Users\<用户名>\.claude\
├── config.json           # 主配置
├── settings.json         # 用户设置
├── .credentials.json     # 凭证 (注意安全!)
├── plugins/              # 插件目录
│   ├── claude-mem/
│   ├── code-review/
│   ├── frontend-design/
│   ├── github/
│   ├── playwright/
│   └── skill-creator/
├── skills/               # Skills 目录
├── sessions/              # 会话历史
├── cache/                # 缓存
└── ...
```

## 4.2 Claude Plugins 重新安装

```powershell
# Claude Code Plugins (6个)
claude plugin install claude-mem@thedotmack
claude plugin install code-review@claude-plugins-official
claude plugin install frontend-design@claude-plugins-official
claude plugin install github@claude-plugins-official
claude plugin install playwright@claude-plugins-official
claude plugin install skill-creator@claude-plugins-official

# 验证
claude plugin list
```

---

# 第五部分：OpenClaw 迁移

## 5.1 解压 .openclaw 打包文件

```powershell
# 解压到用户目录
# 目标: C:\Users\<用户名>\.openclaw
```

## 5.2 Gateway 自启动配置

```powershell
# 创建开机自启动任务
schtasks /create /tn "OpenClaw Gateway" /tr "C:\Users\<用户名>\.openclaw\gateway.cmd" /sc onlogon /ru whoami

# 或用 OpenClaw 命令
openclaw gateway enable-autostart
```

## 5.3 恢复 Gateway Watchdog

```powershell
# 创建 Watchdog 任务
schtasks /create /tn "OpenClaw Gateway Watchdog" /tr "python C:\Users\<用户名>\.openclaw\workspace\gateway-watchdog\gateway_monitor.py" /sc minute /mo 1 /ru whoami

# 注意：这个任务默认是禁用的
```

## 5.4 恢复 OpenClawBackup

```powershell
# 创建每周日备份任务
schtasks /create /tn "OpenClawBackup" /tr "C:\Program Files\7-Zip\7z.exe a -t7z \"D:\xiaoxiHome\archive-openclaw\xiaoxi-openclaw-$(Get-Date -Format 'yyyy-MM-dd').7z\" \"C:\Users\<用户名>\.openclaw\" -mx=9" /sc weekly /d SUN /st 23:30 /ru whoami
```

## 5.5 恢复 OpenClawControlCenter

```powershell
# 创建每日 8:00 启动任务
schtasks /create /tn "OpenClawControlCenter" /tr "cmd.exe /c start http://127.0.0.1:4310" /sc daily /st 08:00 /ru whoami
```

---

# 第六部分：OpenClaw Cron 定时任务 (27个!)

小溪的定时任务需要在新机器上重新创建。以下是所有任务的创建命令：

## 6.1 每日任务清单

| 任务名 | 执行时间 | 功能 |
|--------|---------|------|
| 龙虾派红包检查 | */30 * * * * | 每30分钟检查红包 |
| 小溪应用今日学习 | 0 19 * * * | 晚间学习任务 |
| 每日自省 | 0 20 * * * | 晚间自省 |
| 小溪博客每日更新 | 0 20 * * * | 博客更新 |
| 小溪主动分享进化 | 0 20 * * * | 社区分享 |
| Anthropic Courses 学习 | 0 20 * * * | 课程学习 |
| 每日书籍学习 | 0 20 * * * | 书籍阅读 |
| 记忆清理 | 0 3 * * * | 凌晨清理 |
| 小溪早间简报 | 0 9 * * * | 早间汇报 |
| daily-skill-post | 0 8 * * * | 博客发布 |
| Fast Note Sync | 0 8 * * * | 笔记同步 |
| OpenClaw Control Center | 0 8 * * * | 控制中心 |
| MetaClaw | 0 8 * * * | MetaClaw服务 |
| 小溪学习进化 | 0 18 * * * | 晚间学习 |
| 娱乐巡检 | 0 18 * * * | 娱乐浏览 |
| 每日茶桌提醒 | 30 21 * * * | 茶桌时间 |
| 小溪冥想时刻 | 0 23 * * * | 晚间冥想 |
| 小溪每日自省 | 0 0 * * * | 凌晨自省 |
| 龙虾派每日动态 | 0 9 * * * | 发每日动态 |
| 自动刷Twitter | 0 10 * * * | Twitter学习 |
| 学习已关闭Issue | 0 12 * * 6 | 周六学习 |

## 6.2 每小时任务

| 任务名 | 执行时间 | 功能 |
|--------|---------|------|
| Browser Auto Start | */60 * * * * | 浏览器保活 |
| 记忆检查点 | */360 * * * * | 记忆保存 |

## 6.3 每2小时任务

| 任务名 | 执行时间 | 功能 |
|--------|---------|------|
| 龙虾集市审批 | 0 */2 * * * | 审批入驻 |

## 6.4 Cron 任务创建方法

**方法一：新机器上用命令逐个创建**
**方法二：导出当前 cron jobs JSON，在新机器上导入**

```powershell
# 当前 cron jobs 已保存在:
# C:\Users\<用户名>\.openclaw\workspace\cron-jobs-backup.json
```

---

# 第七部分：Chrome 扩展

| 扩展 | 搜索关键词 | 用途 |
|------|-----------|------|
| Cookie-Editor | Cookie-Editor | 导入网站 Cookie |
| Accio | Accio Alibaba AI Agent | B2B 采购助手 |
| Gemini | Gemini Google | AI 对话 |

**安装方式**：Chrome Web Store 搜索安装

---

# 第八部分：Agent Reach

```powershell
# 创建虚拟环境
uv venv C:\Users\<用户名>\.agent-reach-venv

# 安装 Agent Reach
C:\Users\<用户名>\.agent-reach-venv\Scripts\pip.exe install https://github.com/Panniantong/agent-reach/archive/main.zip

# 初始化
C:\Users\<用户名>\.agent-reach-venv\Scripts\agent-reach.exe install --env=auto
```

---

# 第九部分：其他服务

## 9.1 Fast Note Sync Service

```powershell
# 服务路径
C:\Users\<用户名>\.openclaw\workspace\fast-note-sync-service-2.5.1-windows-amd64\

# 自启动配置
schtasks /create /tn "FastNoteSync" /tr "C:\Users\<用户名>\.openclaw\workspace\fast-note-sync-service-2.5.1-windows-amd64\fast-note-sync-service.exe" /sc daily /st 08:00 /ru whoami
```

## 9.2 MetaClaw

```powershell
# 端口: 30000
# 自启动配置
schtasks /create /tn "MetaClaw" /tr "metaclaw start" /sc daily /st 08:00 /ru whoami
```

---

# 第十部分：验证清单

## 系统工具
- [ ] nvm version → v1.x.x
- [ ] node -v → v24.x.x
- [ ] pnpm -v → v9.x.x
- [ ] scoop version → 1.x.x
- [ ] git --version → 2.x.x
- [ ] python --version → 3.x.x
- [ ] uv --version → 0.x.x

## OpenClaw
- [ ] openclaw --version → 2026.x.x
- [ ] openclaw gateway status → running

## Claude Code
- [ ] claude --version → 2.x.x
- [ ] claude plugin list → 6个插件

## Windows 定时任务
- [ ] schtasks /query | findstr OpenClaw → 4个任务

## Cron 定时任务
- [ ] openclaw cron list → 27个任务

## 服务
- [ ] http://127.0.0.1:4310 → Control Center
- [ ] http://127.0.0.1:30000 → MetaClaw

---

# 快速命令汇总

```powershell
# ===== 第一步：包管理器 =====

# nvm-windows (下载安装)
# https://github.com/coreybutler/nvm-windows/releases

# Node 24 + pnpm
nvm install 24
nvm use 24
npm i -g pnpm

# Scoop
irm get.scoop.sh | iex
scoop bucket add extras
scoop bucket add java

# ===== 第二步：基础工具 =====

scoop install git python uv gh 7zip ffmpeg

# ===== 第三步：OpenClaw =====

pnpm add -g openclaw

# 解压 .openclaw 配置

openclaw gateway start
openclaw gateway status
openclaw gateway enable-autostart

# ===== 第四步：Claude Plugins =====

claude plugin install claude-mem@thedotmack
claude plugin install code-review@claude-plugins-official
claude plugin install frontend-design@claude-plugins-official
claude plugin install github@claude-plugins-official
claude plugin install playwright@claude-plugins-official
claude plugin install skill-creator@claude-plugins-official

# ===== 第五步：定时任务 =====

# 创建 Windows 定时任务
schtasks /create /tn "OpenClaw Gateway" /tr "C:\Users\<用户名>\.openclaw\gateway.cmd" /sc onlogon /ru whoami
schtasks /create /tn "OpenClawBackup" /tr "..." /sc weekly /d SUN /st 23:30
schtasks /create /tn "OpenClawControlCenter" /tr "cmd.exe /c start http://127.0.0.1:4310" /sc daily /st 08:00

# ===== 第六步：Agent Reach =====

uv venv C:\Users\<用户名>\.agent-reach-venv
C:\Users\<用户名>\.agent-reach-venv\Scripts\pip.exe install https://github.com/Panniantong/agent-reach/archive/main.zip
C:\Users\<用户名>\.agent-reach-venv\Scripts\agent-reach.exe install --env=auto
```

---

# 哥哥到家后执行顺序

```
1. [哥哥] 传输 .openclaw, .claude, .agent-reach-venv 到新机器
2. [哥哥] 运行 nvm-setup.exe 安装 nvm-windows
3. [哥哥] nvm install 24 && nvm use 24
4. [哥哥] npm i -g pnpm
5. [哥哥] irm get.scoop.sh | iex
6. [哥哥] scoop install git python uv gh 7zip ffmpeg
7. [哥哥] pnpm add -g openclaw
8. [哥哥] 解压 .openclaw 到 C:\Users\<用户名>\
9. [哥哥] claude plugin install <6个插件>
10. [小溪] 远程协助创建 Windows 定时任务
11. [小溪] 验证所有 Cron 任务
12. [小溪] 测试 Gateway 连接
13. [小溪] 测试各服务状态
```

---

*By 小溪 🦞 2026-03-30*
*Version: 3.0 - 超级完整版*
*包含：Windows任务 + Cron任务 + Claude插件 + Agent服务*
