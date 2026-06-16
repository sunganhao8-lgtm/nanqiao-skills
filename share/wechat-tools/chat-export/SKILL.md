---
name: wechat-chat-export
description: 微信聊天记录导出（群聊 + 私聊）。封装 WeChatDaily 项目的 wechat.py CLI 和 WeChatDecrypt 子模块的 export_messages.py，支持按群名/联系人名导出 Markdown/JSON 格式。新增私聊导出支持。
version: 2.0.0
tags: [微信, 聊天记录, 导出, 私聊, 离线]
---

# wechat-chat-export — 微信聊天记录导出

## 触发条件

- "导出微信群聊天记录"
- "导出我的私聊记录"
- "导出我和 XX 的聊天"
- "列出我有哪些联系人"
- "解一下微信数据库"
- "我需要本地拿历史聊天记录"

## 前置检查

- [ ] 项目存在：`C:\Users\11390\tools\WeChatDaily\`
- [ ] 微信 4.1+ 桌面版已登录
- [ ] `.env` 已配置 `WX_RAW_KEY`
- [ ] 已执行过 `setup` + `decrypt`（解密 18 个 db）

## 工作流

### Step 1：列出所有会话（群聊 + 私聊）

```python
# 使用 WeChatDecrypt 子模块的 export_messages.py
cd C:\Users\11390\tools\WeChatDaily
python tools/wechat-decrypt/export_messages.py
```

输出示例：
```
[message_0.db] 家人闲聊: 185 条
[message_0.db] 梦瑶: 9790 条
[message_0.db] 爸: 3182 条
[message_0.db] Lexi.W: 6610 条
...
```

**区分群聊和私聊**：
- 名字带 `@chatroom` 或群名明显 = 群聊
- 单独人名/wxid = 私聊

### Step 2：导出全量数据（一次性，首次必做）

```bash
cd C:\Users\11390\tools\WeChatDaily
python tools/wechat-decrypt/export_messages.py
```

输出到：`tools/wechat-decrypt/wechat_files/{wxid}/`，每个会话一个子目录
包含文件：`message_0.db.csv` / `.json` / `.html`

### Step 3：读取特定会话的聊天记录

**方式 A：读 JSON（推荐，结构最好）**
```python
import json
with open(f'.../{会话名}/message_0.db.json', 'r', encoding='utf-8') as f:
    data = json.load(f)
# data['messages'] = 消息列表
# 每条消息有: sender, content, create_time, type_name
```

**方式 B：用 wechat.py export 导群聊（原命令也保留）**
```bash
python wechat.py export --date YYYY-MM-DD --groups "群名" --limit 99999
```

### Step 4：风格分析 → 用 `style-analyzer` 子技能

## 输出格式

JSON 格式：
```json
{
  "chat_username": "wxid_xxx",
  "display_name": "联系人名",
  "is_group": false,
  "message_count": 3182,
  "messages": [
    {
      "local_id": 1,
      "type": 1,
      "type_name": "文本",
      "sender": "爸",
      "sender_username": "wxid_xxx",
      "content": "他电脑开不了机",
      "create_time": 1757747540,
      "time_str": "2025-09-13 15:12:20"
    }
  ]
}
```

CSV 格式：`时间,发送者,消息类型,内容,图片路径,server_id`

## 输出目录

```
C:\Users\11390\tools\WeChatDaily\tools\wechat-decrypt\wechat_files\{wxid}\
├── {会话名1}/
│   ├── message_0.db.csv
│   ├── message_0.db.json
│   └── message_0.db.html
├── {会话名2}/
│   └── ...
```

## WeChatDaily 原始项目

项目位置：`C:\Users\11390\tools\WeChatDaily\`
原始仓库：`https://github.com/Bryan-Cyf/WeChatDaily`
WeChatDecrypt 子模块：`tools/wechat-decrypt/`（`ylytdeng/wechat-decrypt`）

## 升级说明 v1→v2

v2 新增：
- **私聊导出**：通过 WeChatDecrypt 的 export_messages.py 直接导出所有会话
- 不再是 wechat.py 的群聊限制
- 135 个会话全量导出验证通过

## 红线

- [ ] **绝不**把 `.env` / `export/` / `wechat_files/` 提交 Git
- [ ] 不修改 upstream 项目文件
- [ ] 微信重启必须重新提取密钥
- [ ] 私聊数据涉及隐私，分析时只提取"我"发的消息
- [ ] Persona 输出只保存风格特征，不保存原始对话内容
