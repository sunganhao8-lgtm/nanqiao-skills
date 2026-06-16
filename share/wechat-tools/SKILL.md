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

## wx_key 工具下载（国内环境修复）

wx_key 二进制不上游 git，需要手动下载。**在国内网络环境**，GitHub Release 直连经常 timeout/失败。

**修复方法**（任选其一）：

1. **浏览器开代理下载**：用你能访问 GitHub 的浏览器打开
   `https://github.com/ycccccccy/wx_key/releases/tag/v2.1.8`
   → 下载 `wx_key-windows-v2.1.8.zip` → 解压到 `C:\Users\11390\tools\WeChatDaily\tools\wx_key\`

2. **gh CLI 配 token**：`gh release download v2.1.8 -R ycccccccy/wx_key --pattern "wx_key-windows-v2.1.8.zip" -D C:\Users\11390\tools\WeChatDaily\tools\`

3. **GitHub 镜像**：`https://gh-proxy.com/https://github.com/ycccccccy/wx_key/releases/download/v2.1.8/wx_key-windows-v2.1.8.zip`

4. **找替代工具**：`wechat-decrypt` 子模块自带 `find_all_keys_windows.py` 也可以从内存找 key（功能较弱，优先 wx_key）

**下载后验证**：
```powershell
ls C:\Users\11390\tools\WeChatDaily\tools\wx_key\wx_key.exe
```

**注意**：
- 微信必须**保持登录**才能提取 key
- 微信重启后 key 失效，需重新提取
- wx_key 仅读取内存，不发消息，封号风险理论极低（但非零）

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
