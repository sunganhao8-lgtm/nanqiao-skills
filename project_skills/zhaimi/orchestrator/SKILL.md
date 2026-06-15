---
name: zhaimi-orchestrator
description: 宅咪内容创作调度器。用户唯一入口，一句"帮我做条笔记"走完从选题到成品全流程。自动判断走模式A/B/C，路由到对应 profile 的子 skill。
version: 1.0.0
tags: [宅咪, 调度器, 编排, 入口]
references:
  - ../research/SKILL.md
  - ../vidiator/SKILL.md
  - ../creator/SKILL.md
---

# zhaimi-orchestrator — 内容创作调度器

> 适用 profile：default（调度者）
> 这是用户**不用记 8 个子 skill 名**的唯一入口。
> "帮我做一条笔记" → 走这里。

## 触发条件

**任何**以下说法触发：
- "帮我做一条笔记 / 出一篇内容"
- "今天发什么 / 写一篇关于 X 的"
- "帮我创作一条内容"
- 给了选题方向要求出完整内容
- 给了爆款链接要求仿写
- 给了评论/弹幕要求出笔记

## 调度模式（自动判断）

### 模式 A：完整创作（从零到一）

用户给了一个模糊方向/关键词时触发。

```
用户输入方向
  │
  ▼
① 调 researcher profile → zhaimi-research topic-scoring
  │
  ├─ ≥ 35/50 → 自动继续
  └─ < 35/50 → "评分只有 X/50，原因是 Y。继续吗？"
  │
  ▼
② 调 vidiator profile → zhaimi-vidiator script-writing
  │
  ▼
③ 调 vidiator profile → humanizer-zh 润色（content-processing）
  │
  ▼
④ 调 creator profile → zhaimi-creator title-cover-check
  │
  ▼
⑤ 调 vidiator profile → zhaimi-vidiator visual-design
  │
  ▼
交付：完整内容包
```

### 模式 B：爆款改造

用户给了爆款 URL/描述时触发。

```
① 调 researcher → viral-deconstruct（拆爆款公式）
② 调 vidiator → script-writing（用拆解结果写宅咪版）
③ humanizer-zh
④ creator → title-cover-check
⑤ vidiator → visual-design
```

### 模式 C：评论→选题

用户给评论/弹幕原文时触发。

```
① 调 researcher → comment-insight（分类→选题池）
② 出 Top 3，用户选一个
③ vidiator → script-writing
④ humanizer-zh
⑤ creator → title-cover-check
⑥ vidiator → visual-design
```

## 执行要点

### 并行调度
- topic-scoring 和 competitor-analysis **可并行**
- 互不依赖的子任务用 `delegate_task` 的 `tasks` 数组并行

### 中间交付
- 评分 ≥ 35/50 **不墨迹直接继续**
- 评分 < 35/50 才问：**一句话问，不多解释**
- 每一步做完说结果（"评分过了"→"脚本写好了"→"检查完了"）
- 最终交付**完整内容包**，不是一堆分散文件

### 输出目录
```
宅咪小红书笔记/
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
- [ ] 发布清单（checklist）
- [ ] 未完成的出图标注"待出图"并说明原因

## 红线（最重要）

- [ ] **我不自己写脚本/文案** → 派给 vidiator profile
- [ ] **分析类任务派给 researcher** → 我不做"标题抄写员"
- [ ] **评分够就不问用户要不要继续** → 直接走
- [ ] **交付完整版** → 不给选项让用户选
- [ ] **香肠式汇报** → 做完一步说一步，"我将要做 X" 扣分
- [ ] **最终输出目录规范** → prompt 和图片同目录
