# 宅咪项目技能索引

> 品牌：宅咪 ZHAIMI | 宠物上门洗护 @ 临港万达周边
> 项目根：`C:\Users\11390\Desktop\宅咪小红书笔记\`
> 冷启动阶段：0→1

## 技能目录（单职责拆分）

```
zhaimi/
├── research/                                ← researcher profile
│   ├── topic-scoring/         只做：选题评分
│   ├── competitor-analysis/   只做：竞品分析
│   ├── viral-deconstruct/     只做：爆款拆解
│   └── comment-insight/       只做：评论洞察
├── vidiator/                                ← vidiator profile
│   ├── script-writing/        只做：脚本写作
│   └── visual-design/         只做：视觉设计
├── creator/                                 ← creator profile
│   └── title-cover-check/     只做：标题封面检查
└── orchestrator/                            ← default profile
    └── content-orchestrator/  只做：调度编排
```

## 工作流总览

```
用户说"帮我做一条笔记"
        │
        ▼
┌─────────────────────────────┐
│  orchestrator/ 内容调度器    │  ← 总入口
│  决定走模式A/B/C             │
└──────────┬──────────────────┘
           │
    ┌──────┼──────┐
    ▼      ▼      ▼
 模式A  模式B  模式C
 完整   爆款   评论→
 创作   改造   选题
           │
           ▼
    ┌──────────────┐
    │  research/   │  ← 选题/竞品/拆解/洞察
    │  vidiator/   │  ← 脚本/视觉
    │  creator/    │  ← 检查
    └──────┬───────┘
           ▼
      完整内容包
```

## 依赖的公用技能

| 项目技能 | 依赖的 share 技能 |
|----------|-------------------|
| research/* | `../../share/web-scraping/` + `../../share/research/` + `../../share/content-processing/` |
| vidiator/* | `../../share/browser-automation/` + `../../share/image-gen/` + `../../share/content-processing/` |
| creator/* | `../../share/content-processing/` |
| orchestrator/* | 以上全部 |

## 核心约束（全项目通用）

1. [ ] 性别中性（不写"姐妹/妈妈/女主人"）
2. [ ] 团队人数弹性措辞（不写"就我俩/两姐妹"）
3. [ ] 节日/节点以 `date` 实际输出为准
4. [ ] 钩子方向 = 咨询导向（非收藏/娱乐）
5. [ ] 0→1 阶段不立"不接的单/拒客原则"
6. [ ] 出图必须读品牌视觉规范
7. [ ] 脚本必须走 humanizer-zh 润色
8. [ ] prompt .md 与图片同目录
9. [ ] 私域钩子 = 评论扣【区域+宠物】

## 与宅电项目的对比

| 维度 | 宅咪 ZHAIMI | 宅电 ZHAIDIAN |
|------|-------------|---------------|
| 服务 | 上门洗护 | 上门电竞房搭建 |
| 主色 | 暖橙治愈 | 暗色科技 |
| 痛点 | 怕猫应激/嫌出门麻烦 | 没空间/不会选设备/怕踩坑 |
| 私域钩子 | 评论扣【区域+宠物】 | 评论扣【城市+房型】 |
| 决策周期 | 短（单次服务） | 长（万元级改造） |
| 内容节奏 | 周更 3-4 篇 | 月更 1-2 篇 |
