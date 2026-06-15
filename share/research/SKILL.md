---
name: research
description: 调研工具集。学术论文搜索、RSS 博客监控、B 站竞品分析、知识库构建。合并自 arxiv、blogwatcher、bilibili-competitive-benchmarking、llm-wiki。
version: 1.0.0
---

# research — 调研

## 触发条件

- "搜索论文 / 查文献"
- "监控博客更新"
- "分析 B 站 UP 主 / 竞品"
- "建知识库"

## 功能

### A. 学术论文（arxiv）
- 按关键词/作者/分类搜索 arXiv
- 输出标题/摘要/链接

### B. RSS 监控（blogwatcher）
```bash
blogwatcher list    # 查看订阅源
blogwatcher update  # 拉取更新
```
监控 AI Agent / MCP / 开源项目动态。

### C. B 站竞品（bilibili-competitive-benchmarking）
- 赛道扫描 → UP 主拆解 → 视频解构 → 爆款公式 → 脚本产出
- 包含反爬下载和数据分析

### D. LLM 知识库（llm-wiki）
Karpathy 风格互链 Markdown KB 的构建/查询。
