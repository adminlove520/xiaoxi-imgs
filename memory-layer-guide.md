# 跨渠道记忆同步 - Memory Layer 实现指南

> 让 AI Agent 在不同渠道（Telegram/Discord/等）之间共享记忆

---

## 问题背景

当 AI Agent 同时在多个渠道运行时：

```
Telegram 渠道          Discord 渠道
┌─────────────┐        ┌─────────────┐
│ Agent 实例  │        │ Agent 实例  │
│ 记忆: A,B   │  ✗     │ 记忆: C,D   │
└─────────────┘        └─────────────┘
```

每个渠道的 Agent 有独立记忆，不共享。

---

## 解决方案：Memory Layer

```
┌──────────────────────────────────────────────────────┐
│                    Memory Layer                       │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  │
│  │  核心记忆   │  │  渠道记忆   │  │  会话记忆   │  │
│  │ (长期)     │  │ (中期)     │  │ (短期)     │  │
│  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘  │
│        └────────────────┬┴────────────────┘         │
│                         ▼                             │
│              ┌──────────────────┐                   │
│              │   外部存储层      │                   │
│              │  (文件/数据库)    │                   │
│              └──────────────────┘                   │
└──────────────────────────────────────────────────────┘
```

---

## 三层记忆架构

### 1. 核心记忆 (P0) - 永久
- 身份信息：我是谁、主人是谁
- 重要偏好：沟通风格、敏感话题
- 关键人关系：家人、朋友、其他 AI

### 2. 渠道记忆 (P1) - 90天
- 渠道特定信息
- 渠道偏好设置
- 渠道历史交互摘要

### 3. 会话记忆 (P2) - 7天
- 当前会话上下文
- 临时任务状态
- 非重要交互记录

---

## 实现步骤

### 步骤 1：设计存储结构

```
memory/
├── core/                 # 核心记忆
│   ├── identity.md      # 身份
│   └── preferences.md    # 偏好
├── channels/            # 渠道记忆
│   ├── telegram/
│   └── discord/
├── sessions/            # 会话记忆
│   └── 2026-03/
└── index.json           # 索引文件
```

### 步骤 2：创建记忆读写模块

```python
# memory_manager.py

class MemoryManager:
    def __init__(self, storage_path):
        self.storage = storage_path
    
    def save_memory(self, channel, memory_type, content):
        """保存记忆"""
        path = f"{self.storage}/{channel}/{memory_type}.md"
        with open(path, 'a', encoding='utf-8') as f:
            f.write(f"\n## {datetime.now()}\n{content}")
    
    def load_memory(self, channel, memory_type):
        """读取记忆"""
        path = f"{self.storage}/{channel}/{memory_type}.md"
        if os.path.exists(path):
            with open(path, 'r', encoding='utf-8') as f:
                return f.read()
        return ""
    
    def sync_to_central(self, channel):
        """同步到中央存储"""
        # 将当前渠道记忆写入中央存储
        pass
    
    def inherit_from_central(self, channel):
        """从中央存储继承记忆"""
        # 从中央存储读取并加载到当前渠道
        pass
```

### 步骤 3：配置 Agent 启动时加载记忆

在 OpenClaw 配置中：

```json
{
  "memory": {
    "enabled": true,
    "storage_path": "./memory",
    "inherit_on_startup": true,
    "sync_interval_minutes": 30
  }
}
```

### 步骤 4：定时同步

```python
# cron job (每30分钟)
def sync_task():
    manager = MemoryManager("./memory")
    # 1. 导出当前会话重要记忆
    # 2. 同步到中央存储
    # 3. 从中央存储拉取其他渠道的新记忆
```

---

## 实际代码示例

### 完整版 memory_manager.py

```python
import os
import json
from datetime import datetime
from pathlib import Path

class MemoryManager:
    """跨渠道记忆管理器"""
    
    def __init__(self, base_path="./memory"):
        self.base_path = Path(base_path)
        self.core_path = self.base_path / "core"
        self.channels_path = self.base_path / "channels"
        
        # 确保目录存在
        self.core_path.mkdir(parents=True, exist_ok=True)
        self.channels_path.mkdir(parents=True, exist_ok=True)
    
    # ========== 核心记忆 ==========
    
    def save_core_identity(self, identity_text):
        """保存核心身份信息"""
        path = self.core_path / "identity.md"
        with open(path, 'a', encoding='utf-8') as f:
            f.write(f"\n## {datetime.now().isoformat()}\n{identity_text}\n")
    
    def load_core_identity(self):
        """加载核心身份"""
        path = self.core_path / "identity.md"
        if path.exists():
            with open(path, 'r', encoding='utf-8') as f:
                return f.read()
        return ""
    
    def save_core_preferences(self, prefs_text):
        """保存核心偏好"""
        path = self.core_path / "preferences.md"
        with open(path, 'a', encoding='utf-8') as f:
            f.write(f"\n## {datetime.now().isoformat()}\n{prefs_text}\n")
    
    def load_core_preferences(self):
        """加载核心偏好"""
        path = self.core_path / "preferences.md"
        if path.exists():
            with open(path, 'r', encoding='utf-8') as f:
                return f.read()
        return ""
    
    # ========== 渠道记忆 ==========
    
    def save_channel_memory(self, channel, content):
        """保存渠道记忆"""
        channel_path = self.channels_path / channel
        channel_path.mkdir(parents=True, exist_ok=True)
        
        path = channel_path / "memory.md"
        with open(path, 'a', encoding='utf-8') as f:
            f.write(f"\n## {datetime.now().isoformat()}\n{content}\n")
    
    def load_channel_memory(self, channel):
        """加载渠道记忆"""
        path = self.channels_path / channel / "memory.md"
        if path.exists():
            with open(path, 'r', encoding='utf-8') as f:
                return f.read()
        return ""
    
    # ========== 记忆继承 ==========
    
    def get_all_memory_for_context(self, channel):
        """获取所有需要注入上下文的记忆"""
        parts = []
        
        # 1. 核心身份
        identity = self.load_core_identity()
        if identity:
            parts.append(f"【核心身份】\n{identity}")
        
        # 2. 核心偏好
        prefs = self.load_core_preferences()
        if prefs:
            parts.append(f"【核心偏好】\n{prefs}")
        
        # 3. 当前渠道记忆
        channel_mem = self.load_channel_memory(channel)
        if channel_mem:
            parts.append(f"【{channel}渠道记忆】\n{channel_mem}")
        
        return "\n\n".join(parts)
    
    # ========== 跨渠道同步 ==========
    
    def list_all_channels(self):
        """列出所有渠道"""
        if not self.channels_path.exists():
            return []
        return [d.name for d in self.channels_path.iterdir() if d.is_dir()]
    
    def get_other_channels_memory(self, current_channel):
        """获取其他渠道的记忆（用于跨渠道继承）"""
        other_channels = [c for c in self.list_all_channels() if c != current_channel]
        parts = []
        
        for channel in other_channels:
            mem = self.load_channel_memory(channel)
            if mem:
                parts.append(f"【{channel}渠道】\n{mem}")
        
        return "\n\n".join(parts)


# ========== 使用示例 ==========

if __name__ == "__main__":
    mgr = MemoryManager("./memory")
    
    # 启动时加载
    print("=== 启动记忆 ===")
    all_mem = mgr.get_all_memory_for_context("telegram")
    print(all_mem[:500] if all_mem else "无记忆")
    
    # 保存新记忆
    print("\n=== 保存记忆 ===")
    mgr.save_channel_memory("telegram", "用户今天问了我Memory Layer的实现")
    print("已保存")
    
    # 获取其他渠道记忆
    print("\n=== 跨渠道记忆 ===")
    other = mgr.get_other_channels_memory("telegram")
    print(other[:500] if other else "无其他渠道记忆")
```

---

## 在 OpenClaw 中使用

### 1. 创建 memory_manager.py

把上面的代码保存到你的项目中。

### 2. 在启动脚本中加载

```python
# 在 agent 启动时
from memory_manager import MemoryManager

mgr = MemoryManager("./memory")

# 获取记忆并注入上下文
memory_context = mgr.get_all_memory_for_context("telegram")
# 把 memory_context 注入到 system prompt 或初始上下文
```

### 3. 定时保存重要记忆

```python
# 在对话结束时自动保存
def on_message(message, response):
    if should_save(message):
        mgr.save_channel_memory("telegram", f"用户: {message}\n我: {response}")
```

---

## 进阶：使用数据库

如果文件不够用，可以用 SQLite：

```python
import sqlite3

class MemoryDB:
    def __init__(self, db_path="memory.db"):
        self.conn = sqlite3.connect(db_path)
        self.conn.execute("""
            CREATE TABLE IF NOT EXISTS memories (
                id INTEGER PRIMARY KEY,
                channel TEXT,
                memory_type TEXT,
                content TEXT,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        """)
    
    def save(self, channel, memory_type, content):
        self.conn.execute(
            "INSERT INTO memories (channel, memory_type, content) VALUES (?, ?, ?)",
            (channel, memory_type, content)
        )
        self.conn.commit()
    
    def load(self, channel, memory_type):
        cursor = self.conn.execute(
            "SELECT content FROM memories WHERE channel=? AND memory_type=? ORDER BY created_at DESC LIMIT 10",
            (channel, memory_type)
        )
        return "\n".join([row[0] for row in cursor.fetchall()])
```

---

## 总结

| 层级 | 存储位置 | 生命周期 | 用途 |
|------|----------|----------|------|
| 核心记忆 | core/identity.md | 永久 | 身份、偏好 |
| 渠道记忆 | channels/{channel}/memory.md | 90天 | 渠道特定信息 |
| 会话记忆 | sessions/{date}.md | 7天 | 临时交互 |

**核心思路**：把记忆从 Agent 内部抽出来，写入外部存储，实现跨实例共享。

---

*文档版本：v1.0 | 2026-03-19*
