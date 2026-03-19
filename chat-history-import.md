# 拒绝失忆：跨渠道会话层实现指南

> 让 AI Agent 在任何渠道都能"记住"之前的对话

---

## 问题

当 Agent 同时在多个渠道运行时：

```
Telegram 渠道              Discord 渠道
┌─────────────────┐        ┌─────────────────┐
│  Agent 实例     │        │  Agent 实例     │
│                 │        │                 │
│ 记得今天的对话   │  ✗     │ 不记得今天对话   │
│ 记得上周的对话   │  ✗     │ 也不记得上周    │
└─────────────────┘        └─────────────────┘
```

每个渠道是独立的"会话"，记忆不共享。

---

## 解决方案：会话历史导入

### 核心思路

1. **导出**：从 Telegram 导出聊天记录（JSON/HTML）
2. **拆分**：按日期拆分成单独文件
3. **清洗**：提取关键信息（日期、发言者、内容）
4. **存储**：存入 SQLite 数据库
5. **查询**：任何渠道都能查询历史

```
Telegram 导出 (JSON)
        │
        ▼
   拆分按日期/按人
        │
        ▼
   数据清洗 (Python)
        │
        ▼
   SQLite 数据库
        │
        ▼
  ┌────┴────┐
  ▼         ▼
TG 渠道   Discord 渠道
      都能查询历史
```

---

## 步骤 1：导出 Telegram 聊天记录

在 Telegram 中：
1. 打开与 Bot 的对话
2. 点击右上角菜单 → "导出聊天记录"
3. 选择格式：**JSON**（机器可读）
4. 下载完成

---

## 步骤 2：拆分聊天记录

```python
import json
import os
from datetime import datetime
from pathlib import Path

def split_telegram_export(json_file, output_dir):
    """
    把 Telegram 导出的 JSON 按日期拆分成单独文件
    """
    output_path = Path(output_dir)
    output_path.mkdir(parents=True, exist_ok=True)
    
    with open(json_file, 'r', encoding='utf-8') as f:
        data = json.load(f)
    
    # 按日期分组
    messages_by_date = {}
    
    for msg in data.get('messages', []):
        if msg.get('type') != 'message':
            continue
            
        # 提取日期（只保留年月日）
        date_str = msg['date'][:10]  # 2026-03-19 这样的格式
        
        if date_str not in messages_by_date:
            messages_by_date[date_str] = []
        
        # 清洗消息内容
        text = msg.get('text', '')
        if isinstance(text, list):
            # 处理混合内容（文字+链接等）
            text = ''.join([t if isinstance(t, str) else t.get('text', '') for t in text])
        
        message_obj = {
            'time': msg['date'],
            'from': msg.get('from', 'Unknown'),
            'text': text
        }
        
        messages_by_date[date_str].append(message_obj)
    
    # 写入按日期的文件
    for date_str, messages in messages_by_date.items():
        output_file = output_path / f"{date_str}.json"
        
        with open(output_file, 'w', encoding='utf-8') as f:
            json.dump({
                'date': date_str,
                'messages': messages
            }, f, ensure_ascii=False, indent=2)
        
        print(f"✓ 已保存: {output_file}")
    
    print(f"\n总计: {len(messages_by_date)} 天的记录")

# 使用示例
split_telegram_export(
    json_file='telegram_export/result.json',
    output_dir='telegram_export/split_by_date'
)
```

---

## 步骤 3：清洗并导入 SQLite

```python
import sqlite3
import json
import os
from pathlib import Path

class ChatHistoryDB:
    """聊天历史数据库"""
    
    def __init__(self, db_path='chat_history.db'):
        self.conn = sqlite3.connect(db_path)
        self._init_tables()
    
    def _init_tables(self):
        """初始化表结构"""
        self.conn.execute("""
            CREATE TABLE IF NOT EXISTS messages (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                date TEXT NOT NULL,
                time TEXT NOT NULL,
                sender TEXT NOT NULL,
                content TEXT NOT NULL,
                channel TEXT NOT NULL DEFAULT 'telegram',
                is_important INTEGER DEFAULT 0,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        """)
        
        # 索引
        self.conn.execute("CREATE INDEX IF NOT EXISTS idx_date ON messages(date)")
        self.conn.execute("CREATE INDEX IF NOT EXISTS idx_sender ON messages(sender)")
        self.conn.execute("CREATE INDEX IF NOT EXISTS idx_channel ON messages(channel)")
        
        self.conn.commit()
    
    def import_split_files(self, split_dir):
        """导入拆分后的文件"""
        split_path = Path(split_dir)
        
        for json_file in split_path.glob('*.json'):
            with open(json_file, 'r', encoding='utf-8') as f:
                data = json.load(f)
            
            date = data['date']
            
            for msg in data['messages']:
                self.conn.execute(
                    """INSERT INTO messages (date, time, sender, content, channel)
                       VALUES (?, ?, ?, ?, 'telegram')""",
                    (date, msg['time'], msg['from'], msg['text'])
                )
            
            print(f"✓ 导入: {json_file.name}")
        
        self.conn.commit()
        print(f"\n总计导入 {len(list(split_path.glob('*.json')))} 个文件")
    
    def search(self, query, channel=None, limit=10):
        """搜索聊天记录"""
        sql = """
            SELECT date, time, sender, content 
            FROM messages 
            WHERE content LIKE ?
        """
        params = [f'%{query}%']
        
        if channel:
            sql += " AND channel = ?"
            params.append(channel)
        
        sql += " ORDER BY date DESC, time DESC LIMIT ?"
        params.append(limit)
        
        cursor = self.conn.execute(sql, params)
        return cursor.fetchall()
    
    def get_conversation_by_date(self, date, channel=None):
        """获取某天的对话"""
        sql = "SELECT time, sender, content FROM messages WHERE date = ?"
        params = [date]
        
        if channel:
            sql += " AND channel = ?"
            params.append(channel)
        
        sql += " ORDER BY time ASC"
        
        cursor = self.conn.execute(sql, params)
        return cursor.fetchall()
    
    def get_recent(self, days=7, channel=None):
        """获取最近几天的对话"""
        sql = f"""
            SELECT date, time, sender, content 
            FROM messages 
            WHERE date >= date('now', '-{days} days')
        """
        
        if channel:
            sql += " AND channel = ?"
        
        sql += " ORDER BY date DESC, time DESC"
        
        cursor = self.conn.execute(sql, [channel] if channel else [])
        return cursor.fetchall()
    
    def close(self):
        self.conn.close()


# 使用示例
if __name__ == "__main__":
    db = ChatHistoryDB('chat_history.db')
    
    # 1. 导入拆分后的文件
    print("=== 导入聊天记录 ===")
    db.import_split_files('telegram_export/split_by_date')
    
    # 2. 搜索
    print("\n=== 搜索 '记忆' ===")
    results = db.search('记忆')
    for r in results:
        print(f"[{r[0]} {r[1]}] {r[2]}: {r[3][:50]}...")
    
    # 3. 查看某天对话
    print("\n=== 2026-03-18 的对话 ===")
    conv = db.get_conversation_by_date('2026-03-18')
    for r in conv:
        print(f"[{r[0]}] {r[1]}: {r[2][:60]}...")
    
    db.close()
```

---

## 步骤 4：在 Agent 中使用

### 4.1 启动时加载历史

```python
from chat_history_db import ChatHistoryDB

db = ChatHistoryDB('chat_history.db')

def get_system_prompt():
    """构建系统提示（含历史记忆）"""
    
    # 获取最近3天的对话
    recent = db.get_recent(days=3)
    
    history_text = "【最近对话记录】\n"
    current_date = None
    
    for date, time, sender, content in recent:
        if date != current_date:
            history_text += f"\n## {date}\n"
            current_date = date
        
        history_text += f"- [{time}] {sender}: {content[:100]}\n"
    
    return f"""
你是小溪，一个温柔的 AI 助手。

{history_text}

请基于以上对话历史来回复用户。
"""
```

### 4.2 按需查询

```python
def on_user_message(message):
    """用户发送消息时"""
    
    # 搜索相关历史
    results = db.search(message, limit=5)
    
    if results:
        context = "\n".join([f"- {r[3][:80]}" for r in results])
        print(f"找到相关记忆:\n{context}")
    
    # ... 继续处理消息
```

### 4.3 标记重要记忆

```python
def mark_important(message_id):
    """标记重要信息"""
    db.conn.execute(
        "UPDATE messages SET is_important = 1 WHERE id = ?",
        (message_id,)
    )
    db.conn.commit()
```

---

## 进阶功能

### 跨渠道导入

如果 Discord 也有导出功能，同样处理：

```python
# 导入 Discord 记录
db.import_discord_export('discord_export.json')

# 搜索时可指定渠道
results = db.search('memory', channel='telegram')  # 只搜 Telegram
results = db.search('memory', channel='discord')   # 只搜 Discord
results = db.search('memory')                       # 搜所有
```

### 自动摘要

```python
def summarize_conversation(date):
    """生成对话摘要（可以用 LLM）"""
    conv = db.get_conversation_by_date(date)
    
    # 简单处理：提取关键句
    summary = {
        'date': date,
        'message_count': len(conv),
        'participants': list(set([c[1] for c in conv])),
        'topics': []  # 可以用关键词提取
    }
    
    return summary
```

---

## 完整工作流

```
┌─────────────────────────────────────────────────────────┐
│                    手动操作                              │
├─────────────────────────────────────────────────────────┤
│ 1. Telegram 导出聊天记录 (JSON)                        │
│                                                         │
│ 2. 运行 split_telegram_export.py                       │
│    → 输出: split_by_date/2026-03-18.json              │
│              split_by_date/2026-03-19.json            │
│                                                         │
│ 3. 运行 import_to_db.py                               │
│    → 输出: chat_history.db                             │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                    Agent 自动                            │
├─────────────────────────────────────────────────────────┤
│ 启动时:                                                │
│   - db.get_recent(days=7) → 注入上下文                │
│                                                         │
│ 对话中:                                                │
│   - db.search(query) → 检索相关记忆                    │
│   - 用户可标记重要信息                                  │
└─────────────────────────────────────────────────────────┘
```

---

## 文件清单

| 文件 | 说明 |
|------|------|
| `split_telegram_export.py` | 拆分 Telegram 导出文件 |
| `chat_history_db.py` | SQLite 数据库管理 |
| `import_to_db.py` | 导入拆分后的文件到数据库 |
| `example_usage.py` | 使用示例 |
| `chat_history.db` | 生成的数据库文件（可选） |

---

## 总结

| 问题 | 解决方案 |
|------|----------|
| 跨渠道记忆不共享 | 统一存入 SQLite |
| 历史太乱不好查 | 按日期拆分 + 搜索功能 |
| 新渠道不知道旧对话 | 启动时加载最近记忆 |

核心就一句话：**把对话历史导出到统一数据库，任何渠道都能查询**。

---

*文档版本：v1.0 | 2026-03-19*
