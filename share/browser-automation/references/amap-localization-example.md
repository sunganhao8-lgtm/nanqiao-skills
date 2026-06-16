# 高德地图 POI/KPI/Walking 全本地化模式

## 背景

为「宅咪·临港 LIN 舍」制作选址分析地图（site-map.html），最初依赖 AMap.PlaceSearch + Geocoder + Walking 三次云端 API。实际发现：
- 11 个 POI 分 3 级 fallback 仍多次搜索失败
- Walking 在部分坐标对之间报错
- 每次加载需要 ~5-10 秒网络等待
- **关键是**：数据是展示用，不需要实时查询

## 架构：本地 STORE_DATA + 几何算法

### 坐标数据结构

```js
const STORE_DATA = {
  store: {
    id: 'store', name: '宅咪·临港 LIN 舍', role: 'store',
    lng: 121.815712, lat: 30.906358,
    address: '上海市浦东新区泥城镇五岭路×咏梅路交叉口西 120 米',
    source: '实测坐标'
  },
  school: { /* 类似结构 */ },
  wanda: { /* 类似结构 */ },
  estates: [
    { id: 'e1', name: '新元·露华苑', role: 'estate',
      lng: 121.814523, lat: 30.910012,
      address: '浦东新区泥城镇露华苑', population: 4200, buildYear: 2018 },
    // ... 总共 8 个小区
  ]
};
```

### 初始化：一次填入

```js
function bootstrapLocalResults() {
  STATE.results = {};
  STATE.results.store = { ...STORE_DATA.store, missing: false };
  STATE.results.wanda = { ...STORE_DATA.wanda, missing: false };
  STATE.results.school = { ...STORE_DATA.school, missing: false };
  STORE_DATA.estates.forEach(e => {
    STATE.results[e.id] = { ...e, missing: false };
  });
  STATE.successCount = Object.keys(STATE.results).length;  // = 11
  STATE.failCount = 0;
}
```

### 步行路径：几何折线算法

不用 AMap.Walking（云端），用本地几何折线 + haversine 距离 × 系数：

```js
function computeWalkingRoute() {
  const a = [STORE_DATA.store.lng, STORE_DATA.store.lat];
  const b = [STORE_DATA.wanda.lng, STORE_DATA.wanda.lat];
  // 沿途关键节点（沿主干道走）
  const mid1 = [a[0] + 0.003, a[1] + 0.0025];
  const mid2 = [b[0] - 0.002, b[1] + 0.001];
  const path = [a, mid1, mid2, b];
  // 累计距离
  let total = 0;
  for (let i = 1; i < path.length; i++) total += haversine(path[i-1], path[i]);
  // 步行 5km/h × 1.15 路径系数
  const distance = total * 1.15;
  const minutes = Math.round(distance / 5000 * 60);
  return { path, distance, duration: minutes };
}
```

结果：store→wanda = **1.33 km · 16 分钟**（与高德 Walking API 实测结果非常接近）。

### KPI：本地统计

```js
function updateKPIs() {
  // 步行到万达
  const km = STATE.routeInfo.distance / 1000;
  const min = STATE.routeInfo.duration;
  document.getElementById('kpi-distance').textContent = km.toFixed(2) + ' km';
  document.getElementById('kpi-distance-desc').textContent = `约 ${min} 分钟 · 本地几何算法`;

  // 1km / 3km 统计
  let c1 = 0, c3Pop = 0;
  const center = STATE.storeCenter;
  Object.values(STATE.results).forEach(r => {
    if (r.role !== 'estate') return;
    const d = haversine(center, [r.lng, r.lat]) / 1000;
    if (d <= 1.0) c1++;
    if (d <= 3.0) c3Pop += (r.population || 4200);
  });
  document.getElementById('kpi-1km').textContent = c1 + ' / 8';
  document.getElementById('kpi-3km').textContent = (c3Pop / 10000).toFixed(2) + ' 万';
}
```

## 效果对比

| 指标 | 云端路线 | 本地路线 |
|------|---------|---------|
| store→wanda 距离 | 1.1-1.4 km | **1.33 km** |
| 加载延迟 | 5-10 秒 | **0 秒** |
| 网络依赖 | 必须联网 | **零依赖** |
| 成功率 | 受 API Key/白名单/网络影响 | **100%** |

## 适用场景

- 展示给客户的选址分析地图
- 坐标已经过一次校验确认准确
- 不需要"实时路况"或"动态路径规划"
- 需要在局域网/内网离线展示

## 不适用场景

- 需要实时交通信息的导航
- 坐标未知、需要自动搜索定位
- 需要精确到每栋楼的步行路径
