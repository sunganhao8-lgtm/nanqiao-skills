---
name: browser-automation
description: 浏览器自动化。Chrome CDP 9222 远程调试 + chrome-devtools MCP + 豆包官网出图。覆盖本地浏览器控制、登录态页面操作、豆包/doubao.com AI 图片生成。
version: 1.0.0
---

# browser-automation — 浏览器自动化

## 触发条件

- 需要操作浏览器（点击、填表、截图）
- 需要打开带登录态的网页（小红书、抖音、豆包）
- 需要在豆包官网 doubao.com 生成图片
- Chrome CDP 相关操作

## 工具矩阵

### A. Chrome 远程调试 (CDP 9222)

```bash
# 连接本地 Chrome（先确保 Chrome 以远程调试模式启动）
# 命令行：chrome.exe --remote-debugging-port=9222 --profile-directory="DebugProfile"
```

可用工具：`mcp_chrome_devtools_*`（共 36 个工具）

**常用操作**：
- `mcp_chrome_devtools_new_page(url)` — 打开新页面
- `mcp_chrome_devtools_take_snapshot()` — 取页面 a11y 树
- `mcp_chrome_devtools_take_screenshot()` — 截图
- `mcp_chrome_devtools_evaluate_script(fn)` — 执行 JS 取数据
- `mcp_chrome_devtools_fill(uid, value)` — 填表单
- `mcp_chrome_devtools_click(uid)` — 点击元素
- `mcp_chrome_devtools_navigate_page(type, url)` — 导航

**B 站/小红书/抖音等登录态页面**：
- 优先用 MCP（有登录态 Cookie）
- 脚本方式：Python + playwright 连接 ws://localhost:9222

### B. 豆包官网出图 (doubao.com)

**流程**：
1. `mcp_chrome_devtools_new_page('https://www.doubao.com/chat/create-image')` 
2. 等待页面加载
3. `mcp_chrome_devtools_take_snapshot()` 获取当前页面互动元素
4. 找到图片输入区域（通常是一个上传/粘贴区域，或 AI 生成入口）
5. 填写 prompt 并提交
6. 等待生成（`mcp_chrome_devtools_wait_for` 或轮询）
7. 截图或取结果链接

**注意**：
- 确保 Chrome 已登录豆包（Cookie 在 DebugProfile 中）
- 单次耗时会较长（30s-2min），需要耐心

### C. 通用浏览器脚本

```python
# Python + CDP 脚本模版
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.connect_over_cdp("http://localhost:9222")
    page = browser.contexts[0].pages[0]
    page.goto("url")
    # 操作...
```

脚本写入 `C:\Users\11390\AppData\Local\Temp\` 再 PowerShell 执行。

## 参考

- Chrome DevTools Protocol：[https://chromedevtools.github.io/devtools-protocol/](https://chromedevtools.github.io/devtools-protocol/)
- Playwright CDP：[https://playwright.dev/python/docs/api/class-browser#browser-connect-over-cdp](https://playwright.dev/python/docs/api/class-browser#browser-connect-over-cdp)

## 红线

- [ ] 不要用 WSL Playwright — 必须用原生 Windows
- [ ] CDP 工具命名要精确（区分"chrome-devtools MCP"和"本地 9222 脚本"）
- [ ] uid 格式 `{pageId}_{n}` ≠ `e{n}`，每次操作前重新 take_snapshot
- [ ] Browserbase `browser_*` 国内可能 timeout，优先本地 CDP
