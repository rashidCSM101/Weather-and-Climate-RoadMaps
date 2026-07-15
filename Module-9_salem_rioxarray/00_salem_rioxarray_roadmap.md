# 🗺️ salem + rioxarray — Complete Learning Roadmap
## Raster Operations, CRS Handling & Reprojection

---

## 📚 What are these libraries?

| Library | Purpose | Key Use |
|---------|---------|---------|
| **rioxarray** | Extends xarray with rasterio capabilities | CRS, reprojection, clipping, GeoTIFF I/O |
| **salem** | WRF/climate-focused geo-processing | WRF output, map projections, terrain data |

**rioxarray is the modern standard** — it's well-maintained and works seamlessly with xarray. Salem is more specialized for WRF model output.

---

## 🗺️ Roadmap Overview

```
RIOXARRAY:
  Level 1: Setup & Basics         → Spatial metadata, CRS assignment
  Level 2: CRS Operations         → Reprojection, coordinate transformation
  Level 3: Clipping & Masking     → Clip to shapefile, bounding box
  Level 4: Resampling             → Upsampling, downsampling, regridding
  Level 5: GeoTIFF Operations     → Read/write, DEM, satellite data
  Level 6: NetCDF → GeoTIFF       → Format conversion workflow

SALEM:
  Level 7: Salem Basics           → Installation, salem datasets
  Level 8: WRF Output Handling    → WRF projections, unstaggering
```

---

## 📁 RIOXARRAY — LEVEL 1: Setup & Basics

### 1.1 Installation & Imports

```bash
pip install rioxarray rasterio pyproj
```

```python
import xarray as xr
import rioxarray          # This PATCHES xarray with .rio accessor
import numpy as np
from pyproj import CRS

# After importing rioxarray, all xarray objects get a .rio accessor
# da.rio → gives access to all rioxarray methods
```

### 1.2 Assigning CRS (Coordinate Reference System)

```python
# Load climate data (ERA5, CMIP6, etc.)
ds = xr.open_dataset("ERA5_t2m.nc")
da = ds["t2m"]

# ERA5 and most climate data is in WGS84 (EPSG:4326)
# You MUST tell rioxarray about this

# Set spatial dimensions
da = da.rio.set_spatial_dims(
    x_dim="lon",      # Longitude dimension name
    y_dim="lat",      # Latitude dimension name
    inplace=True
)

# Write CRS
da = da.rio.write_crs("EPSG:4326", inplace=True)   # WGS84 geographic

# Verify
print(da.rio.crs)              # CRS object
print(da.rio.width)            # Number of longitude points
print(da.rio.height)           # Number of latitude points
print(da.rio.bounds())         # (west, south, east, north)
print(da.rio.transform())      # Affine transform matrix
print(da.rio.shape)            # (height, width) = (lat_count, lon_count)
print(da.rio.nodata)           # NoData value (if set)
```

### 1.3 Common CRS Reference

```python
from pyproj import CRS

# WGS84 Geographic (lat/lon) — most climate data
CRS("EPSG:4326")

# UTM Zone 42N — Pakistan west (Balochistan, KPK, Punjab west)
CRS("EPSG:32642")

# UTM Zone 43N — Pakistan east (Sindh, Punjab east)
CRS("EPSG:32643")

# Web Mercator (Google Maps)
CRS("EPSG:3857")

# Albers Equal Area (South Asia)
CRS("ESRI:102028")

# Define custom CRS from proj string
custom = CRS.from_proj4("+proj=lcc +lat_1=25 +lat_2=35 +lat_0=30 +lon_0=70")

# Check if CRS is geographic (lat/lon) or projected (meters)
print(CRS("EPSG:4326").is_geographic)    # True
print(CRS("EPSG:32642").is_projected)    # True
```

---

## 📁 RIOXARRAY — LEVEL 2: CRS Operations

### 2.1 Reprojection

```python
import rioxarray
import xarray as xr

# Load and setup
da = xr.open_dataarray("ERA5_Pakistan.nc")
da = da.rio.set_spatial_dims(x_dim="lon", y_dim="lat")
da = da.rio.write_crs("EPSG:4326")

# Reproject to UTM for area calculations
da_utm = da.rio.reproject("EPSG:32642")

# Now coordinates are in meters (Easting/Northing)
print(da_utm.coords)
print(da_utm.rio.crs)           # EPSG:32642
print(da_utm.rio.transform())   # Transform in meters

# Reproject to Lambert Conformal Conic
da_lcc = da.rio.reproject(
    "EPSG:4326",   # Always reproject FROM current CRS
    dst_crs="+proj=lcc +lat_1=25 +lat_2=35 +lat_0=30 +lon_0=70"
)

# Reproject to specific resolution
da_utm_1km = da.rio.reproject(
    "EPSG:32642",
    resolution=1000,          # 1000m = 1km
    resampling=rioxarray.enums.Resampling.bilinear
)
```

### 2.2 Coordinate Transformation

```python
from pyproj import Transformer

# Transform specific points between CRS
transformer = Transformer.from_crs("EPSG:4326", "EPSG:32642", always_xy=True)

# Single point (lon, lat) → (easting, northing)
lon, lat = 73.09, 33.73   # Islamabad
easting, northing = transformer.transform(lon, lat)
print(f"Islamabad: E={easting:.0f}m, N={northing:.0f}m")

# Array of points
lons = np.array([73.09, 67.01, 74.35])
lats = np.array([33.73, 24.86, 31.52])
eastings, northings = transformer.transform(lons, lats)

# Inverse: projected → geographic
transformer_inv = Transformer.from_crs("EPSG:32642", "EPSG:4326", always_xy=True)
lon_back, lat_back = transformer_inv.transform(easting, northing)
```

---

## 📁 RIOXARRAY — LEVEL 3: Clipping & Masking

### 3.1 Clip to Bounding Box

```python
# Clip to Pakistan bounding box
# Pakistan approx: 60-78°E, 23-37°N
da_pak = da.rio.clip_box(
    minx=60,    # Western longitude
    miny=23,    # Southern latitude
    maxx=78,    # Eastern longitude
    maxy=37     # Northern latitude
)

print(f"Original: {da.shape}")
print(f"Clipped: {da_pak.shape}")
```

### 3.2 Clip to Shapefile (Most Useful!)

```python
import geopandas as gpd
import rioxarray
import xarray as xr

# Load Pakistan boundary shapefile
pak_boundary = gpd.read_file("pakistan_boundary.shp")

# Ensure same CRS
pak_boundary = pak_boundary.to_crs("EPSG:4326")

# Setup rioxarray
da = xr.open_dataarray("ERA5_t2m.nc")
da = da.rio.set_spatial_dims(x_dim="lon", y_dim="lat")
da = da.rio.write_crs("EPSG:4326")

# Clip to exact Pakistan boundary
da_pak = da.rio.clip(
    pak_boundary.geometry,     # Shapefile geometries
    pak_boundary.crs,          # Shapefile CRS
    drop=True,                 # Drop rows/cols outside boundary
    all_touched=True,          # Include cells that touch boundary
)

print(da_pak)
print(f"Cells: {da_pak.count().values}")
```

### 3.3 Clip to Individual Province

```python
# Clip ERA5 to Punjab Province only
provinces = gpd.read_file("pakistan_provinces.shp")
punjab = provinces[provinces["NAME"] == "Punjab"]

da_punjab = da.rio.clip(
    punjab.geometry,
    punjab.crs,
    drop=True
)

# Punjab time series
weights = np.cos(np.deg2rad(da_punjab.lat))
punjab_ts = da_punjab.weighted(weights).mean(["lat", "lon"])
```

### 3.4 Clip Multiple Districts (Loop)

```python
districts = gpd.read_file("pakistan_districts.shp")
district_results = {}

for district_name in districts["DISTRICT"].unique():
    district = districts[districts["DISTRICT"] == district_name]
    try:
        clipped = da.rio.clip(district.geometry, district.crs, drop=True)
        district_mean = float(clipped.mean())
        district_results[district_name] = district_mean
    except Exception:
        district_results[district_name] = np.nan

# Add to GeoDataFrame
districts["mean_temp"] = districts["DISTRICT"].map(district_results)
```

---

## 📁 RIOXARRAY — LEVEL 4: Resampling

### 4.1 Spatial Resampling (Regridding)

```python
# Resample from 0.25° ERA5 to 0.5° resolution
da_025 = da   # 0.25° ERA5
new_lat = np.arange(-90, 90.5, 0.5)
new_lon = np.arange(-180, 180.5, 0.5)

# Method 1: xarray interpolation (nearest/linear)
da_05 = da_025.interp(lat=new_lat, lon=new_lon, method="linear")

# Method 2: rioxarray reproject to specific resolution
da_reprojected = da_025.rio.reproject(
    "EPSG:4326",
    shape=(len(new_lat), len(new_lon)),
    resampling=rioxarray.enums.Resampling.bilinear
)

# Resampling methods:
# nearest      = Nearest neighbor (for categorical data)
# bilinear     = Linear interpolation (for continuous fields)
# cubic        = Cubic (smoother, good for smooth fields)
# average      = Average of source pixels (good for downsampling)
# sum          = Sum of source pixels (for precipitation totals!)
# max/min      = Max/Min of source pixels (for extremes)
```

### 4.2 Match Grid to Reference

```python
# Reproject/resample data to exactly match another dataset's grid
# (useful for bias correction, data fusion)

reference_da = xr.open_dataarray("reference_grid.nc")
reference_da = reference_da.rio.set_spatial_dims(x_dim="lon", y_dim="lat")
reference_da = reference_da.rio.write_crs("EPSG:4326")

# Reproject to match reference
da_matched = da.rio.reproject_match(
    reference_da,
    resampling=rioxarray.enums.Resampling.bilinear
)

print(f"Reference shape: {reference_da.shape}")
print(f"Matched shape: {da_matched.shape}")  # Should match!
```

---

## 📁 RIOXARRAY — LEVEL 5: GeoTIFF Operations

### 5.1 Reading GeoTIFF (DEM, Satellite Data)

```python
import rioxarray

# Open DEM (Digital Elevation Model) — SRTM, ASTER, etc.
dem = rioxarray.open_rasterio("pakistan_dem.tif", masked=True)
print(dem)
print(dem.rio.crs)          # Often EPSG:32642 or 4326
print(dem.rio.nodata)       # NoData value (e.g., -9999)

# Single-band GeoTIFF
dem_2d = dem.squeeze("band", drop=True)  # Remove band dimension

# Multi-band (satellite): Landsat, MODIS
ndvi = rioxarray.open_rasterio("NDVI.tif", masked=True)
print(ndvi.shape)    # (bands, height, width)
ndvi_band1 = ndvi.sel(band=1)
```

### 5.2 Writing GeoTIFF

```python
# Save xarray DataArray as GeoTIFF
da = xr.open_dataarray("ERA5_mean_temp.nc")
da = da.rio.set_spatial_dims(x_dim="lon", y_dim="lat")
da = da.rio.write_crs("EPSG:4326")

# Write single time step to GeoTIFF
da.isel(time=0).rio.to_raster("ERA5_jan2020.tif")

# Write with compression
da.isel(time=0).rio.to_raster(
    "ERA5_jan2020_compressed.tif",
    driver="GTiff",
    compress="LZW",          # Lossless compression
    tiled=True,              # Cloud-optimized
    blockxsize=256,
    blockysize=256,
)
```

---

## 📁 RIOXARRAY — LEVEL 6: NetCDF → GeoTIFF Conversion

### 6.1 Complete Conversion Workflow

```python
import xarray as xr
import rioxarray
import numpy as np
from pathlib import Path

def netcdf_to_geotiff(nc_file, variable, output_dir, time_steps=None):
    """
    Convert NetCDF file to GeoTIFF files (one per time step).

    Args:
        nc_file: Path to NetCDF file
        variable: Variable name in NetCDF
        output_dir: Directory to save GeoTIFFs
        time_steps: List of time indices (None = all)
    """
    Path(output_dir).mkdir(exist_ok=True)

    # Load data
    da = xr.open_dataarray(nc_file) if variable is None else \
         xr.open_dataset(nc_file)[variable]

    # Setup rioxarray
    da = da.rio.set_spatial_dims(x_dim="lon", y_dim="lat")
    da = da.rio.write_nodata(np.nan)
    da = da.rio.write_crs("EPSG:4326")

    # Determine time steps to process
    if time_steps is None:
        time_steps = range(len(da.time))

    for t in time_steps:
        da_t = da.isel(time=t)
        date_str = str(da_t.time.values)[:10]    # YYYY-MM-DD
        out_path = f"{output_dir}/{variable}_{date_str}.tif"
        da_t.rio.to_raster(out_path, driver="GTiff", compress="LZW")
        print(f"Saved: {out_path}")

# Usage
netcdf_to_geotiff(
    nc_file="ERA5_t2m_2020.nc",
    variable="t2m",
    output_dir="output_geotiffs/",
    time_steps=range(12)    # First 12 months
)
```

### 6.2 Annual NetCDF from Monthly GeoTIFFs

```python
import rioxarray
import glob
import xarray as xr
import numpy as np

# Read multiple GeoTIFFs back as time series
tif_files = sorted(glob.glob("output_geotiffs/t2m_*.tif"))

arrays = []
for f in tif_files:
    da = rioxarray.open_rasterio(f, masked=True)
    arrays.append(da.squeeze("band", drop=True))

# Stack along time dimension
da_ts = xr.concat(arrays, dim="time")
da_ts["time"] = [np.datetime64(f.split("_")[-1].replace(".tif","")) for f in tif_files]

# Annual mean
annual_mean = da_ts.mean("time")
annual_mean.rio.to_raster("annual_mean.tif")
```

---

## 📁 SALEM — LEVEL 7: Basics

### 7.1 Installation & Dataset Access

```bash
pip install salem
```

```python
import salem

# Salem comes with built-in geographic datasets
# Download shapefiles for visualization
shdf = salem.read_shapefile(salem.get_demo_file("world_borders.shp"))

# Open a raster with salem (alternative to rioxarray)
grid_dem = salem.open_xr_dataset("dem.nc")

# Salem grid object — defines a geographic grid
from salem import Grid
g = Grid(
    nxny=(360, 180),       # (nx, ny) grid size
    dxdy=(1., 1.),          # Grid resolution in degrees
    ll_corner=(-180, -90), # Lower-left corner
    proj=salem.wgs84       # Projection
)
```

### 7.2 WRF Output Handling

```python
import salem
import xarray as xr

# Open WRF output (automatic WRF coordinate handling)
ds = salem.open_wrf_dataset("wrfout_d01_2020-07-01_00:00:00")

# Access variables (automatically converts to lat/lon coordinates)
t2 = ds.T2 - 273.15      # Near-surface temperature
rain = ds.RAINNC          # Accumulated rainfall

# Get lat/lon arrays (unstaggered automatically!)
lat = ds.lat.values
lon = ds.lon.values

# WRF-specific functions
u10 = ds.U10              # 10m U-wind
v10 = ds.V10              # 10m V-wind

# Convert to salem map projection
smap = ds.salem.get_map(crs=salem.googlemaps)
smap.set_data(t2)

# De-accumulate rainfall
rain_daily = rain.diff("time")    # Daily rain from accumulated
rain_daily = rain_daily.clip(min=0)  # Remove negative values
```

---

## 📋 rioxarray Quick Reference

| Operation | Code |
|-----------|------|
| Set spatial dims | `da.rio.set_spatial_dims(x_dim="lon", y_dim="lat")` |
| Write CRS | `da.rio.write_crs("EPSG:4326")` |
| Get CRS | `da.rio.crs` |
| Clip to bbox | `da.rio.clip_box(minx, miny, maxx, maxy)` |
| Clip to shapefile | `da.rio.clip(gdf.geometry, gdf.crs)` |
| Reproject | `da.rio.reproject("EPSG:32642")` |
| Match grid | `da.rio.reproject_match(reference_da)` |
| Save GeoTIFF | `da.rio.to_raster("output.tif")` |
| Open GeoTIFF | `rioxarray.open_rasterio("file.tif", masked=True)` |
| Get bounds | `da.rio.bounds()` |
| Get transform | `da.rio.transform()` |

---

## 📚 Resources

- [rioxarray Documentation](https://corteva.github.io/rioxarray)
- [rasterio Documentation](https://rasterio.readthedocs.io)
- [pyproj Documentation](https://pyproj4.github.io/pyproj)
- [salem Documentation](https://salem.readthedocs.io)
- [EPSG Registry](https://epsg.io) — Find CRS codes for any region
