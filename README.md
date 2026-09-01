# Travel Plan

Visite domain: https://alvincode.github.io/travelplan/ + *.html

# Prompt Template

# 携宠自驾攻略生成 Prompt（完整模板）

> 复制下方「Prompt 正文」整段使用。`{{...}}` 为每次规划时需替换的变量。

---

## Prompt 正文

```
我和我爱人及一只金毛犬（6岁）计划十一假期期间从北京出发自驾去湖北和河南，开特斯拉 Model Y，途中均住宿宠物友好酒店，请帮我规划一条完整的旅行日程安排。

---

## 一、行程约束

### 基本信息
- 出行人员：2 成人 + 1 只金毛犬（6 岁）
- 车辆：特斯拉 Model Y
- 总天数：{{总天数}} 天（{{出发日期}} 出发，{{返程日期}} 回到北京）
- 请假：中间请 {{请假天数}} 天假

### 时间约束
- 每日出发时间：8:30–10:00
- 每日到达住宿地：平均约 20:00，不晚于 21:00
- 每日最大车程：不超过 {{最大日里程}} km

### 路线策略
- 必去城市：{{必去城市列表，如：鹤壁市、洛阳市、许昌市、随州市、武汉市}}
- 顺序：先湖北后河南（湖北在去程，河南在返程）
- 去程与返程走**不同路线**，途径不同地点，感受不同人文与美食
- 按推荐度安排景点：高推荐度至少 2 天 1 夜；路过型 1 天；低推荐度可不去
- 整体节奏轻松，不要一天换一座城

### 住宿约束
- 全部宠物友好酒店/民宿
- 每个住宿点：1 个主选 + 2 个备选
- 优先自带充电桩或附近有充电桩（方便夜间补电）
- 价格上限：{{每晚最高价格}} 元/晚

### 充电约束
- 消除充电焦虑：规划途中充电站及充电时间
- 充分考虑节假日排队时长
- 每个关键节点提供备选充电方案

### 景区信息要求
每个景区需说明：
- 是否宠物友好
- 不可带宠时，游览时长是否适合将狗留车内
- 建议游玩时长、开放时间、预约情况
- 票价及购票渠道

### 信息来源与验证
1. 搜索小红书「{{出发地}}-{{目的地区域}}」自驾环线或相关攻略
2. 整理：景区、美食、充电站/桩、宠物友好酒店、避坑信息（排除体验较差的）
3. 有官方渠道时与官方交叉验证；无官方渠道时在小红书内多源交叉验证

### 其他输出
- 必备物品及安全提示（人 + 宠）

---

## 二、输出格式

请输出：
1. **详细文字行程安排**（逐日、含车程/充电/住宿/景点/美食）
2. **一个完整、自包含的单文件 HTML**（内联 CSS + JS，无外部依赖除高德 API）

HTML 要求：
- 排版清晰、支持响应式（移动端 + 桌面端）
- 可视化强、信息完整无遗漏
- 能用表格的数据一律用表格
- 参考样式：https://github.com/ALvinCode/travelplan/blob/master/%E5%8C%97%E4%BA%AC%E8%A5%BF%E5%AE%89%E5%BC%A0%E5%AE%B6%E7%95%8C%E5%87%A4%E5%87%B0%E5%8F%A4%E5%9F%8E%E9%83%91%E5%B7%9E%E8%87%AA%E9%A9%BE%E8%AE%A1%E5%88%92.html

### HTML 内容模块（必须包含）
1. 头部：标题、日期、核心标签（路线策略/宠物/车型等）
2. 行程总览表：日期、星期、行程、住宿地、车程
3. 逐日详细安排（分区/分栏）
4. 充电总攻略（超充 + 高速服务区 + 节假日策略）
5. 宠物政策表 + 携宠必备清单 + 安全提示
6. **交互式高德地图**（见下文技术规范）
7. 页脚摘要

---

## 三、高德地图 API 配置

```html
<script>
  window._AMapSecurityConfig = {
    securityJsCode: "{{securityJsCode}}"
  };
</script>
<script src="https://webapi.amap.com/maps?v=2.0&key={{amapKey}}&plugin=AMap.Driving,AMap.Scale"
  onerror="document.getElementById('mapError').style.display='block';document.getElementById('loading').style.display='none';">
</script>
```

**重要：插件只加载 `AMap.Driving` 和 `AMap.Scale`，禁止加载 `AMap.ToolBar`、`AMap.ControlBar`、`AMap.MarkerClusterer`。**

---

## 四、地图功能规范（必须严格实现）

### 4.1 地图容器结构

`#mapContainer` 内按以下层级放置（均为 `#mapContainer` 的子元素，绝对定位）：

| 元素 | 位置 | z-index | 说明 |
|------|------|---------|------|
| `#mapControls` | 右上角 | 1001 | 缩放 + 全屏按钮 |
| `#legendPanel` | 左下角 | 1000 | 图例筛选 + 重置/刷新 |
| `#markerTooltip` | 动态定位 | **9999** | hover 信息弹层（最高层级） |
| `#loading` / `#mapError` | 居中 | 999 / 1000 | 加载/错误态 |

各控件**不得互相遮挡**，移动端需适配缩小尺寸。

### 4.2 地图控件（自定义 HTML 按钮，禁止用 AMap.ToolBar）

`#mapControls` 包含 3 个按钮：
- `#zoomInBtn`：放大（调用 `map.zoomIn()`）
- `#zoomOutBtn`：缩小（调用 `map.zoomOut()`）
- `#fullscreenBtn`：全屏/退出全屏（`requestFullscreen` / `exitFullscreen`）

全屏时 `#mapContainer:fullscreen` 撑满视口（100vw × 100vh）。

图例面板底部另含：
- `#resetViewBtn`：重置视图（自适应当前可见 marker）
- `#refreshMapBtn`：刷新地图（清除 marker 和路线后重建）

### 4.3 图例筛选

5 类 checkbox，默认全选：

| data-type | 分类 | 颜色 |
|-----------|------|------|
| `hotel` | 住宿（主选 + 备选） | `#2563eb` |
| `scenic` | 景点 | `#16a34a` |
| `charger` | 充电站 | `#f59e0b` |
| `service` | 服务区 | `#64748b` |
| `food` | 美食 | `#e11d48` |

实现要求：
- 每个 `.legend-item` 带 `data-type` 属性
- 点击整行可切换 checkbox
- 筛选逻辑：`marker.setExtData({ cat, data })` 存分类，勾选变化时对 marker 调用 `show()` / `hide()`
- **禁止**使用 MarkerClusterer 替代 marker 渲染（会导致标记消失）
- marker 必须通过 `map.add(marker)` 直接挂载到地图

### 4.4 路线规划

- 使用 `AMap.Driving`，**必须**设置 `map: map`，让高德默认渲染路线（含起终点、途经点「经」字标记）
- 去程、返程各创建一个 Driving 实例，途经主要城市作为 waypoints
- `autoFitView: false`（由自定义逻辑控制视野）
- **禁止**手动画自定义颜色 Polyline（不使用红/蓝分色，统一高德默认路线色）
- 刷新时用 `driving.clear()` 清除路线后重新 search

### 4.5 标记点数据

所有 POI 写入 `mapData` 数组，字段：

```javascript
{ name: '名称', cat: 'hotel|scenic|charger|service|food', lng: 经度, lat: 纬度, info: '字段1|字段2|字段3...' }
```

- 经纬度必须精确，禁止重叠到无法选中
- 每个住宿城市标记主选 + 2 个备选（共 3 个 hotel 点）
- 标记用彩色圆点（16px，`content` 为 DOM 元素 `.map-marker-dot`）

### 4.6 Hover 信息弹层（关键：禁止用 setLabel）

**禁止**使用 `marker.setLabel()`（存在 z-index 低、mouseout 后弹层不消失等问题）。

必须使用独立 DOM 弹层 `#markerTooltip`：

**显示逻辑：**
- 在 marker 圆点 DOM 上绑定 `mouseenter` → 显示弹层
- 绑定 `mouseleave` → 立即隐藏弹层
- 同时只有一个弹层实例（全局单例）
- 地图拖拽开始时隐藏弹层
- 地图 move/zoom 时若弹层可见，跟随 marker 更新位置（`map.lngLatToContainer`）

**样式要求：**
- `z-index: 9999`，`pointer-events: none`
- 白底卡片 + 圆角 + 阴影 + 底部箭头
- 头部：分类色 badge + 地点名称
- 正文：`info` 字段按 `|` 拆分为 tag 标签展示，清晰易读

**各类 info 字段规范：**

| 分类 | info 必含字段（`|` 分隔） |
|------|---------------------------|
| 住宿 | 主选/备选、均价、宠物政策、停车场、充电桩详情 |
| 景点 | 门票、宠物政策、建议时长、开放时间、预约方式 |
| 充电站 | 经营方、桩数、功率 |
| 服务区 | 是否餐饮、充电桩数量与功率 |
| 美食 | 人均、地址/分店、营业时间、预约/排队攻略 |

### 4.7 重置视图 & 刷新

- **重置视图**：对当前可见 marker 调用 `map.setFitView(visibleMarkers, false, [80,80,80,280])`，留足图例边距
- **刷新**：移除所有 marker → `driving.clear()` → 重新 `addMarkers()` + `drawRoute()` → 延迟 fitView

### 4.8 地图交互

- 可缩放、可拖拽
- 最小缩放级别为全国视野（zoom ≈ 4–6）
- 仅添加 `AMap.Scale` 比例尺（左下角），不添加 ToolBar

---

## 五、地图标记 hover 信息详情（写入 info 字段）

### 住宿（每个城市 3 个点：1 主 + 2 备）
平均价格、宠物政策、是否有停车场、是否有充电桩（含桩数、功率）、房源链接

### 景点（每城市至少 2 个推荐）
成人门票、宠物政策、建议游览时间、游览方式、开放时间、是否需预约

### 高速服务区
是否可餐饮、是否有充电桩（桩数、功率）

### 充电站
桩数、经营方（国网/特斯拉超充/星星充电等）、功率

### 美食
地址、人均、是否可预约、排队攻略、营业时间

---

## 六、交付前自检清单（必须逐项验证）

生成 HTML 后，逐项确认：

- [ ] 所有 POI 标记在地图上可见
- [ ] 去程/返程路线可见，含高德默认途经点「经」标记
- [ ] 路线为高德默认蓝色，无自定义红/蓝分色
- [ ] 图例 5 类筛选勾选/取消后 marker 正确 show/hide
- [ ] hover 标记：弹层 z-index 最高、样式清晰、鼠标离开立即消失
- [ ] 缩放按钮只有一组（右上角自定义 +/−/全屏），无重复 ToolBar
- [ ] 重置视图、刷新按钮功能正常
- [ ] 控件与图例、弹层互不遮挡
- [ ] 移动端布局正常（375px 宽度下可用）
- [ ] 直接在浏览器打开 HTML 验证通过

---

## 七、API 密钥

```javascript
securityJsCode: "{{securityJsCode}}"
amapKey: "{{amapKey}}"
```

当前可用值：
- securityJsCode: `8dea8562090a713e47d811c17a1d8806`
- amapKey: `c2de27926382776d9281eb5531d9ac06`

---

请按以上全部要求，输出完整行程安排 + 可直接在浏览器打开的 HTML 单文件。
```

---

## 变量替换表

| 占位符 | 本次示例值 |
|--------|-----------|
| `{{总天数}}` | 12 |
| `{{出发日期}}` | 9.25 |
| `{{返程日期}}` | 10.6 |
| `{{请假天数}}` | 3 |
| `{{最大日里程}}` | 500 |
| `{{每晚最高价格}}` | 500 |
| `{{必去城市列表}}` | 鹤壁市、洛阳市、许昌市、随州市、武汉市 |
| `{{出发地}}` | 北京 |
| `{{目的地区域}}` | 湖北-河南 |

---

## 踩坑备忘（已写入规范）

| 问题 | 错误做法 | 正确做法 |
|------|----------|----------|
| 标记消失 | 仅用 MarkerClusterer | `map.add(marker)` + show/hide 筛选 |
| 途经点不显示 | Driving 不绑 map | `new AMap.Driving({ map: map, ... })` |
| 路线颜色混乱 | 手动画红/蓝 Polyline | 用 Driving 默认渲染 |
| 缩放按钮重复 | 加载 AMap.ToolBar | 仅自定义 `#mapControls` |
| hover 弹层不消失 | `marker.setLabel()` | 独立 `#markerTooltip` + mouseenter/leave |
| 弹层被遮挡 | label 层级低 | `#markerTooltip { z-index: 9999 }` |

---

## 参考实现

已验证的 HTML 样例：`[docs/deepseek_html_20260901_ed9784.html](https://alvincode.github.io/travelplan/%E5%8C%97%E4%BA%AC%E6%B9%96%E5%8C%97%E6%B2%B3%E5%8D%97%E8%87%AA%E9%A9%BE%E6%B8%B8-2026.html)`
