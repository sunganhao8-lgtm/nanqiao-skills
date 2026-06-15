---
name: zhaidian-content-orchestrator
description: 宅电内容创作调度器。一键编排从选题到发布的全流程。接收选题→评分→竞品参考→脚本写作→视觉设计→标题检查→出图，产出完整内容包。它是所有 zhaidian-* 技能的顶层调度入口。
version: 1.0.0
tags: [宅电, 调度器, 编排, 入口]
---

# 宅电内容创作调度器

> **用户唯一入口**。"帮我做一条笔记" → 走这里。

## 触发条件

- "帮我做一条笔记 / 出一篇内容"
- "今天发什么 / 写一篇关于 X 的"
- 用户给选题方向要求出完整内容
- 用户给爆款链接要求仿写
- 用户给评论/弹幕要求出笔记

## 调度模式（自动判断）

### 模式 A：完整创作（从零到一）

用户给了一个模糊方向/关键词时触发。

```
用户输入方向
  │
  ▼
① zhaidian-topic-scoring → 出评分报告
  │
  ├─ ≥ 35/50 → 自动继续
  └─ < 35/50 → "评分 X/50，原因是 Y。继续吗？"
  │
  ▼
② zhaidian-competitor-analysis（可选）
  │
  ▼
③ zhaidian-script-writing → 出完整文案/脚本
  │
  ▼
④ humanizer-zh → 去 AI 痕迹（../../share/content-processing/）
  │
  ▼
⑤ zhaidian-title-cover-check → 标题+正文检查
  │
  ▼
⑥ zhaidian-visual-design → 出图方案 + prompt
  │
  ▼
交付：完整内容包
```

### 模式 B：爆款改造

用户给了爆款 URL/描述时触发。

```
① zhaidian-viral-deconstruct（拆爆款公式）
② zhaidian-script-writing（用拆解结果写宅电版）
③ humanizer-zh
④ zhaidian-title-cover-check
⑤ zhaidian-visual-design
```

### 模式 C：评论→选题

用户给了评论/弹幕原文时触发。

```
① zhaidian-comment-insight（分类→选题池）
② 出 Top 3，让用户选一个
③ zhaidian-script-writing
④ humanizer-zh
⑤ zhaidian-title-cover-check
⑥ zhaidian-visual-design
```

## 执行要点

### 并行调度
- topic-scoring 和 competitor-analysis **可并行**
- 互不依赖的子任务用 `delegate_task` 的 `tasks` 数组并行

### 中间交付
- 评分 ≥ 35/50 **不墨迹直接继续**
- 评分 < 35/50 才问：**一句话问，不多解释**
- 每一步做完说结果（"评分过了"→"脚本写好了"→"检查完了"）
- 最终交付**完整内容包**

### 输出目录

```
宅电项目根（待用户指定）/
├── 笔记-{YYYY-MM-DD}-{选题关键词}/
│   ├── inputs/           ← 评分/分析报告
│   ├── work/             ← 脚本草稿
│   ├── prompts/          ← prompt + 图（同目录）
│   ├── review/           ← 检查报告
│   ├── outputs/          ← 完整内容包
│   └── final/            ← 终稿
```

### 交付物清单
最终交付必须包含：
- [ ] 标题（推荐 + 2 备选 + 理由）
- [ ] 正文（≤ 500 字，已润色）
- [ ] 视频分镜（如果是视频形式）
- [ ] 配图方案（9 图/封面的视觉方案 + prompt）
- [ ] 评论区预埋话术（5 条）
- [ ] 发布清单
- [ ] 未完成的出图标注"待出图"

## 宅电业务边界（调度器必查）

- [ ] 不写性别限定词
- [ ] 不写"两人团队/小工作室"
- [ ] 私域钩子 = 评论扣【城市+房型】
- [ ] 不写"我们不接小单/拒客原则"（0→1 阶段）
- [ ] 关键数字要具体（"2K 240Hz" 而不是"高端"）
- [ ] 玩家只画背影/侧影

## 红线

- [ ] **我不自己写脚本/文案** → 派给对应 profile
- [ ] **分析类任务派给 researcher** → 不做"标题抄写员"
- [ ] **评分够就不问用户** → 直接走
- [ ] **交付完整版** → 不给选项让用户选
- [ ] **香肠式汇报** → 做完一步说一步
- [ ] **最终输出目录规范** → prompt 和图片同目录
