# nanqiao_skills — 个人技能体系

> 个人 Hermes 技能体系。公用工具 + 项目技能（宅咪 / 宅电），单职责拆分。
> 路径：`<hermes_skills>/nanqiao_skills/`
> 仓库：https://github.com/sunganhao8-lgtm/nanqiao-skills

---

## 项目文档

| 文档 | 用途 |
|------|------|
| [README.md](./README.md) | 本文件 — 顶层架构 |
| [CHANGELOG.md](./CHANGELOG.md) | 所有重要变更记录（必看） |
| [CONTRIBUTING.md](./CONTRIBUTING.md) | 贡献者规范（必读） |
| [PROFILE_STRUCTURE.md](./PROFILE_STRUCTURE.md) | profile 与 skill 的对应关系 |
| [TEMPLATE-SKILL.md](./TEMPLATE-SKILL.md) | 新技能模板 |

---

## 结构总览

```
nanqiao_skills/
├── README.md
├── CHANGELOG.md
├── CONTRIBUTING.md
├── PROFILE_STRUCTURE.md
├── TEMPLATE-SKILL.md
├── .gitignore
│
├── share/                                ← 11 个公用技能
│   ├── README.md
│   ├── browser-automation/               ← Chrome CDP + MCP + 豆包
│   ├── image-gen/                        ← 豆包/即梦/MiniMax 出图
│   ├── content-processing/               ← 润色/翻译/格式化
│   ├── web-scraping/                     ← 抓取/搜索
│   ├── social-publish/                   ← 公众号/X/微博
│   ├── media-audio/                      ← YouTube/GIF
│   ├── design-visual/                    ← 图表/信息图/PPT
│   ├── research/                         ← 学术/RSS/B站
│   ├── productivity/                     ← 地图/笔记/Kanban
│   ├── observing/                        ← 监控/看门狗
│   └── dev-tools/                        ← GitHub/Codex
│
└── project_skills/                       ← 项目专用技能
    ├── README.md
    ├── zhaimi/                           ← 宅咪 ZHAIMI（宠物上门洗护）
    │   ├── README.md
    │   ├── research/{topic-scoring, competitor-analysis, viral-deconstruct, comment-insight}/
    │   ├── vidiator/{script-writing, visual-design}/
    │   ├── creator/title-cover-check/
    │   └── orchestrator/content-orchestrator/
    │
    └── zhaidian/                         ← 宅电 ZHAIDIAN（上门电竞房搭建）
        ├── README.md
        ├── research/...
        ├── vidiator/...
        ├── creator/...
        └── orchestrator/...
```

## 设计原则

1. **单职责**：一个 SKILL.md 只做一件事。
2. **按 profile 分目录**：每个 Hermes profile 有独立的 skill 子目录。
3. **公用/项目分离**：`share/` 不依赖具体项目，`project_skills/` 放业务逻辑。
4. **相对路径引用**：项目技能通过 `../../share/xxx/` 引用公用技能，可移植。
5. **软链接管理**：profile 下的 `skills/` 目录都是软链接，零复制。

## 现有项目

| 项目 | 私域钩子 | 主色 | 决策周期 | 内容节奏 |
|------|----------|------|----------|----------|
| 宅咪 ZHAIMI | 评论扣【区域+宠物】 | 暖橙治愈 | 短（单次） | 周更 3-4 篇 |
| 宅电 ZHAIDIAN | 评论扣【城市+房型】 | 暗色科技 | 长（万元级） | 月更 1-2 篇 |

## 快速开始

- 想用某个 skill → 看 [PROFILE_STRUCTURE.md](./PROFILE_STRUCTURE.md)
- 想新增/修改 skill → 看 [CONTRIBUTING.md](./CONTRIBUTING.md)
- 想看最近改了啥 → 看 [CHANGELOG.md](./CHANGELOG.md)
- 想写新 skill → 复制 [TEMPLATE-SKILL.md](./TEMPLATE-SKILL.md)

## 维护 checklist（每月一次）

- [ ] 检查软链是否断裂
- [ ] 检查 CHANGELOG 是否有堆积
- [ ] 验证私域钩子格式未漂移
- [ ] 验证 share 技能被正确引用
