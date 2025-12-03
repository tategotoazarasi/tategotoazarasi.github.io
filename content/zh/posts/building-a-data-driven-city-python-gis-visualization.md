---
date: '2025-12-03T20:32:25Z'
draft: false
title: '打造数据驱动的城市：利用 Python 整合住房、治安与社会剥夺指标'
summary: '本文复盘了如何利用 Python、Geopandas 和 Folium 将住房规划、治安数据与社会经济指标融合，构建支持动态权重的城市决策可视化系统。'
tags: ['python', 'geopandas', 'folium', 'data-visualization', 'gis', 'urban-planning', 'spatial-analysis', 'smart-city', 'data-science', 'hackathon']
---

最近，我有幸参加了由利物浦城市地区联合管理局（LCR CA）与 Homes England 联合举办的一场黑客松。这次活动的背景非常宏大且务实：政府目前掌握着一份包含约 350 个地块、总计超过 57,000 套新房的住房开发清单（Housing Pipeline）。然而，这些数据静静地躺在 Excel 表格里，仅仅是冰冷的数字。

我们的任务是：**如何让这些数据“活”过来？**

作为开发者，我们深知仅仅在地图上标出“这里要建房”是远远不够的。城市规划是一个复杂的系统工程。我们在哪里建房？那个地方安全吗？那里的能源贫困率如何？是否存在社会剥夺现象？我们是否应当优先开发废弃的褐地（Brownfield）？

为了回答这些问题，我带领团队构建了一个基于 Python 的地理空间分析与可视化系统。我们没有止步于简单的数据展示，而是将治安数据、社会剥夺指数、褐地数据与住房规划进行了深度融合，并通过自定义 JavaScript 实现了一个动态权重的“脆弱性热力图”。

这篇文章将复盘整个技术实现过程，希望能给对 GIS（地理信息系统）和数据可视化感兴趣的朋友提供一些思路。

---

## 数据的孤岛与桥梁

在开始写代码之前，我们面临的最大挑战是数据源的异构性。我们手头有五种完全不同格式的数据：

1.  **住房规划数据（Housing Pipeline）**：这是核心数据，Excel 格式，包含项目名称、开发商、邮编、规划单元数等。
2.  **行政边界数据（Boundaries）**：GeoJSON 格式，定义了利物浦城市地区的地理边界。
3.  **LSOA 统计分区数据**：LSOA（Lower Layer Super Output Areas）是英国统计局定义的小地理单元，是我们进行空间分析的基准。
4.  **多重剥夺指数（IMD）与燃料贫困数据**：Excel 格式，提供了每个 LSOA 的社会经济指标。
5.  **警方治安数据**：这是最棘手的部分。它是按月分文件夹存储的数百个 CSV 文件，包含具体的经纬度坐标。
6.  **褐地（Brownfield）数据**：GeoJSON 格式，包含废弃土地的位置和面积。

我们的目标是将这些散落在不同维度的“孤岛”数据，全部统一映射到 **LSOA** 这个地理单元上。

### 数据清洗与地理编码

住房规划数据中只有邮编（Postcode），没有经纬度。这是地理分析中常见的问题。我们使用了 `pgeocode` 库来解决这个问题，它基于 GeoNames 数据库，查询速度极快，不需要调用昂贵的 Google Maps API。

```python
import pgeocode
import pandas as pd

# 初始化英国的邮编查询对象
nomi = pgeocode.Nominatim("GB")

def batch_geocode(df, postcode_col):
    # 清洗邮编格式
    clean_postcodes = df[postcode_col].astype(str).str.strip()
    unique_pcs = clean_postcodes.unique()
    
    # 批量查询，提高效率
    geo_results = nomi.query_postal_code(unique_pcs)
    
    # 建立映射字典
    pc_to_lat = dict(zip(unique_pcs, geo_results.latitude))
    pc_to_lon = dict(zip(unique_pcs, geo_results.longitude))
    
    df["lat"] = clean_postcodes.map(pc_to_lat)
    df["lon"] = clean_postcodes.map(pc_to_lon)
    
    return df.dropna(subset=["lat", "lon"])
```

这一步至关重要，它将业务数据（Excel）转化为了空间数据（Points）。

### 处理海量治安数据

警方提供的数据非常详尽，涵盖了过去三年的每个月的犯罪记录，甚至包括“拦截搜查”（Stop & Search）的具体坐标。文件数量多达几百个。

如果我们直接读取所有 CSV 并试图在地图上画点，浏览器会瞬间崩溃。因此，**空间聚合（Spatial Aggregation）** 是唯一的出路。我们需要计算出每个 LSOA 区域内发生了多少起犯罪。

这里我们使用了 `pathlib` 进行递归文件查找，并结合 Pandas 进行数据清洗：

```python
from pathlib import Path

# 递归查找所有 CSV 文件
all_csvs = list(Path("police_data").rglob("*.csv"))
stop_search_points = []

for csv_file in all_csvs:
    try:
        # 只读取必要的经纬度列，节省内存
        df = pd.read_csv(csv_file, usecols=["Latitude", "Longitude"])
        df = df.dropna()
        # 粗略过滤掉不在利物浦范围内的点，加速后续处理
        mask = (df['Latitude'].between(53.0, 54.0)) & \
               (df['Longitude'].between(-3.5, -2.0))
        stop_search_points.append(df[mask])
    except:
        continue

# 合并所有数据帧
all_stops = pd.concat(stop_search_points, ignore_index=True)
```

---

## 空间连接

现在我们有了一堆犯罪地点的“点”，和一堆代表社区的“面”（LSOA Polygons）。我们要回答的问题是：**这个点属于哪个面？**

在 `geopandas` 中，`sjoin`（Spatial Join）函数是处理这类问题的神器。它是基于几何关系的数据库连接操作。

```python
import geopandas as gpd

# 将警方数据转换为 GeoDataFrame
gdf_stops = gpd.GeoDataFrame(
    all_stops, 
    geometry=gpd.points_from_xy(all_stops.Longitude, all_stops.Latitude),
    crs="EPSG:4326"
)

# 核心操作：判断点在哪个多边形内 (predicate='within')
# lsoa_lcr 是我们的行政区划多边形数据
stops_with_lsoa = gpd.sjoin(gdf_stops, lsoa_lcr[['LSOA21CD', 'geometry']], predicate='within')

# 聚合统计：计算每个 LSOA 的犯罪数量
stop_counts = stops_with_lsoa['LSOA21CD'].value_counts().reset_index()
stop_counts.columns = ['LSOA21CD', 'StopSearch_Count']
```

通过这一步，我们将数以万计的离散坐标点，转化为了每个社区的单一统计指标。这是数据降维的关键。

同理，我们将燃料贫困数据（Fuel Poverty）和多重剥夺指数（IMD）通过 `LSOA21CD` 代码 merge 到我们的主 GeoDataFrame 中。

---

## 构建“脆弱性评分”与动态归一化

有了原始数据后，我们面临一个新问题：不同指标的量纲完全不同。犯罪数量可能是 0 到 500，而贫困率是 0% 到 100%，IMD 是 1 到 10 的等级。为了在一个地图上综合展示这些信息，我们需要进行**归一化（Normalization）**。

我们采用了 Min-Max 归一化，将所有指标映射到 0 到 1 之间。值得注意的是，对于 IMD（1 代表最贫困），我们需要反转它，使得 1 代表“高风险/高脆弱性”。

```python
# 归一化燃料贫困率
f_min, f_max = master_gdf["Fuel_Poverty_Rate"].min(), master_gdf["Fuel_Poverty_Rate"].max()
master_gdf["norm_fuel"] = (master_gdf["Fuel_Poverty_Rate"] - f_min) / (f_max - f_min)

# 归一化 IMD (反转，10为最不贫困，1为最贫困 -> 映射后 1.0 为高风险)
master_gdf["norm_imd"] = (11 - master_gdf["IMD_Decile"]) / 10.0

# 归一化犯罪数据 (使用 95 分位值作为最大值，防止极端异常值拉低整体区分度)
c_max = master_gdf["Crime_Count"].quantile(0.95)
master_gdf["norm_crime"] = (master_gdf["Crime_Count"] / c_max).clip(upper=1.0)
```

**为什么使用 95 分位数？**
在实际的城市数据中，市中心（City Centre）的犯罪率往往是郊区的几十倍。如果直接使用最大值进行归一化，市中心会是深红色，而其他所有区域都会变成近乎白色的浅色，导致郊区之间的差异无法体现。截断极端值可以保留大部分区域的对比度。

---

## 可视化

如果只是用 `matplotlib` 画一张静态图，那对于决策者来说意义有限。我们需要的是交互性。我们选择了 `Folium`（基于 Leaflet.js），但原生的 Folium 功能有限，无法实现“拖动滑块实时改变权重”的功能。

为了解决这个问题，我采用了**注入自定义 JavaScript** 的方案。

### 图层分级策略

在地图上叠加多个图层时，遮挡是最大的敌人。我们通过自定义 Pane 并设置 `z-index` 来严格控制图层层级：

1.  **LSOA 基础着色层 (z=400)**：最底层，显示脆弱性热力图。
2.  **住房项目层 (z=620)**：中间层，显示规划中的新房（气泡图）。
3.  **褐地数据层 (z=650)**：最顶层，显示可用的废弃土地点位。

```python
import folium

m = folium.Map(location=[centre_lat, centre_lon], zoom_start=11, tiles="CartoDB positron")

# 创建自定义图层面板
folium.map.CustomPane("poly_pane", z_index=400).add_to(m)
folium.map.CustomPane("housing_pane", z_index=620).add_to(m)
folium.map.CustomPane("brownfield_pane", z_index=650).add_to(m)
```

### 注入 JavaScript 实现动态权重计算

这是本项目最核心的创新点。我们不希望仅仅展示一张死板的地图，而是希望让用户（比如警察局长、城市规划师或社会工作者）根据他们的关注点调整权重。

我们将处理好的 GeoJSON 数据和前端逻辑直接写入 HTML 中。

```python
from branca.element import Element

# 将 Python 处理好的数据转为 JSON 字符串注入前端
geojson_str = master_gdf.to_json()

custom_ui = f"""
<div id="controls">
    <h4>Layer Weights (LSOA Scoring)</h4>
    <!-- 滑块控件 -->
    <div><label>Fuel Poverty</label><input type="range" id="w_fuel" oninput="updateWeights()"></div>
    <div><label>Crime Density</label><input type="range" id="w_crime" oninput="updateWeights()"></div>
    <!-- ... 其他滑块 ... -->
</div>

<script>
    var lsoaData = {geojson_str}; // 注入数据
    
    // 动态计算分数的函数
    function updateWeights() {{
        // 获取滑块的值
        var w_fuel = parseFloat(document.getElementById('w_fuel').value);
        var w_crime = parseFloat(document.getElementById('w_crime').value);
        
        // 遍历每个地理单元重新计算颜色
        geojsonLayer.eachLayer(function(layer) {{
            var p = layer.feature.properties;
            // 加权平均公式
            var score = (p.norm_fuel * w_fuel + p.norm_crime * w_crime) / (w_fuel + w_crime);
            
            // 动态设置样式
            layer.setStyle({{ fillColor: getColor(score) }});
        }});
    }}
</script>
"""
m.get_root().html.add_child(Element(custom_ui))
```

这段代码实现了一个完全前端的交互逻辑：数据在 Python 后端处理完毕后，前端浏览器负责实时的渲染和数学计算。这保证了地图的流畅性，用户在拖动滑块时，地图颜色会平滑过渡，直观地展示出不同侧重下的高风险区域。

### 住房规划与褐地的可视化细节

对于住房项目，我们使用 `CircleMarker`。
*   **大小**：代表规划的房屋数量（使用了平方根缩放，避免气泡过大）。
*   **填充颜色**：代表开发阶段（短期、中期、长期）。
*   **边框颜色**：代表地块面积大小。

对于褐地（Brownfield），我们将其作为最顶层的黑色小点显示，点击后会弹出详细的规划许可状态和此前用途。这样的设计让规划者可以一目了然地看到：**在这个高风险（红色）区域，是否有可利用的褐地（黑点）来建设新房，从而起到旧城改造的作用？**