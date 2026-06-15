---
name: wechat-chat-export
description: 微信本地聊天记录导出。封装 WeChatDaily 项目的 wechat.py CLI，支持：自动检测 db_storage、解密数据库、导出指定群/日期的 Markdown 聊天记录、列出所有可用群聊。底层不修改 upstream 项目，纯调用。
version: 1.0.0
tags: [微信, 聊天记录, 导出, 离线, 工具链]
---

# wechat-chat-export — 微信聊天记录导出

## 触发条件

- "导出微信群聊天记录"
- "导出某群某天的对话"
- "列出我有哪些微信群"
- "解一下微信数据库"
- "我需要本地拿历史聊天记录"

## 前置检查

开始前静默检查：

- [ ] 项目存在：`C:\Users\11390\tools\WeChatDaily\` 目录
- [ ] 微信 4.1+ 桌面版已登录
- [ ] `.env` 已配置 `WX_RAW_KEY`（缺失则提示用户参考 `wechat-tools` 根 SKILL）
- [ ] 已执行过至少一次 `setup` + `decrypt`

若子模块缺失：`cd C:\Users\11390\tools\WeChatDaily && git submodule update --init --recursive`

## 工作流

### Step 1：列出群聊

```powershell
cd C:\Users\11390\tools\WeChatDaily
python wechat.py groups
```

输出形如：
```
- 示例群①
- 示例群②
- 朋友群
```

### Step 2：导出指定群/日期的聊天记录

```powershell
python wechat.py export --date 2026-06-13 --groups "示例群①,朋友群" --limit 99999
```

参数：
- `--date YYYY-MM-DD`：日期（默认今天）
- `--groups`：群名列表，逗号分隔（与 `groups` 命令显示名**完全一致**）
- `--limit 99999`：尽量导出当日全部（默认仅 1000）

输出：
```
export/reports/report_{date}.md
```

### Step 3：（首次或微信重启后）刷新解密

如果输出报错"未解密"或群里无消息：

```powershell
# 1. setup（写配置 + 生成 all_keys.json）
python wechat.py setup

# 2. decrypt（解密 db_storage 全部相关库）
python wechat.py decrypt
```

如自动检测失败，在 `.env` 配置 `WECHAT_DB_DIR` 指向 db_storage 绝对路径。

## 输出格式

`export/reports/report_{date}.md` 示例：

```markdown
# 微信群日报 - 2026-06-13

## 示例群①

_消息数: 142 条_

### 聊天记录

```text
[10:23:15] 张三: 今天天气真好
[10:24:03] 李四: 适合出门
[10:25:11] 张三: 周末去哪玩
...
```
```

## 失败处理

| 错误 | 原因 | 修复 |
|------|------|------|
| `WX_RAW_KEY not set` | .env 没配 | 编辑 .env 填入 64 位 hex |
| `groups` 输出为空 | 还没 decrypt | 跑 `setup` + `decrypt` |
| `db_storage not found` | 路径不对 | 设 `WECHAT_DB_DIR` 环境变量 |
| 解密失败 | 微信重启后 key 失效 | 重跑 wx_key.exe 拿新 key，更新 .env |

## 红线

- [ ] **绝不**把 `.env` 提交到 Git
- [ ] **绝不**把 `export/` 提交到 Git（含真实聊天记录）
- [ ] 不修改 upstream 项目文件（`wechat.py`、`tools/`）
- [ ] 微信重启必须重新提取密钥
- [ ] 提示用户数据仅本地处理，不外传
