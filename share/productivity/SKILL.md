---
name: productivity
description: 效率工具集。地图/笔记/OCR/PDF/日历/看板/工作区。合并自 maps、notion、airtable、ocr-and-documents、nano-pdf、obsidian、google-workspace、kanban-swarm-workflow、project-workspace-manager、hermes-memory、profile-llm-routing。
version: 1.0.0
---

# productivity — 效率工具

## 触发条件

- 需要地图查询 / 导航
- 需要管理 Notion / Airtable
- OCR 提取文字
- 编辑 PDF
- 日历/邮件
- Kanban 任务管理
- 项目工作区搭建
- 记忆/配置迁移

## 功能索引

| 功能 | 触发词 | 工具 |
|------|--------|------|
| 地理编码/路线 | "查地图 / 导航 / POI" | maps: OSM + OSRM |
| Notion 操作 | "写 Notion / 查数据库" | notion + ntn CLI |
| Airtable 操作 | "查 Airtable / 更新记录" | airtable: REST API + curl |
| OCR 文字提取 | "提取图片文字 / PDF文字" | ocr-and-documents: pymupdf + marker-pdf |
| PDF 编辑 | "改PDF / 修PDF文字" | nano-pdf CLI |
| Obsidian 笔记 | "查笔记 / 写笔记" | obsidian: read/search/create/edit |
| Google 套件 | "查日历 / 发邮件 / 写文档" | google-workspace: gws CLI |
| Kanban 并行 | "派并行任务 / 多个 worker" | kanban-swarm-workflow |
| 项目工作区 | "建项目 / 创建工作区" | project-workspace-manager: 5 目录结构 |
| 记忆迁移 | "迁移记忆 / 配置搬家" | hermes-memory |
| LLM 路由 | "给 profile 配模型" | profile-llm-routing |

### 项目工作区标准结构（project-workspace-manager）

所有出产物类 Kanban 任务必须确立 PROJECT_ROOT + 5 目录：
```
PROJECT_ROOT/
├── inputs/    ← 输入资料
├── work/      ← 工作文件
├── outputs/   ← 产出
├── review/    ← 审核
└── final/     ← 终稿
```

宅咪项目根：`C:\Users\11390\Desktop\宅咪小红书笔记\`
