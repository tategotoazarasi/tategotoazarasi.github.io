---
date: '2025-12-03T20:32:25Z'
draft: false
summary: 'This post details how to build an interactive urban planning visualization system using Python, Geopandas, and Folium by integrating housing pipelines, crime data, and socioeconomic metrics.'
title: 'Building a Data-Driven City: Integrating Housing, Safety, and Deprivation Metrics with Python'
tags: ['python', 'geopandas', 'folium', 'data-visualization', 'gis', 'urban-planning', 'spatial-analysis', 'smart-city', 'data-science', 'hackathon']
---

Recently, I had the opportunity to participate in a hackathon organized jointly by the Liverpool City Region Combined Authority (LCR CA) and Homes England. The context was both ambitious and pragmatic: the government holds a housing pipeline list containing approximately 350 sites, totalling over 57,000 new homes. However, this data was sitting quietly in Excel spreadsheets, existing merely as cold numbers.

Our task was: **How do we make this data "come alive"?**

As developers, we know that simply plotting "here is a house" on a map is far from sufficient. Urban planning is a complex systemic engineering challenge. Where are we building? Is that area safe? What is the fuel poverty rate there? Is there social deprivation? Should we prioritize developing abandoned "Brownfield" sites?

To answer these questions, my team and I built a geospatial analysis and visualization system based on Python. We didn't stop at simple data display; instead, we deeply integrated crime statistics, deprivation indices, brownfield data, and housing plans. Furthermore, we implemented a "vulnerability heatmap" with dynamic weighting using custom JavaScript.

This post will review the entire technical implementation process, hopefully providing some insights for those interested in GIS (Geographic Information Systems) and data visualization.

---

## Data Silos and Bridges

Before writing a single line of code, our biggest challenge was the heterogeneity of our data sources. We had five completely different formats:

1.  **Housing Pipeline Data**: The core dataset, in Excel format, containing project names, developers, postcodes, unit counts, etc.
2.  **Boundaries**: GeoJSON format, defining the geographic boundaries of the Liverpool City Region.
3.  **LSOA Data**: Lower Layer Super Output Areas (LSOA) are small geographic units defined by the UK Office for National Statistics, serving as our baseline for spatial analysis.
4.  **Index of Multiple Deprivation (IMD) & Fuel Poverty**: Excel format, providing socioeconomic indicators for each LSOA.
5.  **Police Crime Data**: The trickiest part. Hundreds of CSV files stored in monthly folders, containing specific latitude/longitude coordinates.
6.  **Brownfield Data**: GeoJSON format, containing the location and area of abandoned land.

Our goal was to map these "siloed" datasets from different dimensions onto a unified geographic unit: the **LSOA**.

### Data Cleaning and Geocoding

The housing pipeline data only contained postcodes, not coordinates. This is a common issue in geospatial analysis. We used the `pgeocode` library to solve this. It relies on the GeoNames database, offering extremely fast queries without the need for expensive API calls like Google Maps.

```python
import pgeocode
import pandas as pd

# Initialize UK postcode query object
nomi = pgeocode.Nominatim("GB")

def batch_geocode(df, postcode_col):
    # Clean postcode format
    clean_postcodes = df[postcode_col].astype(str).str.strip()
    unique_pcs = clean_postcodes.unique()
    
    # Batch query for efficiency
    geo_results = nomi.query_postal_code(unique_pcs)
    
    # Create mapping dictionaries
    pc_to_lat = dict(zip(unique_pcs, geo_results.latitude))
    pc_to_lon = dict(zip(unique_pcs, geo_results.longitude))
    
    df["lat"] = clean_postcodes.map(pc_to_lat)
    df["lon"] = clean_postcodes.map(pc_to_lon)
    
    return df.dropna(subset=["lat", "lon"])
```

This step was crucial as it converted business data (Excel) into spatial data (Points).

### Handling Massive Police Data

The data provided by the police was exhaustive, covering every month of crime records for the past three years, including coordinates for "Stop & Search" events. There were hundreds of files.

If we tried to read all CSVs and plot every point on a map, the browser would crash instantly. Therefore, **Spatial Aggregation** was the only way forward. We needed to calculate how many crimes occurred within each LSOA.

We used `pathlib` for recursive file searching combined with Pandas for cleaning:

```python
from pathlib import Path

# Recursively find all CSV files
all_csvs = list(Path("police_data").rglob("*.csv"))
stop_search_points = []

for csv_file in all_csvs:
    try:
        # Read only necessary coordinate columns to save memory
        df = pd.read_csv(csv_file, usecols=["Latitude", "Longitude"])
        df = df.dropna()
        # Roughly filter points outside Liverpool to speed up processing
        mask = (df['Latitude'].between(53.0, 54.0)) & \
               (df['Longitude'].between(-3.5, -2.0))
        stop_search_points.append(df[mask])
    except:
        continue

# Concatenate all dataframes
all_stops = pd.concat(stop_search_points, ignore_index=True)
```

---

## Spatial Join

Now we had a pile of "points" (crime locations) and a pile of "polygons" (LSOA communities). The question we needed to answer was: **Which polygon does this point belong to?**

In `geopandas`, the `sjoin` (Spatial Join) function is the ultimate tool for this. It performs a database join operation based on geometric relationships.

```python
import geopandas as gpd

# Convert police data to GeoDataFrame
gdf_stops = gpd.GeoDataFrame(
    all_stops, 
    geometry=gpd.points_from_xy(all_stops.Longitude, all_stops.Latitude),
    crs="EPSG:4326"
)

# Core operation: Check which polygon contains the point (predicate='within')
# lsoa_lcr is our administrative boundary polygon data
stops_with_lsoa = gpd.sjoin(gdf_stops, lsoa_lcr[['LSOA21CD', 'geometry']], predicate='within')

# Aggregation: Count crimes per LSOA
stop_counts = stops_with_lsoa['LSOA21CD'].value_counts().reset_index()
stop_counts.columns = ['LSOA21CD', 'StopSearch_Count']
```

Through this step, we transformed tens of thousands of discrete coordinate points into a single statistical metric for each community. This is the key to dimensionality reduction in spatial data.

Similarly, we merged Fuel Poverty data and Index of Multiple Deprivation (IMD) data into our master GeoDataFrame using the `LSOA21CD` code.

---

## Constructing a "Vulnerability Score" & Dynamic Normalization

With the raw data in hand, we faced a new problem: the dimensions of the metrics were completely different. Crime counts might range from 0 to 500, poverty rates from 0% to 100%, and IMD is a rank from 1 to 10. To display these comprehensively on a single map, we needed **Normalization**.

We used Min-Max normalization to map all metrics to a range between 0 and 1. Notably, for IMD (where 1 represents the most deprived), we needed to invert it so that 1.0 represents "High Risk/Vulnerability".

```python
# Normalize Fuel Poverty Rate
f_min, f_max = master_gdf["Fuel_Poverty_Rate"].min(), master_gdf["Fuel_Poverty_Rate"].max()
master_gdf["norm_fuel"] = (master_gdf["Fuel_Poverty_Rate"] - f_min) / (f_max - f_min)

# Normalize IMD (Invert: 10 is least deprived, 1 is most -> mapped so 1.0 is high risk)
master_gdf["norm_imd"] = (11 - master_gdf["IMD_Decile"]) / 10.0

# Normalize Crime Data (Use 95th percentile as max to prevent outliers skewing the scale)
c_max = master_gdf["Crime_Count"].quantile(0.95)
master_gdf["norm_crime"] = (master_gdf["Crime_Count"] / c_max).clip(upper=1.0)
```

**Why use the 95th percentile?**
In real-world urban data, crime rates in the City Centre are often orders of magnitude higher than in the suburbs. If we normalized using the absolute maximum, the City Centre would be deep red, while everywhere else would wash out to white, making suburban variations invisible. Clipping outliers preserves contrast for the majority of the region.

---

## Visualization

A static chart generated by `matplotlib` is of limited use to decision-makers. We needed interactivity. We chose `Folium` (based on Leaflet.js), but native Folium functionality is limited—it doesn't support "dragging a slider to change weights in real-time."

To solve this, I adopted a strategy of **injecting custom JavaScript**.

### Layer Z-Index Management

When overlaying multiple layers on a map, occlusion is the enemy. We strictly controlled layer hierarchy by creating custom Panes and setting `z-index`:

1.  **LSOA Base Layer (z=400)**: Bottom layer, displaying the vulnerability heatmap.
2.  **Housing Project Layer (z=620)**: Middle layer, displaying planned new homes (bubble chart).
3.  **Brownfield Layer (z=650)**: Top layer, displaying available abandoned land points.

```python
import folium

m = folium.Map(location=[centre_lat, centre_lon], zoom_start=11, tiles="CartoDB positron")

# Create custom panes
folium.map.CustomPane("poly_pane", z_index=400).add_to(m)
folium.map.CustomPane("housing_pane", z_index=620).add_to(m)
folium.map.CustomPane("brownfield_pane", z_index=650).add_to(m)
```

### Injecting JavaScript for Dynamic Weighting

This was the core innovation of the project. We didn't want to show a rigid map; we wanted to let users (like police chiefs, urban planners, or social workers) adjust weights based on their priorities.

We wrote the processed GeoJSON data and frontend logic directly into the HTML.

```python
from branca.element import Element

# Convert Python processed data to JSON string for frontend injection
geojson_str = master_gdf.to_json()

custom_ui = f"""
<div id="controls">
    <h4>Layer Weights (LSOA Scoring)</h4>
    <!-- Sliders -->
    <div><label>Fuel Poverty</label><input type="range" id="w_fuel" oninput="updateWeights()"></div>
    <div><label>Crime Density</label><input type="range" id="w_crime" oninput="updateWeights()"></div>
    <!-- ... other sliders ... -->
</div>

<script>
    var lsoaData = {geojson_str}; // Inject data
    
    // Function to calculate score dynamically
    function updateWeights() {{
        // Get slider values
        var w_fuel = parseFloat(document.getElementById('w_fuel').value);
        var w_crime = parseFloat(document.getElementById('w_crime').value);
        
        // Iterate through features and recalculate color
        geojsonLayer.eachLayer(function(layer) {{
            var p = layer.feature.properties;
            // Weighted average formula
            var score = (p.norm_fuel * w_fuel + p.norm_crime * w_crime) / (w_fuel + w_crime);
            
            // Set style dynamically
            layer.setStyle({{ fillColor: getColor(score) }});
        }});
    }}
</script>
"""
m.get_root().html.add_child(Element(custom_ui))
```

This code implements a fully frontend-based interaction logic: once the data is processed by the Python backend, the browser handles real-time rendering and mathematical calculation. This ensures fluidity; when users drag sliders, the map colors transition smoothly, visually revealing high-risk areas under different priorities.

### Visualization Details for Housing & Brownfields

For housing projects, we used `CircleMarker`.
*   **Size**: Represents the number of planned homes (using square root scaling to keep bubble sizes sane).
*   **Fill Color**: Represents development stage (Short, Medium, Long term).
*   **Border Color**: Represents site area size.

For Brownfields, we displayed them as small black dots on the topmost layer. Clicking them reveals details about planning permission status and prior use. This design allows planners to see at a glance: **In this high-risk (red) area, are there available brownfield sites (black dots) that could be used to build new homes, thereby catalyzing urban regeneration?**