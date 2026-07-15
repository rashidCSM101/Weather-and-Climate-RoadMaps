# 🌍 regionmask + geopandas — Complete Learning Roadmap
## Spatial Masking, Country/Region Boundaries & Shapefile Analysis

---

## 📚 What are these libraries?

| Library | Purpose | Key Use |
|---------|---------|---------|
| **regionmask** | Mask gridded data to geographic regions | Extract Pakistan from ERA5 global data |
| **geopandas** | Geospatial analysis with shapefiles | Pakistan district boundaries, spatial statistics |

---

## 🗺️ Roadmap Overview

```
REGIONMASK:
  Level 1: Built-in Regions        → Countries, SREX, IPCC AR6 regions
  Level 2: Masking Gridded Data    → Apply mask to ERA5, CMIP6
  Level 3: Custom Regions          → User-defined polygons
  Level 4: Multiple Regions        → Regional comparisons

GEOPANDAS:
  Level 5: GeoDataFrame Basics     → Loading shapefiles, CRS
  Level 6: Spatial Analysis        → Spatial joins, area statistics
  Level 7: Raster-Vector Analysis  → Zonal statistics with climate data
  Level 8: Visualization           → Choropleth maps, district-level plots
```

---

## 📁 REGIONMASK — LEVEL 1: Built-in Regions

### 1.1 Available Region Definitions

```python
import regionmask
import xarray as xr
import numpy as np
import matplotlib.pyplot as plt
import cartopy.crs as ccrs

# ---- Built-in region sets ----

# 1. Natural Earth countries (most useful!)
countries = regionmask.defined_regions.natural_earth_v5_0_0.countries_110
# Options: countries_10, countries_50, countries_110 (110m = coarsest/fastest)

# 2. SREX Regions (IPCC Special Report on Extreme Events — 26 regions)
srex = regionmask.defined_regions.srex
# Includes: SAS (South Asia), WAS (West Asia), etc.

# 3. IPCC AR6 Reference Regions (new standard — 58 regions)
ar6 = regionmask.defined_regions.ar6.all
ar6_land = regionmask.defined_regions.ar6.land
ar6_ocean = regionmask.defined_regions.ar6.ocean

# 4. Continental regions
continents = regionmask.defined_regions.natural_earth_v5_0_0.land_110

# ---- Inspect regions ----
print(countries)              # Shows region list with indices
print(countries.names)        # Names of all countries
print(countries.abbrevs)      # Abbreviations
print(len(countries))         # Number of regions

# Find specific country index
pak_idx = countries.map_keys("Pakistan")
print(f"Pakistan index: {pak_idx}")   # e.g., 61

# Get region object
pak_region = countries["Pakistan"]
print(pak_region.bounds)   # [west, south, east, north]
```

### 1.2 Plotting Available Regions

```python
# Plot all countries
fig, ax = plt.subplots(subplot_kw={"projection": ccrs.PlateCarree()},
                       figsize=(14, 7))
countries.plot(ax=ax, add_label=False, line_kws={"linewidth": 0.5})
ax.set_global()
plt.title("Natural Earth Countries")
plt.show()

# Plot SREX regions
fig, ax = plt.subplots(subplot_kw={"projection": ccrs.Robinson()},
                       figsize=(12, 6))
srex.plot(ax=ax, add_label=True, label="abbrev")
ax.set_global()
plt.title("SREX Regions (IPCC AR4/AR5)")

# Plot AR6 regions
fig, ax = plt.subplots(subplot_kw={"projection": ccrs.Robinson()},
                       figsize=(12, 6))
ar6_land.plot(ax=ax, add_label=True, label="abbrev",
              text_kws={"fontsize": 7})
```

---

## 📁 REGIONMASK — LEVEL 2: Masking Gridded Data

### 2.1 Basic Masking

```python
import regionmask
import xarray as xr
import numpy as np

# Load ERA5 data
ds = xr.open_dataset("ERA5_t2m_2020.nc")
da = ds["t2m"] - 273.15

lat = da.lat.values
lon = da.lon.values

# Define region set
countries = regionmask.defined_regions.natural_earth_v5_0_0.countries_110

# Create mask (2D array — same lat/lon as data)
mask = countries.mask(lon, lat)
# mask values: 0, 1, 2, ... = region indices; NaN = no region (ocean)

print(mask)
print(mask.shape)   # (lat, lon)
print(np.unique(mask.values[~np.isnan(mask.values)]))  # All region indices

# Get Pakistan index
pak_idx = countries.map_keys("Pakistan")
print(f"Pakistan index: {pak_idx}")
```

### 2.2 Extracting a Single Country

```python
# Method 1: Create mask and apply where()
countries = regionmask.defined_regions.natural_earth_v5_0_0.countries_110
mask = countries.mask(da)   # xarray-aware version (pass DataArray or Dataset)

pak_idx = countries.map_keys("Pakistan")
pak_mask = mask == pak_idx

# Extract Pakistan data
pak_data = da.where(pak_mask)    # Non-Pakistan = NaN
pak_data_clean = da.where(pak_mask).dropna("lat", how="all").dropna("lon", how="all")

# Time series: area-weighted mean over Pakistan
weights = np.cos(np.deg2rad(da.lat))
pak_ts = pak_data.weighted(weights).mean(["lat", "lon"])
print(pak_ts)   # Time series for Pakistan

# Seasonal analysis
pak_monthly = pak_ts.groupby("time.month").mean("time")
pak_annual = pak_ts.resample(time="1Y").mean()
```

### 2.3 Multiple Countries / Regions

```python
# Extract multiple countries simultaneously
countries_of_interest = ["Pakistan", "India", "Afghanistan", "Iran", "Bangladesh"]

results = {}
for country in countries_of_interest:
    idx = countries.map_keys(country)
    mask_c = mask == idx
    weights = np.cos(np.deg2rad(da.lat))
    ts = da.where(mask_c).weighted(weights).mean(["lat", "lon"])
    results[country] = ts

# Plot comparison
fig, ax = plt.subplots(figsize=(12, 5))
for country, ts in results.items():
    annual = ts.resample(time="1Y").mean()
    ax.plot(annual.time.values, annual.values, label=country)
ax.legend()
ax.set_ylabel("Temperature (°C)")
ax.set_title("Annual Mean Temperature by Country")
plt.tight_layout()
plt.savefig("country_comparison.png", dpi=150)
```

### 2.4 SREX Regional Masks

```python
srex = regionmask.defined_regions.srex

# South Asia (SAS) region
sas_idx = srex.map_keys("SAS")
mask_srex = srex.mask(da)
sas_data = da.where(mask_srex == sas_idx)

# All SREX regions loop
fig, axes = plt.subplots(5, 6, figsize=(20, 16),
                         subplot_kw={"projection": ccrs.PlateCarree()})
for idx, ax in enumerate(axes.flat):
    if idx >= len(srex):
        ax.axis("off")
        continue
    region_mask = mask_srex == idx
    region_ts = da.where(region_mask).mean(["lat", "lon"])
    # ... plot each region
```

### 2.5 3D Mask (Time Dimension)

```python
# For Dataset with (time, lat, lon), apply mask correctly
# mask is (lat, lon), data is (time, lat, lon) → broadcasting works!

mask = countries.mask(da)
pak_idx = countries.map_keys("Pakistan")

# This works automatically due to xarray broadcasting
pak_data = da.where(mask == pak_idx)  # (time, lat, lon) — non-Pakistan = NaN

# Save masked data
pak_data.to_netcdf("ERA5_Pakistan.nc")
```

### 2.6 Custom Region Mask

```python
# Define your own polygon (e.g., Indus Basin, northern Pakistan)
from shapely.geometry import Polygon
import geopandas as gpd

# Approximate Indus Basin polygon (longitude, latitude)
indus_basin = Polygon([
    (67, 22), (68, 28), (70, 35), (75, 37), (80, 35),
    (76, 28), (73, 22), (67, 22)
])

# Create regionmask from polygon
custom_region = regionmask.Regions(
    outlines=[indus_basin],
    names=["Indus Basin"],
    abbrevs=["IND"],
    name="Custom Regions"
)

mask_custom = custom_region.mask(da)
indus_data = da.where(mask_custom == 0)   # 0 = first (only) region
```

---

## 📁 GEOPANDAS — LEVEL 5: Basics

### 5.1 Loading Shapefiles

```python
import geopandas as gpd
import pandas as pd
import matplotlib.pyplot as plt

# Load Pakistan district boundaries (shapefile)
pak_districts = gpd.read_file("pakistan_districts.shp")
print(pak_districts.head())
print(pak_districts.columns.tolist())
print(pak_districts.crs)              # Coordinate reference system
print(pak_districts.geometry.type)   # Polygon/MultiPolygon

# Check fields
print(pak_districts[["DISTRICT", "PROVINCE", "AREA_KM2"]].head(10))

# Load province boundaries
pak_provinces = gpd.read_file("pakistan_provinces.shp")
```

### 5.2 CRS (Coordinate Reference Systems)

```python
# Common CRS for Pakistan/South Asia:
# WGS84 (EPSG:4326)  — Geographic, lat/lon degrees — use for climate data
# UTM Zone 42N (EPSG:32642) — Projected, meters — use for area calculations
# UTM Zone 43N (EPSG:32643) — For eastern Pakistan

# Check current CRS
print(pak_districts.crs)          # EPSG:4326 or similar

# Reproject
pak_utm = pak_districts.to_crs("EPSG:32642")   # For accurate area in m²

# Calculate area (must be in projected CRS)
pak_utm["area_km2"] = pak_utm.geometry.area / 1e6   # m² → km²
print(pak_utm[["DISTRICT", "area_km2"]].sort_values("area_km2", ascending=False))
```

### 5.3 Basic GeoDataFrame Operations

```python
# Filtering
sindh = pak_districts[pak_districts["PROVINCE"] == "Sindh"]
punjab = pak_districts[pak_districts["PROVINCE"] == "Punjab"]

# Dissolve (merge) districts to province level
provinces = pak_districts.dissolve(by="PROVINCE")

# Centroid (geometric center of each district)
pak_districts["centroid"] = pak_districts.geometry.centroid
pak_districts["lon_center"] = pak_districts.centroid.x
pak_districts["lat_center"] = pak_districts.centroid.y

# Bounding box
bbox = pak_districts.total_bounds   # [west, south, east, north]
print(f"Pakistan extent: {bbox}")

# Spatial index for fast queries
pak_districts.sindex  # STRtree spatial index
```

---

## 📁 GEOPANDAS — LEVEL 6: Spatial Analysis

### 6.1 Zonal Statistics (Extract Climate Values per District)

```python
import xarray as xr
import geopandas as gpd
import numpy as np
from shapely.geometry import Point

# Load data
ds = xr.open_dataset("ERA5_Pakistan.nc")
da = ds["t2m"] - 273.15
pak_districts = gpd.read_file("pakistan_districts.shp")

# Method 1: Point-in-polygon (station-to-district assignment)
stations = pd.DataFrame({
    "station": ["Islamabad", "Karachi", "Lahore"],
    "lat": [33.73, 24.86, 31.52],
    "lon": [73.09, 67.01, 74.35],
    "tmax_trend": [0.03, 0.04, 0.035]
})

# Create GeoDataFrame from station points
gdf_stations = gpd.GeoDataFrame(
    stations,
    geometry=gpd.points_from_xy(stations.lon, stations.lat),
    crs="EPSG:4326"
)

# Spatial join: assign district to each station
gdf_joined = gpd.sjoin(gdf_stations, pak_districts, how="left", predicate="within")
print(gdf_joined[["station", "DISTRICT", "PROVINCE", "tmax_trend"]])
```

### 6.2 Raster-to-District Zonal Statistics

```python
# Install: pip install exactextract (or rasterstats)
import exactextract as ee  # or use rasterstats

# With rasterstats:
from rasterstats import zonal_stats
import rioxarray

# Save DataArray as GeoTIFF first
da_mean = da.mean("time")
da_mean.rio.set_spatial_dims(x_dim="lon", y_dim="lat", inplace=True)
da_mean.rio.write_crs("EPSG:4326", inplace=True)
da_mean.rio.to_raster("mean_temp.tif")

# Zonal statistics per district
stats = zonal_stats(
    pak_districts,
    "mean_temp.tif",
    stats=["mean", "min", "max", "std", "count"],
    geojson_out=True
)

# Merge results back to GeoDataFrame
stats_df = pd.DataFrame(stats)
pak_districts["mean_temp"] = [s["mean"] for s in stats]
pak_districts["max_temp"]  = [s["max"] for s in stats]
pak_districts["min_temp"]  = [s["min"] for s in stats]
```

---

## 📁 GEOPANDAS — LEVEL 7: Visualization

### 7.1 Choropleth Maps (Climate Data per District)

```python
import geopandas as gpd
import matplotlib.pyplot as plt
import matplotlib.colors as mcolors
import cartopy.crs as ccrs

pak_districts = gpd.read_file("pakistan_districts.shp")
# pak_districts["mean_temp"] = ... (from zonal stats above)

# --- Simple choropleth ---
fig, ax = plt.subplots(figsize=(10, 12))
pak_districts.plot(
    column="mean_temp",
    ax=ax,
    cmap="YlOrRd",
    legend=True,
    legend_kwds={"label": "Mean Annual Temperature (°C)", "shrink": 0.7},
    edgecolor="gray",
    linewidth=0.3,
    missing_kwds={"color": "lightgray"},
)
ax.set_title("Annual Mean Temperature by District\nPakistan, 1981–2010",
             fontsize=14, fontweight="bold")
ax.axis("off")
plt.tight_layout()
plt.savefig("pakistan_district_temp.png", dpi=300, bbox_inches="tight")

# --- Multi-panel (seasonal) ---
seasons = {"DJF": "Winter", "MAM": "Spring", "JJA": "Summer", "SON": "Autumn"}
fig, axes = plt.subplots(2, 2, figsize=(16, 18))
for ax, (season, title) in zip(axes.flat, seasons.items()):
    pak_districts.plot(
        column=f"temp_{season}",
        ax=ax,
        cmap="RdYlBu_r",
        legend=True,
        edgecolor="gray",
        linewidth=0.3,
    )
    ax.set_title(title, fontsize=12, fontweight="bold")
    ax.axis("off")
plt.suptitle("Seasonal Temperature by District", fontsize=15, fontweight="bold")
plt.tight_layout()
plt.savefig("pakistan_seasonal_district.png", dpi=300, bbox_inches="tight")
```

### 7.2 Province Labels on Map

```python
# Add district/province labels at centroids
fig, ax = plt.subplots(figsize=(10, 12))
pak_provinces.plot(ax=ax, facecolor="lightblue", edgecolor="black", linewidth=0.8)
pak_districts.plot(ax=ax, facecolor="none", edgecolor="gray", linewidth=0.3)

# Label provinces
for _, row in pak_provinces.iterrows():
    centroid = row.geometry.centroid
    ax.annotate(
        row["PROVINCE"],
        xy=(centroid.x, centroid.y),
        ha="center", va="center",
        fontsize=10, fontweight="bold",
        color="darkblue"
    )

ax.set_title("Pakistan Administrative Boundaries")
ax.axis("off")
plt.savefig("pakistan_map.png", dpi=300, bbox_inches="tight")
```

---

## 📋 regionmask Quick Reference

| Operation | Code |
|-----------|------|
| Load countries | `regionmask.defined_regions.natural_earth_v5_0_0.countries_110` |
| Load SREX | `regionmask.defined_regions.srex` |
| Load AR6 | `regionmask.defined_regions.ar6.land` |
| Create mask | `countries.mask(da)` |
| Get country index | `countries.map_keys("Pakistan")` |
| Extract country | `da.where(mask == pak_idx)` |
| Area-weighted mean | `da.where(mask==idx).weighted(weights).mean(["lat","lon"])` |
| Custom region | `regionmask.Regions(outlines=[polygon], names=["Name"])` |

## 📋 geopandas Quick Reference

| Operation | Code |
|-----------|------|
| Load shapefile | `gpd.read_file("file.shp")` |
| Check CRS | `gdf.crs` |
| Reproject | `gdf.to_crs("EPSG:32642")` |
| Filter rows | `gdf[gdf["PROVINCE"]=="Punjab"]` |
| Dissolve to province | `gdf.dissolve(by="PROVINCE")` |
| Area (m²) | `gdf.geometry.area` |
| Spatial join | `gpd.sjoin(points_gdf, polygons_gdf)` |
| Plot choropleth | `gdf.plot(column="variable", cmap="RdBu_r", legend=True)` |
| Save shapefile | `gdf.to_file("output.shp")` |
| Save GeoJSON | `gdf.to_file("output.geojson", driver="GeoJSON")` |

---

## 📚 Resources

- [regionmask Documentation](https://regionmask.readthedocs.io)
- [geopandas Documentation](https://geopandas.org/en/stable/)
- [Pakistan Boundary Data](https://www.gadm.org) — Free admin boundaries
- [SREX Region Definitions](https://www.ipcc.ch/report/managing-the-risks-of-extreme-events-and-disasters-to-advance-climate-change-adaptation/)
- [IPCC AR6 Reference Regions](https://github.com/IPCC-WG1/Atlas)
