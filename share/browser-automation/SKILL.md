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
- [ ] **Chrome 进程在跑 ≠ Chrome 可被 MCP 连接**。必须用 `chrome.exe --remote-debugging-port=9222` 启动，否则 MCP 报 "Could not connect to Chrome"——见 `references/chrome-mcp-troubleshooting.md`

## 关键陷阱（必读）

### 1. Chrome MCP 静默失败 = 调试端口未开

症状：`mcp_chrome_devtools_navigate_page` 报 `Could not connect to Chrome. Failed to fetch browser webSocket URL from http://127.0.0.1:9222/json/version`。

**两个独立的检查**：
- `tasklist | grep chrome.exe` → 进程列表（≠ 9222 监听）
- `netstat -ano | grep :9222` → 端口监听（这个才是 MCP 需要的）

如果 Chrome 在跑但 9222 没监听 → 用调试端口重启 Chrome：
```bash
# Win+R 输入：
"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222
```
首次会有 Windows 防火墙弹窗 → 点"允许访问"。

详见 `references/chrome-mcp-troubleshooting.md`。

### 2. CORB 静默拦截 `<script src>` 响应

症状：浏览器控制台报 `Response was blocked by CORB (Cross-Origin Read Blocking)`，script 标签的 `onload` 永远不触发。

**触发条件**：从本地 `file://` 或 `http://localhost` 加载**非标准响应**的 JS（如 Content-Type 不正确、响应包含 HTML 错误页伪装）。

**绕过方法 — fetch + 文本注入**：
```js
const text = await fetch('path/to/script.js').then(r => r.text());
const s = document.createElement('script');
s.textContent = text;
document.body.appendChild(s);
// 响应被作为字符串读取，CORB 不再拦截
```

适用场景：本地 Python HTTP server 提供的 JS 库、跨域加载的 IIFE 脚本。

### 3. 高德地图 AMapLoader 的 callback 陷阱

高德 maps.js **内部会覆盖** `window.___onAPILoaded`，所以提前设置的 callback 不会触发。**改用 setInterval 轮询 `window.AMap`**：

```js
if (window.AMap && typeof AMap.Map === 'function') {
  initMap();
} else {
  let tries = 0;
  const iv = setInterval(() => {
    tries++;
    if (window.AMap && typeof AMap.Map === 'function') {
      clearInterval(iv);
      initMap();
    } else if (tries > 60) {
      clearInterval(iv);
      console.error('AMap 加载超时');
    }
  }, 500);
}
```

**AMap.Map `complete` 事件额外陷阱**：AMap.Map 的 `complete` 事件在所有瓦片加载完成后才触发。如果使用**本地瓦片**（如 OSM tiles）且部分瓦片返回 404，`complete` 事件**可能永远不会触发**，导致 POI/标记/KPI 渲染流程卡死。

**解法**：添加 setTimeout 兜底，超时后强制进入渲染流程：

```js
STATE.map.on('complete', () => {
  bootstrapLocalResults();
  onAllSearchesDone();
});
// 兜底：complete 事件可能因瓦片 404 永不触发
setTimeout(() => {
  if (STATE.successCount === 0) {  // 还没渲染过
    diagLog('⚠️ complete 未触发，强制进入渲染');
    bootstrapLocalResults();
    onAllSearchesDone();
  }
}, 2000);
```

**2021-12-02 后申请的 AMap Key 必须配 `securityJsCode`**，否则 PlaceSearch/Walking/Geocoder 全部失败。

### 4. 高德地图 POI/KPI/Walking 全本地化模式

当制作**展示给客户的选址分析地图**时（非实时查询场景），**不要依赖 AMap.PlaceSearch / Geocoder / Walking**：

- **POI 全部硬编码**：门店、万达、学校、周边小区的坐标一次性在 JS 里定义好
- **Walking 用几何算法**：store→wanda 坐标计算 haversine 距离 × 1.15 路径系数 ≈ 1.33km/16min
- **KPI 用人口估算**：每个小区标人口数（~4200/区），按距离分类统计

**好处**：零网络依赖、零加载延迟、不依赖高德 API Key 可用性。详见 `references/amap-localization-example.md`。

### 5. 全屏功能 for canvas-based 页面（AMap / Konva）

为包含 canvas 绘制的页面（高德地图、Konva 编辑器）添加全屏查看功能时，**禁止使用 `cloneNode(true)`**——canvas 的像素内容不会被克隆，全屏后是空 canvas。

**正确做法：直接对原 DOM 元素做 `position: fixed` 铺满**：

```js
function openFullscreen() {
  // 用一个 overlay div 做激活状态标记
  overlay.classList.add('active');
  // 把承载内容的容器拉伸到全屏
  wrapper.style.position = 'fixed';
  wrapper.style.left = '0';
  wrapper.style.top = '0';
  wrapper.style.width = '100vw';
  wrapper.style.height = '100vh';
  // 通知地图/画布 resize
  if (map) setTimeout(() => map.resize(), 250);
}
function closeFullscreen() {
  overlay.classList.remove('active');
  // 恢复原始尺寸
  wrapper.style.position = orig.position;
  wrapper.style.width = orig.width;
  // ...
  if (map) setTimeout(() => map.resize(), 100);
}
```

**iframe 逃出模式**：当页面作为 iframe 嵌入 BP 标签页时，全屏按钮应调用父级函数而非自身：

```js
btn.addEventListener('click', () => {
  if (window.parent && window.parent !== window &&
      typeof window.parent.openFullscreenTab === 'function') {
    window.parent.openFullscreenTab('map');  // 在父 BP 全屏
  } else {
    openFullscreen();  // 独立页面则自身全屏
  }
});
```

### 4. 调试地图/JS 加载问题的诊断工具

遇到"页面卡在 loading 状态"或"某个 JS 没生效"时：
1. `mcp_chrome_devtools_evaluate_script(() => typeof window.SomeAPI)` → 看 API 是否定义
2. `mcp_chrome_devtools_list_console_messages()` → 看 console 错误
3. `mcp_chrome_devtools_list_network_requests(resourceTypes=['script'])` → 看哪些脚本没加载
4. `mcp_chrome_devtools_take_screenshot(filePath=...)` → 视觉确认
