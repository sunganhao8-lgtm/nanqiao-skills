# CHANGELOG

> nanqiao_skills 的所有重要变更记录。
> 格式遵循 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/)，
> 版本号遵循 [语义化版本](https://semver.org/lang/zh-CN/)。
>
> **每次修改必须追加条目**。新功能放 `Added`、破坏性变更放 `Changed`/`Removed`、
> 修复放 `Fixed`。**版本号不向上回退**。

## [Unreleased]

### 计划中
- 宅电品牌视觉规范建立（参考 `project_skills/zhaidian/vidiator/visual-design/SKILL.md` 中临时规范）
- `share/observing/` skill 完善
- wechat-tools：style-analyzer 实测和样本数据准备

---

## [2.1.0] - 2026-06-15

### Added
- **新 share 子目录 `wechat-tools/`** — 微信本地工具集
  - `SKILL.md` — 父级入口
  - `chat-export/SKILL.md` — 封装 WeChatDaily 项目的 wechat.py CLI
  - `style-analyzer/SKILL.md` — 对话风格画像分析（5 维度 + persona prompt）
- **外部项目集成**：WeChatDaily @ `C:\Users\11390\tools\WeChatDaily\`
  - 已克隆 + 初始化子模块 (wechat-decrypt)
  - 已创建 .env 模板
  - wx_key 工具需用户本地下载（GitHub 国内访问受限）

### Notes
- wechat-chat-export 调 `python wechat.py export --date X --groups Y --limit 99999`
- wechat-style-analyzer 输出 5 维 persona JSON，存到 `export/personas/`
- 隐私：聊天记录 100% 本地处理，不上传

---

## [2.0.0] - 2026-06-15

### Changed — 单职责拆分 + 双项目体系
- **zhaimi 4 个老 SKILL.md 拆成 8 个子技能**（按 profile 单职责拆分）
  - research: topic-scoring / competitor-analysis / viral-deconstruct / comment-insight
  - vidiator: script-writing / visual-design
  - creator: title-cover-check
  - orchestrator: content-orchestrator
- **新项目 zhaidian（宅电 ZHAIDIAN）**：上门电竞房搭建
  - 与 zhaimi 同结构（8 个子技能）
  - 主色：暗色科技（深空灰/冷蓝/赛博紫），区别于 zhaimi 暖橙治愈
  - 私域钩子：评论扣【城市+房型】（区别于 zhaimi 的【区域+宠物】）
- **profile 软链接重构**：
  - researcher: 11 个链接（share 3 + zhaimi research 4 + zhaidian research 4）
  - vidiator: 7 个链接（share 3 + zhaimi vidiator 2 + zhaidian vidiator 2）
  - creator: 4 个链接（share 2 + zhaimi creator 1 + zhaidian creator 1）
- **项目根 README 全面更新**（`project_skills/README.md`、`zhaimi/README.md`、`zhaidian/README.md`）
- `share/` 目录按功能分类（11 个子目录）
- 项目技能通过相对路径引用 `../../share/xxx/SKILL.md`

### Added
- **GitHub 仓库** `sunganhao8-lgtm/nanqiao-skills`（public）
- **CHANGELOG.md**（本文件）
- **CONTRIBUTING.md**（贡献者规范）
- **PROFILE_STRUCTURE.md**（profile 与 skill 的对应关系文档）
- **TEMPLATE-SKILL.md**（新技能模板）

---

## [1.0.0] - 2026-06-15

### Added
- 首次建立 `nanqiao_skills/` 个人 Hermes 技能体系
- 11 个 `share/` 公用技能（合并自 93 个原始 Hermes skills）
  - browser-automation / image-gen / content-processing
  - web-scraping / social-publish / media-audio
  - design-visual / research / productivity
  - observing / dev-tools
- 4 个 zhaimi 集成技能（research / vidiator / creator / orchestrator，打包式）
- 顶层索引：README.md、share/README.md、project_skills/README.md、zhaimi/README.md
- `.gitignore`（排除 Hermes 内部状态文件）
- Git 初始化 + 首次提交

### Removed（被整合/合并）
- 16 个无用官方 builtin skills（已写入 config.yaml disabled 列表）
- 21 个 baoyu 系列 skills（归档到 .archive/）
- 8 个旧版 zhaimi-* skills（归档到 .archive/）
- 多个项目冗余的 skills

### Technical
- 备份原 skills 目录至 `skills.backup.20260615_101135/`
- Hermes 客户端索引清理（归档的不再显示）
- 配置文件：`skills.disabled` 列表（14 个 builtin）
