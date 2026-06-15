# PROFILE_STRUCTURE — Profile 与 Skill 对应关系

> 维护目的：让任何接手的人能一眼看清「某个 profile 能用什么 skill」+「某个 skill 会被谁用到」。
> 上次更新：2026-06-15

---

## 1. Profile 速查

| Profile | 用途 | 软链数 | 位置 |
|---------|------|--------|------|
| **researcher** | 选题/竞品/拆解/洞察 | 11 | `~/.hermes/profiles/researcher/skills/` |
| **vidiator** | 脚本/视觉/出图 | 7 | `~/.hermes/profiles/vidiator/skills/` |
| **creator** | 标题封面检查 | 4 | `~/.hermes/profiles/creator/skills/` |
| **default** | 调度编排 | — | 不直接挂 skill，依赖 `delegate_task` 派给其他 profile |

**注意**：default profile 不直接挂 skill，而是通过 `delegate_task` 命令调用其他 profile。所以它不需要自己的 skill 目录。

---

## 2. researcher profile

| Skill | 路径 | 来自 | 用途 |
|-------|------|------|------|
| web-scraping | `share/web-scraping/` | share | 网页抓取/搜索 |
| research-utils | `share/research/` | share | 学术/RSS/B站 |
| content-processing | `share/content-processing/` | share | 润色/翻译/格式化 |
| zhaimi-topic-scoring | `project_skills/zhaimi/research/topic-scoring/` | zhaimi | 选题评分 |
| zhaimi-competitor-analysis | `project_skills/zhaimi/research/competitor-analysis/` | zhaimi | 竞品分析 |
| zhaimi-viral-deconstruct | `project_skills/zhaimi/research/viral-deconstruct/` | zhaimi | 爆款拆解 |
| zhaimi-comment-insight | `project_skills/zhaimi/research/comment-insight/` | zhaimi | 评论洞察 |
| zhaidian-topic-scoring | `project_skills/zhaidian/research/topic-scoring/` | zhaidian | 选题评分 |
| zhaidian-competitor-analysis | `project_skills/zhaidian/research/competitor-analysis/` | zhaidian | 竞品分析 |
| zhaidian-viral-deconstruct | `project_skills/zhaidian/research/viral-deconstruct/` | zhaidian | 爆款拆解 |
| zhaidian-comment-insight | `project_skills/zhaidian/research/comment-insight/` | zhaidian | 评论洞察 |

**调用方式**：
```bash
hermes -z "帮我看看这个选题值不值得做" chat --profile researcher
```

---

## 3. vidiator profile

| Skill | 路径 | 来自 | 用途 |
|-------|------|------|------|
| browser-automation | `share/browser-automation/` | share | Chrome CDP + MCP + 豆包 |
| wechat-tools | `share/wechat-tools/` | share | 微信本地工具（导出+风格分析） |
| image-gen | `share/image-gen/` | share | AI 图片生成（豆包/即梦/MiniMax） |
| content-processing | `share/content-processing/` | share | 润色/翻译/格式化 |
| zhaimi-script-writing | `project_skills/zhaimi/vidiator/script-writing/` | zhaimi | 脚本写作 |
| zhaimi-visual-design | `project_skills/zhaimi/vidiator/visual-design/` | zhaimi | 视觉设计 |
| zhaidian-script-writing | `project_skills/zhaidian/vidiator/script-writing/` | zhaidian | 脚本写作 |
| zhaidian-visual-design | `project_skills/zhaidian/vidiator/visual-design/` | zhaidian | 视觉设计 |

**调用方式**：
```bash
hermes -z "帮我写一篇上门洗猫的小红书" chat --profile vidiator
```

---

## 4. creator profile

| Skill | 路径 | 来自 | 用途 |
|-------|------|------|------|
| content-processing | `share/content-processing/` | share | 润色/翻译/格式化 |
| design-visual | `share/design-visual/` | share | 图表/PPT/HTML |
| zhaimi-title-cover-check | `project_skills/zhaimi/creator/title-cover-check/` | zhaimi | 标题封面检查 |
| zhaidian-title-cover-check | `project_skills/zhaidian/creator/title-cover-check/` | zhaidian | 标题封面检查 |

**调用方式**：
```bash
hermes -z "帮我检查这个标题和封面" chat --profile creator
```

---

## 5. default profile（调度者）

不直接挂 skill。**所有用户任务都先走 default**，由 default 决定派给哪个 profile。

**典型工作流**：

```
用户：帮我做一条关于 XXX 的笔记
  │
  ▼
default profile
  │
  ├─ 评分（→ researcher）
  ├─ 写脚本（→ vidiator）
  ├─ 润色（→ vidiator + content-processing）
  ├─ 检查（→ creator）
  └─ 出图（→ vidiator + image-gen）
```

**`nanqiao_skills/project_skills/<项目>/orchestrator/content-orchestrator/`** 是 default profile 唯一的项目 skill，但它本身不是"技能执行者"，而是**调度器文档**——告诉 default profile 怎么编排。

---

## 6. 软链接脚本

修改 profile 软链用 PowerShell 脚本：

```
C:\Users\11390\AppData\Local\Temp\update_profile_links.ps1
```

**当前脚本维护的映射**（如有变更请同时更新本文件 §2-§5）：

| profile | 软链 |
|---------|------|
| researcher | 11 个（见 §2） |
| vidiator | 7 个（见 §3） |
| creator | 4 个（见 §4） |

---

## 7. 共享 share 技能的去重

注意：content-processing 同时被 3 个 profile 引用、web-scraping 只被 researcher 用等。这不是冗余——是"按需分配"。

**冗余判定**：
- 同一份 SKILL.md 在多个 profile 出现 = ✅ OK（软链接，同一份文件）
- 多份内容雷同的 SKILL.md 出现在不同目录 = ❌ 违规，必须合并

**验证脚本**：
```bash
# 检查是否有两份内容雷同的 skill
find /c/Users/11390/AppData/Local/hermes/skills/nanqiao_skills -name "SKILL.md" | \
  xargs md5sum | sort | uniq -d -w 32
```
（输出为空 = 无冗余）

---

## 8. 异常处理

| 现象 | 原因 | 修复 |
|------|------|------|
| `hermes skills list` 看不到某 skill | 软链断了 | `find ... -type l ! -exec test -e {} \; -delete` 后重跑 update_profile_links.ps1 |
| Profile 加载到错的 skill | 软链指向错路径 | 检查 `<profile>/skills/` 下的链接目标 |
| 某 skill 被 0 个 profile 引用 | 可能是 dead code | 在 CHANGELOG 标 deprecated 或删 |
| default profile 派任务派错 profile | 调度器 skill 配置错 | 检查 `project_skills/<项目>/orchestrator/content-orchestrator/SKILL.md` 的路由逻辑 |

---

## 9. 添加新 profile 的流程

1. 选名字：kebab-case，能体现用途（如 `translater` / `coder`）
2. 跑脚本创建 profile + 软链占位：
   ```bash
   hermes profile create <name>
   ```
3. 在本文件 §2/§3/§4 增加对应表格
4. 在 CHANGELOG 写 `Added`
5. 在 README 顶层索引添加

---

## 10. 添加新项目的流程

1. `cp -r project_skills/<template>/ project_skills/<new-project>/`
2. 替换所有 skill 名称中的项目前缀
3. 修改私域钩子格式
4. 修改品牌视觉规范
5. 在本文件 §2-§4 添加新项目的 skill
6. 在 CHANGELOG 写 `Added`
7. update_profile_links.ps1 加上新项目的软链
8. 在 `project_skills/README.md` 索引
