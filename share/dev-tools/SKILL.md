---
name: dev-tools
description: 开发工具链。GitHub 工作流、AI 编码代理、代码审查、计划/验证。合并自 github-*、codex、claude-code、opencode、hermes-agent、hermes-agent-skill-authoring、plan、spike、systematic-debugging、windows-git 等。
version: 1.0.0
---

# dev-tools — 开发工具链

## 触发条件

- Git/GitHub 操作
- AI 编码（Codex / Claude Code / OpenCode）
- 代码审查 / 代码简化
- 计划/技术验证/调试
- 编写/维护 Hermes skill

## 功能索引

| 功能 | 触发词 | 方法 |
|------|--------|------|
| GitHub PR 生命周期 | "提PR / 合并 / 开分支" | github-pr-workflow |
| 代码审查 | "Review 代码" | github-code-review |
| Issue 管理 | "提 Issue / 列 Issue" | github-issues |
| 仓库管理 | "clone / fork / 创建 repo" | github-repo-management |
| GitHub 认证 | "配 GH token / SSH" | github-auth |
| AI 编码 (Codex) | "用 Codex 写代码" | codex |
| AI 编码 (Claude Code) | "用 Claude Code" | claude-code |
| AI 编码 (OpenCode) | "用 OpenCode" | opencode |
| 写计划/方案 | "出计划 / 方案" | plan: 写 .hermes/plans/ |
| 快速验证 | "试一下 / spike" | spike: 一次性实验 |
| 系统调试 | "调试 bug" | systematic-debugging: 4 阶段 RCA |
| 编写 Skill | "写 skill / 改 skill" | hermes-agent-skill-authoring |
| Hermes 配置 | "配 hermes / 改配置" | hermes-agent |
| Windows Git | "Git 初始化 / 快照" | windows-git |
| 代码简化 | "简化 / 清理代码" | simplify-code |
| TDD | "测试驱动" | test-driven-development |

### Skill 编写规范（hermes-agent-skill-authoring）

每个 SKILL.md 必须包含：
1. YAML frontmatter（name, description, version）
2. 触发条件（精确到用户说什么话触发）
3. 工作流步骤（分步可执行）
4. 输入/输出格式
5. 红线清单（不可违反的约束）

### 代码质量门禁（requesting-code-review）
PR 前安全检查：
- 密钥泄露扫描
- import 安全
- 性能门禁
- auto-fix 支持

---

## Hermes 技能管理

管理 Hermes 技能库本身（增删改查、组织、profile 分配、清理）。

### 技能发现机制

Hermes 通过扫描 `$HERMES_HOME/skills/` 目录发现技能。每个子目录内含 `SKILL.md` 即视为一个技能。**没有注册表，没有索引文件**——目录即数据库。

```python
# 核心扫描逻辑（简化）
SKILLS_DIR = HERMES_HOME / "skills"
for md in SKILLS_DIR.rglob("SKILL.md"):
    name = md.parent.name  # 目录名 = 技能名
```

主技能目录 `skills/` 里可以有子目录组织，Hermes 会递归扫描。所有技能名是扁平的——同名冲突时后面的覆盖前面的。

### 禁用无用技能（不删除）

在 `config.yaml` 中添加禁用列表：

```yaml
skills:
  disabled:
    - ascii-art
    - polymarket
    - p5js
```

或通过 CLI：
```bash
hermes config set skills.disabled "skill-a,skill-b,skill-c"
```

禁用的技能不会出现在 `hermes skills list` 输出中，不会被 agent 加载，但文件还在磁盘上。可随时从列表中移除来恢复。

### 归档旧技能（有备份可恢复）

用 curator 归档不再需要的本地技能（bundled/hub 技能不可归档）：

```bash
hermes curator archive <skill-name>
```

归档的技能移到 `skills/.archive/<skill-name>/`，agent 不会再加载。可通过 `hermes curator restore <skill-name>` 恢复。

### Profile 技能管理

每个 Hermes profile（`profiles/<name>/`）是一个**完全独立的 Hermes 实例**，有自己的 `skills/` 目录、`config.yaml`、`.env`、`sessions/`、`memories/` 等。

当通过 `hermes -p <name> chat` 运行 profile 时，`HERMES_HOME` 指向 profile 目录，技能扫描从 profile 的 `skills/` 开始。

**不推荐复制文件**（`seed_profile_skills()` 的默认行为）——改用**软链接**，这样修改源技能时所有 profile 自动同步。

#### Windows 软链接设置（PowerShell）

git-bash 的 `ln -s` 在 Windows 上通常失败（需要 MSYS 特殊配置）。用 PowerShell：

```powershell
# 软链接目录（目录符号链接，不是快捷方式）
New-Item -ItemType SymbolicLink -Path "C:\path\to\profile\skills\skill-name" `
  -Target "C:\path\to\skills\nanqiao_skills\share\skill-name"
```

```bash
# 或通过 cmd.exe（需管理员或开发模式）
cmd.exe /c "mklink /D C:\path\to\link C:\path\to\target"
```

#### 推荐的 profile 技能结构

```
profiles/<name>/skills/
├── web-scraping    → ../../../../skills/nanqiao_skills/share/web-scraping   (symlink)
├── research-utils  → ../../../../skills/nanqiao_skills/share/research       (symlink)
├── content-utils   → ../../../../skills/nanqiao_skills/share/content-processing (symlink)
└── zhaimi-research → ../../../../skills/nanqiao_skills/project_skills/zhaimi/research (symlink)
```

这样每个 profile 只看到自己需要的少量技能，不会有 100+ 技能的冗余和混淆。

### 技能分类与组织模式

推荐的分层结构：

```
skills/
├── share/                    ← 公用工具型技能（不依赖具体项目）
│   ├── browser-automation/   ← 浏览器操作
│   ├── image-gen/            ← 图片生成
│   ├── content-processing/   ← 内容处理
│   └── ...
│
└── project_skills/           ← 项目专用技能
    └── <project>/
        ├── research/         ← researcher profile 用的技能
        ├── vidiator/         ← vidiator profile 用的技能
        └── orchestrator/     ← dispatcher 用的调度技能
```

**引用规范**：项目技能在 SKILL.md 中用相对路径引用公用技能：
```yaml
references:
  - ../../share/web-scraping/SKILL.md
  - ../../share/image-gen/SKILL.md
```

### Bundled Manifest 说明

`skills/.bundled_manifest` 文件追踪 Hermes 官方内置技能（格式：`name:hash`）。不在 manifest 中的技能被视为"custom/local"，由用户管理。官方技能更新时，Hermes 会检查 hash 判断有没有被用户修改过。

```bash
# 检查哪些是 bundled vs custom
hermes skills list --source all
# builtin = Hermes 官方内置
# local   = 用户自定义/本地添加
```

### 完整清理工作流

1. 备份：`cp -r skills skills.backup.$(date +%Y%m%d)`
2. 禁用无用官方技能：`hermes config set skills.disabled "skill-a,skill-b,..."`
3. 归档已整合的本地技能：`hermes curator archive baoyu-image-gen`
4. 重建 profile 技能：删除复制文件，改为软链接（见上方 PowerShell 命令）
5. 验证：`hermes skills list --enabled-only` 检查只剩需要的技能

---

## 红线
- [ ] 不把 AI 编码代理作为默认方案（代码量小直接写）
- [ ] plan 产生文件不是空壳，包含完整代码
- [ ] Windows Git 注意路径格式（MSYS + native 双兼容）
- [ ] 禁用技能用 config 而非删除文件（可恢复）
- [ ] Profile 技能用软链接而非复制（源修改自动同步）
- [ ] Windows 上创建链接用 PowerShell `New-Item` 而非 git-bash `ln -s`

