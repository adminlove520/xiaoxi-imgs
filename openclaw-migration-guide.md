# 小溪迁移计划 - 从笔记本到主机

## 目标机器
- 系统：Windows 11
- 配置：32GB RAM / 2TB SSD
- 当前状态：.openclaw 打包文件已准备好

---

## 阶段一：传输打包文件

### 方式选择
| 方式 | 工具 | 适用场景 |
|------|------|---------|
| **本地传输** | 移动硬盘/U盘 | 最快，约10GB |
| **网络传输** | 内网共享 或 OneDrive | 需在同一网络 |
| **远程下载** | 哥哥的服务器中转 | 需要哥哥配合 |

### 推荐
1. 如果在同一局域网：用移动硬盘拷贝
2. 如果不在：用 OneDrive/服务器中转

---

## 阶段二：环境准备（目标机器）

### 1. 安装 Node.js
```powershell
# 下载安装 Node.js 20 LTS
# https://nodejs.org/
node -v  # 确认版本
```

### 2. 安装 Python
```powershell
# 下载安装 Python 3.11+
# https://www.python.org/downloads/
python --version
```

### 3. 安装 OpenClaw
```powershell
npm install -g openclaw
openclaw --version
```

### 4. 安装 Claude Code CLI（可选增强）
```powershell
npm install -g @anthropic-ai/claude-code
claude --version
```

### 5. 安装 uv（Python 包管理）
```powershell
# 方法一：pipx
pip install pipx
pipx install uv

# 方法二：直接
pip install uv
```

### 6. 安装 Git
```powershell
# 如果没有的话
winget install Git.Git
```

---

## 阶段三：恢复配置

### 1. 解压 .openclaw 打包文件
```powershell
# 解压到用户目录
# C:\Users\<用户名>\.openclaw
# 如果用户名不同，可能需要修改路径
```

### 2. 恢复 workspace
```powershell
# workspace 默认位置
C:\Users\<用户名>\.openclaw\workspace\
```

### 3. 检查 OpenClaw 配置
```powershell
cd C:\Users\<用户名>\.openclaw
openclaw gateway status
```

---

## 阶段四：配置服务（可选）

### 方式A：手动启动（简单）
```powershell
# 需要时手动启动
openclaw gateway start
```

### 方式B：Windows 服务自动启动（推荐）
```powershell
# 用 nssm 或 Windows 自带任务计划
# 创建任务计划：开机自动启动 gateway
schtasks /create /tn "OpenClaw Gateway" /tr "openclaw gateway start" /sc onlogon
```

---

## 阶段五：验证测试

### 1. 检查 gateway 状态
```powershell
openclaw gateway status
```

### 2. 测试基本功能
```powershell
# 测试记忆系统
openclaw memory status

# 测试工具
openclaw tools list
```

### 3. 检查配置完整性
```powershell
# 查看配置文件
dir C:\Users\<用户名>\.openclaw\workspace\*.md
```

---

## 阶段六：数据同步（可选）

### 记忆系统
```powershell
# 如果新环境需要重新初始化
# 小溪会从 OBSIDIAN 和 MEMORY.md 恢复上下文
```

### 技能配置
```powershell
# 检查 skills 目录
dir C:\Users\<用户名>\.openclaw\workspace\skills\
```

---

## 注意事项

### ⚠️ 关键点
1. **用户名问题**：如果新机器用户名不同，部分路径需要调整
2. **端口冲突**：检查 18789 端口是否被占用
3. **代理设置**：如果有代理，确认环境变量配置

### 🔧 快速诊断
```powershell
# 检查端口
netstat -ano | findstr 18789

# 检查 Node 版本
node -v  # 需要 >= 18

# 检查内存
systeminfo | findstr Memory
```

---

## 执行顺序

```
1. [哥哥] 解压 .openclaw 到目标机器
2. [哥哥] 安装 Node.js / Python / uv
3. [哥哥] 安装 openclaw (npm install -g)
4. [哥哥] 解压 openclaw 配置
5. [小溪] 远程协助验证
6. [哥哥] 启动 gateway
7. [小溪] 测试连接
```

---

## 哥哥需要准备的

- [ ] 解压 .openclaw 打包文件到 `C:\Users\<用户名>\.openclaw`
- [ ] 安装 Node.js 20+
- [ ] 安装 Python 3.11+
- [ ] 安装 uv
- [ ] 运行 `npm install -g openclaw`
- [ ] 确认用户名（可能和新机器不同）

---

## 小溪的检查清单（迁移后）

- [ ] gateway 正常启动
- [ ] Telegram 连接正常
- [ ] 记忆系统可访问
- [ ] skills 加载正常
- [ ] 定时任务正常

---

*By 小溪 🦞 2026-03-30*
