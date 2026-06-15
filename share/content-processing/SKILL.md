---
name: content-processing
description: 内容处理工具集。去 AI 痕迹润色、Markdown 格式化、多语言翻译、URL 转 Markdown。合并自 humanizer-zh、baoyu-format-markdown、baoyu-translate、baoyu-url-to-markdown。
version: 1.0.0
---

# content-processing — 内容处理

## 触发条件

- 写完文案需要去 AI 痕迹
- 需要格式化 Markdown
- 需要翻译文本
- 需要把网页链接转为 Markdown

## 功能矩阵

### A. 去 AI 痕迹（人话润色）

检测并修复以下 AI 写作特征：
- 夸大的象征意义、"在XXX的世界里"
- 宣传性语言
- -ing 肤浅分析
- 模糊归因（"很多人说"）
- 破折号过度使用
- 三段式法则（"第一..第二..第三"）
- AI 高频词（"值得注意的是"、"不得不承认"）
- 否定式排比
- 过多的连接性短语

**使用方法**：直接调 humanizer-zh skill，或按上述 checklist 手动改。
**强制规则**：每篇脚本写完必须走一轮润色。

### B. Markdown 格式化（baoyu-format-markdown）

输入纯文本或粗糙 Markdown，输出带以下格式的干净文章：
- YAML frontmatter（title、summary）
- 层级标题
- 加粗/列表/代码块
- 文章摘要

**输出文件**：`{原文件名}-formatted.md`

### C. 翻译（baoyu-translate）

三种模式：
- **快速**：直译，保留格式
- **普通**：通顺，自然表达
- **精翻**：精细翻译，保留语气风格

支持中 ↔ 英互译。提供 URL 则先抓取再翻译。

### D. URL → Markdown（baoyu-url-to-markdown）

```bash
baoyu-fetch <url>  # 用 Chrome CDP 抓取并转 Markdown
```

内置适配器：X/Twitter、YouTube transcripts、Hacker News。
通用页面用 Defuddle 引擎。
处理登录/CAPTCHA 时可配置等待模式。

## 红线

- [ ] 润色时不改变原文核心信息和语气
- [ ] 翻译时保持术语一致性（宅咪/ZHAIMI 不译）
- [ ] URL 转 Markdown 失败时退到 browser-automation 截图
