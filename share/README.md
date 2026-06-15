# share/ — 公用工具型技能

> 每个子目录是一个独立的 Hermes 技能（含 SKILL.md），可通过 `skill_view(name)` 加载。
> 所有技能不依赖具体项目，可被 `project_skills/` 下的技能通过相对路径引用。

| 目录 | 合并自 | 用途 |
|------|--------|------|
| browser-automation | chrome-devtools-mcp-doubao-fallback + doubao-image-gen-minimal + xhs-cover-doubao-workflow(核心) | 浏览器操作：Chrome CDP 9222、MCP、豆包官网出图 |
| image-gen | baoyu-image-gen + baoyu-cover-image + baoyu-xhs-images + baoyu-compress-image + dreamina-cli + comfyui | AI 图片生成：API/本地/网页端 |
| content-processing | humanizer-zh + baoyu-format-markdown + baoyu-translate + baoyu-url-to-markdown | 内容处理：去 AI 痕迹、格式化、翻译 |
| web-scraping | web-access + read-private-wiki-via-browser-mcp + xurl | 网页抓取：公开/登录态 |
| social-publish | baoyu-post-to-wechat + baoyu-post-to-x + baoyu-post-to-weibo + baoyu-wechat-summary | 多平台发布。注意：小红书不在此（手动发布） |
| media-audio | youtube-content + gif-search + baoyu-youtube-transcript | YouTube 字幕/音频、GIF 搜索 |
| design-visual | architecture-diagram + excalidraw + baoyu-diagram + baoyu-infographic + baoyu-slide-deck + baoyu-markdown-to-html + baoyu-article-illustrator + baoyu-comic | 设计产出：图表/信息图/PPT/漫画 |
| research | arxiv + blogwatcher + bilibili-competitive-benchmarking + llm-wiki | 调研：学术论文/RSS/B 站 |
| productivity | maps + notion + airtable + ocr-and-documents + nano-pdf + obsidian + google-workspace + kanban-swarm-workflow + project-workspace-manager + hermes-memory + profile-llm-routing | 效率工具 |
| observing | dogfood | 监控/变化检测：网页变更、数据看门狗 |
| dev-tools | github-* + codex + claude-code + opencode + hermes-agent + hermes-agent-skill-authoring + plan + spike + systematic-debugging + windows-git + codebase-inspection + requesting-code-review + simplify-code + test-driven-development | 开发工具链 |

## 引用方式

项目技能引用 share 技能时，在 SKILL.md 的 `references` 或 `related_skills` 中用相对路径：

```yaml
related_skills:
  - browser-automation   # 如果是 Hermes 全局发现
references:
  - ../../share/browser-automation/SKILL.md  # 相对路径引用
```
