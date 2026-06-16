# Chrome DevTools MCP 故障排查

## 快速诊断表

| 症状 | 原因 | 解法 |
|---|---|---|
| `Could not connect to Chrome. Failed to fetch browser webSocket URL from http://127.0.0.1:9222/json/version` | Chrome 没监听 9222 | 用 `--remote-debugging-port=9222` 重启 Chrome |
| `Chrome 进程在跑`但 MCP 报连不上 | 进程 ≠ 端口监听 | 检查 `netstat -ano \| grep :9222` |
| 首次连接 MCP 报 `Permission denied` / `Unauthorized` | Windows 防火墙拦了 | 弹窗点"允许访问" |
| MCP 能连但 `take_snapshot` 失败 | 目标页 close 了 / page id 失效 | 重新 `list_pages` 取新 uid |
| 控制台 `Response was blocked by CORB` | `<script src>` 响应被拦 | 用 `fetch + script.textContent` 注入 |
| AMapLoader `load()` 永远 pending | maps.js 内部 callback 覆盖了 `___onAPILoaded` | 改用 `setInterval` 轮询 `window.AMap` |

## Chrome 启动的两种方式

### Windows 一次性启动（推荐）

```bash
# 完全退出所有 chrome.exe 后，Win+R 输入：
"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222

# 如果用 Edge：
# 访问 edge://inspect/#remote-debugging → 勾选 "Allow remote debugging for this browser instance"
```

启动后**保持这个 Chrome 窗口常开**。MCP 后续都连这一个实例。

### 自定义 Profile（避免污染日常 Chrome）

```bash
chrome.exe --remote-debugging-port=9222 --profile-directory="C:\chrome-debug-profile"
```

> 警告：把生产环境的 Chrome 长期开调试端口，**任何同机进程都能连 9222 执行任意操作**。建议为 web-access agent 单独开一个 Chrome 实例。

## 进程 vs 端口 独立检查

```bash
# 进程检查（≠ 9222 监听）
tasklist | grep chrome.exe

# 端口检查（这个才是 MCP 真正依赖的）
netstat -ano | grep :9222

# HTTP 探测（最终证据）
curl http://127.0.0.1:9222/json/version
```

如果 `netstat` 没 `:9222` 但 `tasklist` 有 `chrome.exe` → 说明 Chrome 进程在跑但**没开调试端口**，MCP 连不上。

## 诊断地图/JS 加载问题的标准流程

```js
// 1. 检查全局 API 是否定义
mcp_chrome_devtools_evaluate_script(() => typeof window.AMap)
// → "undefined" 或 "object" 或 "function"

// 2. 看 console 错误
mcp_chrome_devtools_list_console_messages()
// → 找 [error] 级别消息，特别是 CORB / SyntaxError / Failed to load resource

// 3. 看实际发了什么网络请求
mcp_chrome_devtools_list_network_requests(resourceTypes=["script"])
// → 如果只看到 loader.js 没看到 maps.js，说明 maps.js 根本没请求

// 4. 视觉确认
mcp_chrome_devtools_take_screenshot(filePath="/tmp/page.jpg")
// → 看页面到底渲染成什么样
```

## CORB 绕过方案（fetch + 文本注入）

当 `<script src="...">` 被 CORB 拦截时：

```js
// ❌ 会被 CORB 拦截
const s = document.createElement('script');
s.src = 'vendor/lib.js';
document.body.appendChild(s);

// ✅ 绕过 CORB
fetch('vendor/lib.js')
  .then(r => r.text())
  .then(text => {
    const s = document.createElement('script');
    s.textContent = text;  // 响应作为字符串读取，不触发 CORB
    document.body.appendChild(s);
  });
```

## 高德地图 AMapLoader 集成模版

```js
// 关键：2021-12-02 后的 Key 必须配 securityJsCode
window._AMapSecurityConfig = {
  securityJsCode: '你的安全密钥'
};

// 轮询模式检测 AMap（避开 callback 覆盖问题）
function waitForAMap(timeout = 30000) {
  return new Promise((resolve, reject) => {
    if (window.AMap && typeof window.AMap.Map === 'function') {
      return resolve(window.AMap);
    }
    const start = Date.now();
    const iv = setInterval(() => {
      if (window.AMap && typeof window.AMap.Map === 'function') {
        clearInterval(iv);
        resolve(window.AMap);
      } else if (Date.now() - start > timeout) {
        clearInterval(iv);
        reject(new Error('AMap 加载超时'));
      }
    }, 200);
  });
}

await waitForAMap();
// AMap.Map / AMap.PlaceSearch / AMap.Walking / AMap.Geocoder 全部可用
```

## 网络相关坑

- **Nominatim**（OSM 官方 geocoder）：1 req/s 限速，UA 必须带真实联系方式
- **高德 REST API**：浏览器从 `localhost` 调 `https://restapi.amap.com` 需控制台白名单加 `*`
- **高德瓦片**：无离线包，本地化需手动抓 OSM/Mapbox 瓦片
- **CORB vs CORS**：CORB 是浏览器安全机制，**只挡 fetch 响应被 JS 读取**，对 `<script src>` 也会拦；对 `<img src>` 不拦
