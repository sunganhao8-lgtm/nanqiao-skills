---
name: image-gen
description: AI 图片生成。覆盖豆包 doubao.com、即梦 Dreamina、MiniMax API 三种出图途径。合并自 baoyu-image-gen、xhs-cover-doubao-workflow、dreamina-cli、baoyu-compress-image。
version: 1.0.0
---

# image-gen — 图片生成

## 触发条件

- "帮我生成一张图" / "出图"
- 需要做小红书封面/配图
- 需要图片压缩/格式转换

## 出图途径优先级

### ① 豆包官网（首选）

通过 `browser-automation` 在 doubao.com 操作。

**适用**：小红书封面 3:4、头像 1:1
**优点**：免费、风格好、有参考图功能
**流程**：见 `../browser-automation/SKILL.md` → §B 豆包官网出图
**出图 prompt 格式**：中文 prompt，附参考图 `logo.png` + `主页背景图.png`

### ② 即梦 Dreamina（兜底）

```bash
# dreamina-cli 命令
dreamina text2image --prompt "..." --aspect-ratio 3:4
dreamina image2image --image path.jpg --prompt "..."
```

**注意**：首次使用需登录，`dreamina login` 扫码。
**积分**：`dreamina credits` 查看剩余。

### ③ MiniMax API（备选）

用 `baoyu-image-gen` 的 MiniMax 后端：
```
hermes -z "prompt" chat --profile creator
```
或在 execute_code 中调用 MiniMax image API。

### 图片压缩/格式转换

```bash
# baoyu-compress-image 核心功能
# WebP 压缩
cwebp -q 80 input.png -o output.webp

# PNG 压缩
pngquant --quality=65-80 input.png -o output.png
```

**默认输出格式**：WebP（体积最小）
**批量处理**：Python PIL 脚本遍历目录

## 品牌视觉规范（宅咪出图必读）

路径：`C:\Users\11390\Desktop\宅咪小红书笔记\品牌视觉规范.md`

**核心要求**：
- 3D 软糖治愈系 · 暖橙奶油 · 圆润厚实
- 主色 #F4B860 / 辅色 #FFF4E1 / 文字 #5C3D2E
- 左上方柔光（45° 入射角）
- 大眼 Q 萌的动物（如果有）
- 关键数字深橙加粗加大
- 角标 `宅咪 ZHAIMI` 露出
- ❌ 禁止：平面 2D / doodle / 莫兰迪 / 冷色 / 人脸

## 小红书多图卡片生成

用 `baoyu-xhs-images` 框架生成系列图卡（1-10 张）：
- 12 视觉风格 × 8 布局 × 3 调色板
- 默认 warm 风格（亲民/手绘）
- 3:4 竖图

## 红线

- [ ] 出图前读品牌视觉规范（宅咪项目专用）
- [ ] prompt .md 与生成图同目录（不分子目录）
- [ ] 不画人脸（猫可以、卡通公仔可以）
- [ ] 参考图优先用宅咪 logo.png + 主页背景图.png
- [ ] 不要用跑不动的后端（FAL KEY 未配）
