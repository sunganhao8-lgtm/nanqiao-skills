# project_skills/ — 项目专用技能

每个项目子目录按 Hermes profile 分目录存放技能，只加载当前 profile 需要的技能。

```
project_skills/
├── README.md                     ← 本文件
│
├── zhaimi/                       ← 宅咪 ZHAIMI（宠物上门洗护 @ 临港万达）
│   ├── README.md
│   ├── research/                 ← 选题/竞品/拆解/洞察
│   ├── vidiator/                 ← 脚本/视觉
│   ├── creator/                  ← 检查
│   └── orchestrator/             ← 一刀入口
│
└── zhaidian/                     ← 宅电 ZHAIDIAN（上门电竞房搭建）
    ├── README.md                 ← 宅电索引
    ├── research/
    ├── vidiator/
    ├── creator/
    └── orchestrator/
```

## profile 加载配置

各 profile 的 config.yaml 中加载对应目录：

**researcher profile**：
```yaml
skills:
  - nanqiao_skills/share/web-scraping
  - nanqiao_skills/share/research
  - nanqiao_skills/share/content-processing
  - nanqiao_skills/project_skills/zhaimi/research
  - nanqiao_skills/project_skills/zhaidian/research
```

**vidiator profile**：
```yaml
skills:
  - nanqiao_skills/share/browser-automation
  - nanqiao_skills/share/image-gen
  - nanqiao_skills/share/content-processing
  - nanqiao_skills/project_skills/zhaimi/vidiator
  - nanqiao_skills/project_skills/zhaidian/vidiator
```

**creator profile**：
```yaml
skills:
  - nanqiao_skills/share/content-processing
  - nanqiao_skills/share/design-visual
  - nanqiao_skills/project_skills/zhaimi/creator
  - nanqiao_skills/project_skills/zhaidian/creator
```

**default profile**（调度者）：
```yaml
skills:
  - nanqiao_skills/share/*
  - nanqiao_skills/project_skills/*/orchestrator
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

## 新增项目

1. 复制 `zhaimi/` 整个目录为新项目名（如 `zhaidian/`）
2. 改 `README.md` 里的品牌定位/业务边界/视觉规范
3. 改 4 个 profile 子目录里的 SKILL.md 业务描述
4. 更新 `project_skills/README.md` 的索引
5. 更新各 profile 的 config.yaml 加 softlink
