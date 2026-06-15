---
name: social-publish
description: 多平台社交发布。微信公众号、X(Twitter)、微博内容发布。合并自 baoyu-post-to-wechat、baoyu-post-to-x、baoyu-post-to-weibo。
version: 1.0.0
---

# social-publish — 社交发布

## 触发条件

- "发布到公众号 / 发微博 / 发 X"
- "帮我发一篇 / 发帖"

## 发布通道

### A. 微信公众号（文章 / 贴图）

**文章发布**：
- 输入：Markdown → 自动转 HTML
- 外链 → 自动转底部引用（WeChat 兼容）
- 通过 API 或 Chrome CDP 发布

**贴图发布（原 图文）**：
- 多张图片 + 文字描述
- 通过 Chrome CDP 操作公众号后台

### B. X (Twitter)

**普通推文**：文字 + 图片/视频
**X Articles**：长文 Markdown 发布
**方式**：Chrome CDP（优先）/ API 备选

### C. 微博

**普通微博**：文字 + 图片/视频
**头条文章**：Markdown 长文
**方式**：Chrome CDP

## 注意

- 小红书不在此 skill 中（小红书通过手动/App 发布，或后期接入）
- 微信公众号文章发布前检查格式兼容性
