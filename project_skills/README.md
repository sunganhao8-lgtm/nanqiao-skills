# project_skills/ — 项目专用技能

每个项目子目录按 Hermes profile 分目录存放技能，只加载当前 profile 需要的技能。

```
project_skills/
├── README.md                     ← 本文件
│
└── zhaimi/                       ← 宅咪项目（宠物上门洗护 @ 临港万达）
    ├── README.md                 ← 宅咪索引
    ├── research/                 ← researcher profile：选题/竞品/拆解/洞察
    ├── vidiator/                 ← vidiator profile：脚本/视觉
    ├── creator/                  ← creator profile：标题封面检查
    └── orchestrator/             ← default/dispatcher：创作调度器
```

## profile 加载配置

各 profile 的 config.yaml 中加载对应目录：

**researcher profile**：
```yaml
# ~/.hermes/profiles/researcher/config.yaml
skills:
  - nanqiao_skills/share/web-scraping
  - nanqiao_skills/share/research
  - nanqiao_skills/project_skills/zhaimi/research
```

**vidiator profile**：
```yaml
skills:
  - nanqiao_skills/share/browser-automation
  - nanqiao_skills/share/image-gen
  - nanqiao_skills/project_skills/zhaimi/vidiator
```

**creator profile**：
```yaml
skills:
  - nanqiao_skills/share/content-processing
  - nanqiao_skills/share/design-visual
  - nanqiao_skills/project_skills/zhaimi/creator
```

**default profile**（调度者）：
```yaml
skills:
  - nanqiao_skills/share/*
  - nanqiao_skills/project_skills/zhaimi/orchestrator
```

## 相对路径引用约定

项目 SKILL.md 引用公用技能时统一用相对路径：

```markdown
## 参考
- 浏览器操作 → `../../share/browser-automation/SKILL.md`
- 图片生成 → `../../share/image-gen/SKILL.md`
- 内容润色 → `../../share/content-processing/SKILL.md`
```

这样移动整个 `nanqiao_skills/` 目录时引用不失效。
