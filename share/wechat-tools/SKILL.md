---
name: wechat-tools
description: 微信本地工具集。封装 WeChatDaily 项目，提供：本地聊天记录导出（Markdown/JSON）、解密数据库、生成日报。底层依赖 Bryan-Cyf/WeChatDaily + ylytdeng/wechat-decrypt 子模块，密钥从本地 .env 读取，不上传任何数据。
version: 1.0.0
tags: [微信, 聊天记录, 导出, 离线分析, 工具]
references:
  - chat-export/SKILL.md
  - style-analyzer/SKILL.md
---

# wechat-tools — 微信本地工具集

## 触发条件

- "导出微信聊天记录"
- "微信群聊天记录"
- "分析我的微信对话风格"
- "生成微信日报"
- "wechat"

## 子技能

| 子技能 | 路径 | 用途 |
|--------|------|------|
| **chat-export** | `chat-export/SKILL.md` | 本地导出指定群/日期的聊天记录（Markdown/JSON） |
| **style-analyzer** | `style-analyzer/SKILL.md` | 基于导出数据做 AI 对话风格分析（核心新功能） |

## 项目依赖（本地工具，非 Python 包）

**项目位置**：`C:\Users\11390\tools\WeChatDaily\`

| 依赖 | 用途 |
|------|------|
| `wechat.py` | CLI 入口（export / decrypt / setup / groups 等命令） |
| `tools/wechat-decrypt/` | git submodule，解密 db_storage |
| `tools/wx_key/` | （可选）从内存提取微信密钥，需要手动下载 |
| `scripts/render_manual_report.py` | HTML/PNG 日报渲染 |
| `.env` | 存放 `WX_RAW_KEY`（**绝不提交 Git**） |

## 重要约束

- **100% 本地**：所有解密、导出、分析都在本机完成，**不调用任何远程 API**
- **密钥安全**：`WX_RAW_KEY` 只放 `.env`，从不被读到 LLM 上下文
- **风险声明**：
  - 微信重启后 key 失效，需重新提取
  - 用第三方 hook 工具（wx_key）有理论封号风险，但只读取内存不会发消息
  - 不上传真人对话数据

## 数据流

```
微信本地 db_storage (加密)
   ↓ wx_key 提取密钥
WX_RAW_KEY (在 .env)
   ↓ wechat.py setup + decrypt
export/decrypted/ (解密后的 SQLite)
   ↓ wechat.py export
export/reports/report_{date}.md (Markdown 聊天记录)
   ↓ LLM 读 MD
对话风格画像 / 群日报 JSON
   ↓ scripts/render_manual_report.py
report_{date}.html + .png (可视化)
```

## 安装

1. 克隆项目：`git clone https://github.com/Bryan-Cyf/WeChatDaily.git C:\Users\11390\tools\WeChatDaily`
2. 初始化子模块：`cd WeChatDaily && git submodule update --init --recursive`
3. 安装依赖：`pip install -r requirements.txt` + `pip install -r tools/wechat-decrypt/requirements.txt`
4. 复制 .env：`cp .env.example .env`
5. （可选）下载 wx_key 工具用于提取密钥

详见子技能 SKILL.md。

## 何时不用本 skill

- ❌ 需要实时监听新消息（工具链只支持历史导出）
- ❌ 需要自动回复消息（不提供发送功能）
- ❌ 想要跨平台/跨设备（仅支持本机 Windows 微信 4.1+）
