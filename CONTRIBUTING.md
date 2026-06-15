# CONTRIBUTING — 贡献者指南

> 适用对象：所有维护 nanqiao_skills 的人（含未来的协作者、你自己 6 个月后回来接手）
> 目标：保证多人/多 AI 协助时，技能体系不出乱子。

---

## 0. 入坑前必读

- **README.md** — 顶层架构
- **CHANGELOG.md** — 每次改动必须追加
- **PROFILE_STRUCTURE.md** — profile 与 skill 的对应关系
- **TEMPLATE-SKILL.md** — 新技能的格式模版

---

## 1. 三大铁律

### 铁律 1：单职责

**一个 SKILL.md 只做一件事。**

不要做这种事：
```yaml
# ❌ 错误：把多个功能塞进一个 skill
name: content-everything
description: 选题+脚本+视觉+发布一体化
```

要做这种事：
```yaml
# ✅ 正确：每个 skill 单一职责
name: topic-scoring
description: 选题评分
```
```yaml
name: script-writing
description: 脚本写作
```

**判断标准**：能不能用一句话清晰说出"这个 skill 只做 X"？说不出来就拆。

### 铁律 2：相对路径引用

**项目技能通过相对路径引用 `share/` 下的公用技能。**

```yaml
references:
  - ../../share/image-gen/SKILL.md
  - ../../share/content-processing/SKILL.md
```

不要写绝对路径（`C:\Users\11390\...`），这样项目可移植。

### 铁律 3：修改即更档

**任何对 SKILL.md 的修改必须同步更新 CHANGELOG.md。**

详见 §3。

---

## 2. 目录结构规则

### 2.1 三层目录

```
nanqiao_skills/
├── share/             ← 公用工具（不依赖具体项目）
├── project_skills/    ← 项目专用
│   ├── <项目名>/
│   │   ├── README.md
│   │   ├── research/<子技能>/SKILL.md
│   │   ├── vidiator/<子技能>/SKILL.md
│   │   ├── creator/<子技能>/SKILL.md
│   │   └── orchestrator/<子技能>/SKILL.md
│   └── ...
└── README.md / CHANGELOG.md / CONTRIBUTING.md / PROFILE_STRUCTURE.md
```

### 2.2 新增 skill 流程

1. **决定归属**：
   - 跨项目通用 → `share/<分类>/<skill-name>/`
   - 某个项目专用 → `project_skills/<项目>/<profile>/<skill-name>/`

2. **目录命名**：kebab-case（小写 + 短横线），如 `topic-scoring`/`visual-design`

3. **复制模版**：`cp TEMPLATE-SKILL.md share/<分类>/<skill-name>/SKILL.md`

4. **填写 SKILL.md**：按模版的 5 个章节（触发条件 / 输入 / 工作流 / 输出 / 红线）

5. **更新引用**：上游 skill 加上 `related_skills` 引用

6. **更新 CHANGELOG**：在 `[Unreleased]` 下追加 `Added`

7. **更新 profile 软链接**（如适用）：见 §4

8. **git commit + push**：
   ```bash
   git add -A
   git commit -m "feat(share): add <skill-name>"
   git push
   ```

### 2.3 删除 skill 流程

**禁止直接删除**！必须先归档 30 天观察期：

1. 在 SKILL.md 顶部加 `status: deprecated (replaced by <new-skill>)`
2. 标注 `replacement: <new-skill>` 字段
3. 30 天后无引用再 `git rm`
4. CHANGELOG 写 `Removed`

**例外**：错别字修复、白名单/红线更正等不破坏接口的修改不需要走归档流程。

### 2.4 改名 / 拆分

- **改名**：新加一个、改 30 天后删旧的（保留 git history）
- **拆分**：见 §1 铁律 1，新建多个子目录 + 旧的标注 `deprecated (split into X, Y, Z)`

---

## 3. CHANGELOG 规范

### 3.1 格式

遵循 [Keep a Changelog 1.1.0](https://keepachangelog.com/zh-CN/1.1.0/)。

每次 commit 前在 `[Unreleased]` 段落加新条目：

```markdown
## [Unreleased]

### Added
- 新增 `share/dev-tools/skill-validator`：检查 SKILL.md 是否符合模版

### Changed
- `share/image-gen/SKILL.md`：补全豆包 Dreamina 4.0 支持

### Fixed
- `project_skills/zhaimi/orchestrator/content-orchestrator/SKILL.md`：私域钩子漏写【区域】
```

### 3.2 版本号规则

- **MAJOR**（如 2.0.0）：破坏性变更（删除/重命名/修改接口）
- **MINOR**（如 2.1.0）：新增 skill
- **PATCH**（如 2.0.1）：文档修正、错别字、不破坏接口的优化

发布时把 `[Unreleased]` 改名为新版本号 + 日期：

```markdown
## [2.1.0] - 2026-06-20
（之前的 Unreleased 内容移到这里）
```

### 3.3 commit 信息规范

格式：`<type>(<scope>): <subject>`

| type | 用途 |
|------|------|
| `feat` | 新增 skill |
| `fix` | 修复错误 |
| `docs` | 仅文档变更 |
| `refactor` | 重构（不新增不修复） |
| `chore` | 杂项（gitignore 等） |

scope 示例：`share` / `zhaimi` / `zhaidian` / `docs`

示例：
```
feat(share): add skill-validator
fix(zhaimi/script-writing): 私域钩子漏写【区域】
docs: 更新 CHANGELOG
```

---

## 4. Profile 软链接管理

### 4.1 当前结构

每个 profile 目录下 `skills/` 都是软链接，指向 `nanqiao_skills/` 里具体子技能：

| profile | 软链数 | 软链列表 |
|---------|--------|----------|
| researcher | 11 | web-scraping, research-utils, content-processing + zhaimi research 4 + zhaidian research 4 |
| vidiator | 7 | browser-automation, image-gen, content-processing + zhaimi vidiator 2 + zhaidian vidiator 2 |
| creator | 4 | content-processing, design-visual + zhaimi creator 1 + zhaidian creator 1 |

### 4.2 同步脚本

修改 `nanqiao_skills/` 的 skill 后，**软链接不需要改**（自动同步）。

只有以下情况才需要更新软链接：
- 新增 profile
- 新增/删除某个 profile 需要的 skill

更新软链接用 PowerShell 脚本：`C:\Users\11390\AppData\Local\Temp\update_profile_links.ps1`

### 4.3 验证软链接

```bash
# 列出某个 profile 的所有软链接 + 目标
ls -la /c/Users/11390/AppData/Local/hermes/profiles/researcher/skills/

# 检查死链
find /c/Users/11390/AppData/Local/hermes/profiles/*/skills -type l ! -exec test -e {} \; -print
```

---

## 5. 维护 checklist

每月/季度做这些：

- [ ] 检查 `hermes skills list`，确认 nanqiao_skills 里的所有 skill 都被软链覆盖
- [ ] 检查死链（`find ... -type l ! -exec test -e {} \;`）
- [ ] 看 CHANGELOG 的 Unreleased 段落，确认无未发布内容堆积
- [ ] 验证 `date` 真实日期与项目 README 中的"对应节日"一致
- [ ] 验证每个项目的私域钩子格式未漂移（zhaimi = 【区域+宠物】，zhaidian = 【城市+房型】）
- [ ] 验证 share 技能被各项目正确引用（`grep -r "share/" project_skills/`）

---

## 6. 异常处理

### 6.1 软链断了

```bash
# 删除死链
find ... -type l ! -exec test -e {} \; -delete
# 重建（参考 update_profile_links.ps1）
```

### 6.2 误删了重要 skill

```bash
git log --diff-filter=D --name-only --pretty=format: | grep -i <skill-name>
git checkout HEAD~N -- <path-to-skill>
```

### 6.3 改坏了某个 skill 但忘了改什么

```bash
git log --all --oneline -- <path>
git diff HEAD~1 -- <path>
```

### 6.4 多人改冲突

```bash
git fetch origin
git rebase origin/master
# 解决冲突
git add -A
git rebase --continue
```

---

## 7. 不允许的修改

除非重大重构并提前通知，否则以下操作禁止：

- [ ] 直接删除 share/ 下的任何 skill（必须先 deprecated 30 天）
- [ ] 改 share/ skill 的相对路径（其他 skill 通过相对路径引用，乱改会断链）
- [ ] 改私域钩子格式（zhaimi / zhaidian 各有固定格式）
- [ ] 跨项目改其他项目的 skill（直接 PR 到对应项目目录）
- [ ] 不更新 CHANGELOG 就 commit
- [ ] 不在 commit 信息里写 scope

---

## 8. 联系方式

- 仓库：github.com/sunganhao8-lgtm/nanqiao-skills
- Issues：直接在 GitHub 上开 issue
- 主理人：BosaDong

---

**最后**：本文件是 nanqiao_skills 项目运作的"宪法"。任何与本文档冲突的实践，以本文档为准。
