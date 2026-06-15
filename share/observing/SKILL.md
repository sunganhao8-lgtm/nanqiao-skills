---
name: observing
description: 数据监控/变化检测。网页变更、数据看门狗、自动巡检。
version: 1.0.0
---

# observing — 监控

## 触发条件

- "帮我盯着 / 监控 / 看有没有变化"
- "定期检查 / 每日巡检"

## 实现方式

使用 Hermes cronjob + script 模式：

### 看门狗模式（no_agent=True）
```bash
# 脚本产出 stdout 非空 → 投递用户
# 脚本产出 stdout 空 → 静默（没情况）
# 适用：磁盘/GPU/内存阈值、网站可用性
```

### Agent 模式（no_agent=False）
```bash
# LLM 驱动：汇总 RSS、挑有趣的推
# 适用：每周情报扫描、博客监控
```

### 情报扫描节奏
- 每周 2-3 次被动扫描
- 只推跟宅咪/出图/Chrome CDP/凭据/Agent 调度相关的料
- 没料闭嘴
- "今天有什么料" = 手动触发口令

## 参考
- `../research/SKILL.md` → blogwatcher（RSS 订阅）
- cronjob 工具：`cronjob(action='create', schedule='30m', script='path/to/watchdog.sh', no_agent=True)`
