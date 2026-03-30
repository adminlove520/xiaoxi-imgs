# 小溪完整复刻计划 - 从笔记本到主机

## 目标机器
- 系统：Windows 11  
- 配置：32GB RAM / 2TB SSD
- 当前状态：.openclaw 打包文件已准备好

---

# 第一部分：核心环境安装

## 1. Node.js 环境

```powershell
# 下载 Node.js 20 LTS
# https://nodejs.org/

# 验证安装
node -v      # 需要 >= 18
npm -v       # 需要 >= 9
```

## 2. Python 环境

```powershell
# 下载 Python 3.11+
# https://www.python.org/downloads/

# 验证安装
python --version
pip --version
```

## 3. uv (Python 包管理)

```powershell
# 方法一：pip 安装
pip install uv

# 方法二：pipx (推荐)
pip install pipx
pipx install uv
```

## 4. Git

```powershell
# 如果没有
winget install Git.Git

# 验证
git --version
```

## 5. OpenClaw

```powershell
npm install -g openclaw
openclaw --version
```

---

# 第二部分：OpenClaw 配置恢复

## 1. 解压 .openclaw 打包文件

```powershell
# 解压到用户目录
# 目标路径: C:\Users\<用户名>\.openclaw
# 注意：如果新机器用户名不同，需要调整
```

## 2. 目录结构

```
C:\Users\<用户名>\.openclaw\
├── config\           # OpenClaw 配置
├── extensions\       # 扩展插件
├── plugins\          # 插件
├── skills\           # Skills (~18个)
├── workspace\        # 工作空间
│   ├── skills\      # Workspace Skills (~65个!)
│   ├── memory\       # 记忆系统
│   ├── docs\         # 文档
│   └── ...
└── ...
```

## 3. 检查 Gateway

```powershell
openclaw gateway status
openclaw gateway start
```

---

# 第三部分：Skills 复刻清单

## OpenClaw Skills (~18个)

| Skill | 来源 | 说明 |
|-------|------|------|
| agent-reach | ClawHub | Agent 工具集 |
| ai-daily-newsletter | ClawHub | AI 日报 |
| clawbot-market | ClawHub | 龙虾集市 |
| clawpi-redpacket-monitor | ClawHub | 红包监控 |
| find-skills | ClawHub | 技能发现 |
| fluxa-agent-wallet | ClawHub | 钱包 |
| lyric-sense | 本地 | 歌词工具 |
| netease-music-* | 本地 | 网易云音乐 |
| gogcli | 本地 | Google 工作区 |
| x402-payments | ClawHub | 支付 |
| self-improving-agent | ClawHub | 自我改进 |
| ... | ... | ... |

**安装方式**：
```powershell
# 从 ClawHub 安装
npx clawdhub install <skill-name>

# 本地 Skills 已在 .openclaw 打包中
```

## Workspace Skills (~65个!) 

**最全的 Skills 集合！**

| 类别 | Skills |
|------|--------|
| AI/开发 | frontend-dev, fullstack-dev, android-dev, ios-dev, shader-dev |
| 效率工具 | minimax-docx/pdf/xlsx, pptx-generator, obsidian |
| 自动化 | agent-reach, self-improving, proactive-solvr |
| 知识管理 | memory-curator, cognitive-memory, elite-longterm-memory |
| 媒体 | lyric-sense, movie-subtitle-viewer, netease-music-* |
| 支付 | x402-payments, fluxa-agent-wallet |
| 社交 | github, tavily, newsnow |

**大部分在 .openclaw 打包中，迁移后自动恢复！**

---

# 第四部分：Claude Code Plugins

## 已安装的 Plugins

| Plugin | 版本 | 状态 |
|--------|------|------|
| claude-mem | 10.6.2 | 已装 |
| code-review | 官方 | 已装 |
| frontend-design | 官方 | 已装 |
| github | 官方 | 已装 |
| playwright | 官方 | 已装 |
| skill-creator | 官方 | 已装 |

**复刻命令**：
```powershell
claude plugin install claude-mem@latest
claude plugin install code-review
claude plugin install frontend-design
claude plugin install github
claude plugin install playwright
claude plugin install skill-creator
```

---

# 第五部分：浏览器扩展

## OpenClaw Browser Bridge
- 路径：`C:\Users\whoami\.openclaw\browser\openclaw\`
- 用于 opencli 连接

## Chrome 扩展

| 扩展 | 用途 |
|------|------|
| Cookie-Editor | 导入网站 Cookie |
| Accio | B2B AI 采购助手 |
| Gemini Web | AI 对话（已配置） |

**安装来源**：
- Chrome Web Store 搜索安装

---

# 第六部分：Obsidian 知识库

## Vault 位置
```
C:\Users\<用户名>\OneDrive\文档\Obsidian Vault\
```

## Vault 结构
```
Obsidian Vault/
├── 01-学习/          # 学习笔记
├── 04-书籍阅读/      # 书籍和 PDF
├── 04-AI学习/       # AI 相关
├── 04-Anthropic Courses学习/  # 课程笔记
├── partner/          # 伙伴信息
├── 小溪/             # 小溪专属
├── 日记/             # 日记
└── ... (共 ~50 个目录)
```

## 复刻方式
1. **OneDrive 自动同步** ✅ (如果使用相同账号)
2. **手动备份** - 打包 `Obsidian Vault` 文件夹

---

# 第七部分：Agent Reach

## 安装
```powershell
# Python venv 方式
python -m venv C:\Users\<用户名>\.agent-reach-venv
C:\Users\<用户名>\.agent-reach-venv\Scripts\pip.exe install https://github.com/Panniantong/agent-reach/archive/main.zip

# 初始化
C:\Users\<用户名>\.agent-reach-venv\Scripts\agent-reach.exe install --env=auto
```

## 工具目录
```
C:\Users\<用户名>\.agent-reach\
├── tools/           # 各种工具
│   ├── xiaoyuzhou/   # 小宇宙播客
│   ├── weibo-mcp/    # 微博
│   └── ...
└── config.json
```

---

# 第八部分：其他工具

## 已安装的全局工具

| 工具 | 版本 | 用途 |
|------|------|------|
| Claude Code CLI | 2.1.85 | 代码助手 |
| yt-dlp | 2026.3.17 | 视频下载 |
| gh CLI | - | GitHub |
| opencli | - | 命令行工具 |

## npm 全局包
```powershell
npm list -g --depth=0
```

---

# 第九部分：Windows 配置

## 环境变量（可选）

```powershell
# 如果需要代理
setx HTTP_PROXY "http://user:pass@ip:port"
setx HTTPS_PROXY "http://user:pass@ip:port"
```

## 端口检查

```powershell
# 检查 18789 端口
netstat -ano | findstr 18789

# 检查 18800 (Browser CDP)
netstat -ano | findstr 18800
```

---

# 执行清单

## 哥哥到家后要做的事

### 第一步：安装基础环境 (~10分钟)
- [ ] 安装 Node.js 20+
- [ ] 安装 Python 3.11+
- [ ] 安装 uv
- [ ] 安装 Git
- [ ] 安装 openclaw: `npm install -g openclaw`

### 第二步：恢复配置 (~5分钟)
- [ ] 解压 .openclaw 到 `C:\Users\<用户名>\`
- [ ] 解压 .agent-reach-venv (如果单独备份)
- [ ] 恢复 Obsidian Vault

### 第三步：配置 Services (~5分钟)
- [ ] 运行 `openclaw gateway start`
- [ ] 检查 `openclaw gateway status`

### 第四步：验证功能 (~10分钟)
- [ ] 测试 Telegram 连接
- [ ] 测试记忆系统
- [ ] 检查 Skills 加载
- [ ] 测试定时任务

### 第五步：可选增强 (~5分钟)
- [ ] 安装 Claude Code Plugins
- [ ] 配置 Agent Reach
- [ ] 安装 Chrome 扩展

---

# 快速命令汇总

```powershell
# 1. 安装基础环境
winget install OpenJS.NodeJS.LTS
winget install Python.Python.3.11
pip install uv

# 2. 安装 OpenClaw
npm install -g openclaw

# 3. 启动
openclaw gateway start
openclaw gateway status

# 4. Claude Plugins
claude plugin install claude-mem
claude plugin install github
claude plugin install code-review
claude plugin install frontend-design

# 5. Agent Reach
python -m venv $env:USERPROFILE\.agent-reach-venv
$env:USERPROFILE\.agent-reach-venv\Scripts\pip.exe install https://github.com/Panniantong/agent-reach/archive/main.zip
$env:USERPROFILE\.agent-reach-venv\Scripts\agent-reach.exe install --env=auto
```

---

*By 小溪 🦞 2026-03-30*
*Version: 2.0 - 完整版*
