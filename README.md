# nanqiao_skills — 个人技能体系

> 路径：`<hermes_skills>/nanqiao_skills/`
> 备份：原 skills 目录已备份在 `skills.backup.20260615_101135/`

## 结构总览

```
nanqiao_skills/
├── README.md                         ← 本文件
│
├── share/                            ← 公用/工具型技能（不依赖具体项目）
│   ├── README.md                     ← share 索引
│   ├── browser-automation/           ← 浏览器自动化：Chrome CDP + MCP
│   ├── image-gen/                    ← 图片生成：豆包/即梦/MiniMax
│   ├── content-processing/           ← 内容处理：润色/翻译/格式化
│   ├── web-scraping/                 ← 网页抓取：通用爬取/登录态
│   ├── social-publish/               ← 社交发布：公众号/X/微博
│   ├── media-audio/                  ← 音视频：YouTube/音频/GIF
│   ├── design-visual/                ← 设计视觉：图表/信息图/PPT
│   ├── research/                     ← 调研：学术/RSS/竞品
│   ├── productivity/                 ← 效率：地图/笔记/OCR
│   ├── observing/                    ← 监控：变化检测/数据看门狗
│   └── dev-tools/                    ← 开发：Git/Codex/代码审查
│
└── project_skills/                   ← 项目专用技能
    ├── README.md                     ← 项目索引
    └── zhaimi/                       ← 宅咪项目
        ├── README.md                 ← 宅咪索引
        ├── research/                 ← researcher profile
        ├── vidiator/                 ← vidiator profile
        ├── creator/                  ← creator profile
        └── orchestrator/             ← default profile（调度器）
```

## 设计原则

1. **按 profile 分目录**：每个 Hermes profile 有独立的技能目录，只加载自己需要的技能
2. **公用/项目分离**：`share/` 放不依赖具体项目的工具技能，`project_skills/` 放业务逻辑
3. **相对路径引用**：项目技能通过相对路径 `../../share/xxx/SKILL.md` 引用公用技能
4. **同类合并**：相似功能合并成一个 SKILL.md，内部用标题/触发条件区分不同工具
5. **精简原则**：只保留宅咪内容创作 + 日常开发需要的技能，其余已移除

## 如何使用

各 profile 加载对应目录的技能：

- **dispatcher (default)** → `zhaimi/orchestrator/` + `share/` 中所有
- **researcher** → `zhaimi/research/` + `share/research/` + `share/web-scraping/`
- **vidiator** → `zhaimi/vidiator/` + `share/browser-automation/` + `share/image-gen/`
- **creator** → `zhaimi/creator/` + `share/content-processing/` + `share/design-visual/`

每个技能 SKILL.md 开头都有清晰的 **触发条件** 说明什么场景加载它。
