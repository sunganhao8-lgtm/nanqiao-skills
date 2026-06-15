---
name: media-audio
description: 音视频和动图处理。YouTube 字幕/转录、封面、GIF 搜索。合并自 youtube-content、baoyu-youtube-transcript、gif-search。
version: 1.0.0
---

# media-audio — 音视频处理

## 触发条件

- "下载 YouTube 字幕 / 转录"
- "搜索 GIF"
- "提取视频封面"

## 功能

### A. YouTube 视频

**获取字幕**：
```bash
# baoyu-youtube-transcript
# 输入 URL 或 video ID
# 支持：多语言、翻译、章节、说话人识别
```
缓存原始数据，支持快速重格式化。

**内容总结**（youtube-content skill）：
- 视频摘要
- 转换为推文/博客/Newsletter

### B. GIF 搜索

```bash
# gif-search via Tenor API
curl -s -H "Content-Type: application/json" \
  "https://g.tenor.com/v1/search?q=KEYWORD&key=API_KEY"
```

### C. 封面图提取

YouTube 视频封面 URL 格式：
`https://img.youtube.com/vi/{VIDEO_ID}/maxresdefault.jpg`
