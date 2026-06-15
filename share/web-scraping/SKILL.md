---
name: web-scraping
description: 网页抓取工具集。web_extract / web_search / Chrome CDP / 登录态页面抓取。合并自 web-access、read-private-wiki-via-browser-mcp、baoyu-url-to-markdown。
version: 1.0.0
---

# web-scraping — 网页抓取

## 触发条件

- 需要搜索或抓取网页内容
- 需要抓取登录态页面（飞书 docx、小红书、抖音）
- web_extract 抓不到时退到浏览器方案

## 抓取优先级

### ① web_search + web_extract（首选，最快）

```python
# web_extract 支持普通网页和 PDF
from hermes_tools import web_search, web_extract

web_search(query="宠物上门洗护 小红书", limit=5)
web_extract(urls=["https://example.com"])
```

**适用**：公开网页、PDF、不需要登录的页面
**限制**：封禁严格的反爬站、所有登录态页面

### ② Chrome CDP MCP（登录态页面）

通过 `../browser-automation/SKILL.md` 的 Chrome 远程调试功能。

**适用**：小红书、抖音、飞书 docx、微信公众号后台
**流程**：mcp_chrome_devtools_new_page → 等待加载 → take_snapshot → 取数据

### ③ 飞书/私有文档专用（read-private-wiki）

对于 web_extract 抓不到的私有文档/飞书 docx：
- 先用 Chrome MCP 打开文档
- 用「sidebar 锚点跳转 + 内部滚动容器 + 截图 + vision_analyze」组合拳
- 不用滚动整个 DOM 抓文本（飞书 docx 文字嵌在图里）

## 工具选择表

| 场景 | 工具 | 示例 |
|------|------|------|
| 搜索公开信息 | web_search | "宠物上门洗护" |
| 抓公开网页 | web_extract | 小红书公开笔记 |
| 抓 PDF | web_extract(URL) | arXiv 论文 |
| 登录态页面 | MCP CDP | 飞书文档、小红书后台 |
| 反爬站 | MCP CDP + 截图 | 某度文库 |
| 直播弹幕 | MCP CDP JS 注入 | 抖音直播间 |

## 红线

- [ ] 能 web_extract 就不用浏览器（更快更省）
- [ ] 登录态抓取优先用已有 Cookie 的 Chrome（DebugProfile）
- [ ] 不暴力爬取（频率限制 + 反爬）
- [ ] 抓取结果标注是否完整（有无被截断）
