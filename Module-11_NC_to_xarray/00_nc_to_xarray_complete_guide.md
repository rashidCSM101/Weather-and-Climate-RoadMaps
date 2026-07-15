# 🔄 NetCDF → xarray: Loading & Operations Complete Guide
## From Raw .nc Files to Full Climate Analysis Workflows

---

## 📚 Overview

This guide covers **everything** about loading `.nc` (NetCDF) files into xarray and applying operations. It bridges the NetCDF commands guide with the full xarray roadmap.

```
Step 1: Understand the NC file structure
Step 2: Open the file in xarray
Step 3: Inspect, fix issues
Step 4: Select & subset
Step 5: Apply operations
Step 6: Save results
```

---

## 📁 PART 1 — Understanding a NetCDF File Before Loading

### Before you open in Python — inspect at command line:

```bash
# Step 0: What is inside this file?
ncdump -h ERA5_t2m_2020.nc
```

**Output to understand:**
```
netcdf ERA5_t2m_2020 {
dimensions:
    longitude = 1440 ;    ← 0.25° resolution: 360/0.25 = 1440 lon points
    latitude = 721 ;      ← 360/0.5 + 1 = 721 lat points
    time = 12 ;           ← 12 monthly time steps

variables:
    float longitude(longitude) ;
        longitude:units = "degrees_east" ;    ← Unit of longitude coord
    float latitude(latitude) ;
        latitude:units = "degrees_north" ;
    int time(time) ;
        time:units = "hours since 1900-01-01 00:00:00.0" ;  ← IMPORTANT!
        time:calendar = "gregorian" ;
    short t2m(time, latitude, longitude) ;    ← stored as int16 (short)
        t2m:scale_factor = 0.00056 ;          ← IMPORTANT: must multiply data by this
        t2m:add_offset = 266.7 ;              ← IMPORTANT: must add this
        t2m:_FillValue = -32767 ;             ← Missing value code
        t2m:units = "K" ;
        t2m:long_name = "2 metre temperature" ;
}
```

**What this tells you:**
- `t2m` is stored as **short** (int16) to save space
- xarray will AUTO-APPLY `scale_factor × data + add_offset` to get real values
- xarray will AUTO-CONVERT `_FillValue = -32767` to `NaN`
- xarray will AUTO-DECODE time from "hours since 1900-01-01" to real datetime
- You get the real float values automatically — no manual conversion needed

---

## 📁 PART 2 — Opening NetCDF Files in xarray

### 2.1 Single File (Basic)

```python
import xarray as xr
import numpy as np

# --- SIMPLEST CASE: Open single file ---
ds = xr.open_dataset("ERA5_t2m_2020.nc")

# What you get:
print(ds)
# <xarray.Dataset>
# Dimensions:    (longitude: 1440, latitude: 721, time: 12)
# Coordinates:
#   * longitude  (longitude) float32 -180.0 -179.75 ... 179.75
#   * latitude   (latitude)  float32 90.0 89.75 ... -90.0
#   * time       (time) datetime64[ns] 2020-01-01 ... 2020-12-01  ← auto-decoded!
# Data variables:
#     t2m        (time, latitude, longitude) float32 ...           ← auto-scaled!
# Attributes:
#     Conventions: CF-1.6
```

### 2.2 Common Opening Options

```python
# --- Full control over how file is opened ---
ds = xr.open_dataset(
    "ERA5_t2m_2020.nc",

    # --- Time decoding ---
    decode_times=True,        # True (default): auto-decode time axis
    # decode_times=False,     # Use this if time decoding fails

    # --- Data scaling ---
    mask_and_scale=True,      # True (default): apply scale_factor, add_offset, _FillValue
    # mask_and_scale=False,   # Use this to get raw int16 values

    # --- Backend engine ---
    engine="netcdf4",         # "netcdf4" (default), "scipy", "cfgrib", "zarr"

    # --- Chunking (enables Dask lazy loading) ---
    chunks={"time": 100},     # Load in chunks of 100 time steps
    # chunks="auto",          # Let xarray decide chunk sizes

    # --- Dimension renaming ---
    # (use after loading if dimension names are weird)

    # --- Drop unnecessary variables ---
    drop_variables=["expver"],  # ERA5 sometimes has this annoying variable
)

print(ds)
```

### 2.3 Multiple Files (Open Many .nc Files at Once)

```python
# --- Open all files matching a pattern ---
# MOST COMMON: ERA5 files split by year
ds = xr.open_mfdataset(
    "ERA5_t2m_*.nc",           # Glob pattern: all files matching this
    combine="by_coords",        # Join files based on their coordinate values
    # combine="nested",         # For files that don't overlap in time/space
    chunks={"time": 100},       # ALWAYS use this for multiple files → lazy loading

    # Join method
    join="outer",               # Keep all time steps, fill gaps with NaN
    # join="inner",             # Keep only overlapping time steps

    # Preprocessing (apply function to each file before concatenation)
    preprocess=lambda ds: ds[["t2m"]],   # Keep only t2m variable from each file
)

print(ds)
print(f"Total time steps: {len(ds.time)}")

# --- Open files from a list ---
import glob
files = sorted(glob.glob("ERA5/ERA5_t2m_19*.nc"))   # All 1990s files
ds = xr.open_mfdataset(files, combine="by_coords", chunks={"time": 100})
```

### 2.4 Handling Common Problems

#### Problem 1: Time axis fails to decode

```python
# Error: "ValueError: unable to decode time units"
# Solution: Open with decode_times=False, then fix manually

ds = xr.open_dataset("ERA5.nc", decode_times=False)
print(ds.time)   # Shows raw values and units

# Fix time manually
import pandas as pd
time_var = ds.time
time_vals = pd.to_datetime(time_var.values, unit="h",
                            origin=pd.Timestamp("1900-01-01"))
ds["time"] = time_vals
```

#### Problem 2: Latitude is upside down (90 to -90)

```python
# ERA5 and many reanalyses have lat going from 90 → -90
# This is FINE for xarray — sel() still works correctly
# But if needed, flip:
ds = ds.sortby("lat")   # Will reorder lat to -90 → 90
```

#### Problem 3: Longitude is 0-360 instead of -180 to 180

```python
# Some models output 0-360 longitude
# Fix:
ds = ds.assign_coords(lon=(ds.lon + 180) % 360 - 180)  # Convert 0-360 → -180-180
ds = ds.sortby("lon")   # Sort after conversion
```

#### Problem 4: Variable has wrong units attribute

```python
# Fix attributes manually after loading
ds["t2m"].attrs["units"] = "K"
ds["pr"].attrs["units"] = "mm d-1"
```

#### Problem 5: Multiple time axes (ERA5 specific "expver" issue)

```python
# ERA5 sometimes has duplicate records from different versions
ds = xr.open_dataset("ERA5.nc", drop_variables="expver")
# Or
ds = xr.open_mfdataset("ERA5_*.nc", combine="by_coords",
                        compat="override")
```

---

## 📁 PART 3 — Inspection: Understanding Your Data

### 3.1 Essential Inspection Commands

```python
# Basic overview
print(ds)              # Full dataset description
print(ds.dims)         # {'time': 12, 'latitude': 721, 'longitude': 1440}
print(ds.coords)       # All coordinate variables
print(ds.data_vars)    # All data variables
print(ds.attrs)        # Global metadata (history, source, etc.)

# Variable-specific inspection
da = ds["t2m"]         # Select specific variable as DataArray
print(da)
print(da.dims)         # ('time', 'latitude', 'longitude')
print(da.shape)        # (12, 721, 1440)
print(da.dtype)        # float32 (after scaling)
print(da.attrs)        # units, long_name, etc.
print(da.coords)       # Coordinate arrays

# Coordinate values
print(ds.latitude.values[:5])    # [90.0, 89.75, 89.5, 89.25, 89.0]
print(ds.longitude.values[:5])   # [-180.0, -179.75, ...]
print(ds.time.values[:5])        # ['2020-01-01', ...]

# Data range
print(f"Min: {float(da.min()):.2f}")
print(f"Max: {float(da.max()):.2f}")
print(f"Mean: {float(da.mean()):.2f}")
print(f"NaN count: {int(da.isnull().sum())}")

# Check for NaN
print(da.isnull().any().values)  # True if any NaN exists
```

### 3.2 Renaming for Convenience

```python
# ERA5 often has "longitude"/"latitude" instead of "lon"/"lat"
# Rename for shorter, standard names:
ds = ds.rename({"longitude": "lon", "latitude": "lat"})
ds = ds.rename({"t2m": "temperature"})  # Optional: rename variable too

# After renaming, use the new names
da = ds["temperature"]
da.sel(lat=33.73, lon=73.09, method="nearest")
```

---

## 📁 PART 4 — Selection & Subsetting

### 4.1 Time Selection

```python
da = ds["t2m"]

# --- By time label ---
da_jan2020 = da.sel(time="2020-01")         # January 2020 only
da_2020 = da.sel(time="2020")               # Entire year 2020
da_jja = da.sel(time=da.time.dt.month.isin([6, 7, 8]))   # JJA all years

# --- By time range ---
da_1990_2020 = da.sel(time=slice("1990-01-01", "2020-12-31"))

# --- By index position ---
da_first = da.isel(time=0)                  # First time step
da_last = da.isel(time=-1)                  # Last time step
da_first_10 = da.isel(time=slice(0, 10))    # First 10 time steps

# --- By date attributes ---
da_june = da.where(da.time.dt.month == 6, drop=True)   # June only
da_summer = da.where(da.time.dt.season == "JJA", drop=True)
```

### 4.2 Spatial Selection

```python
# --- Single point (nearest grid cell) ---
islamabad = da.sel(lat=33.73, lon=73.09, method="nearest")
print(islamabad)   # Time series at Islamabad
print(islamabad.values)

# --- Spatial bounding box ---
pakistan = da.sel(
    lat=slice(37, 23),    # ERA5: lat goes 90→-90, so slice is N→S
    lon=slice(60, 80)
)
# For regular N→S latitude:
# pakistan = da.sel(lat=slice(23, 37), lon=slice(60, 80))

# --- Multiple points ---
cities = {"Islamabad": (33.73, 73.09),
          "Karachi": (24.86, 67.01),
          "Lahore": (31.52, 74.35)}

for city, (lat, lon) in cities.items():
    ts = da.sel(lat=lat, lon=lon, method="nearest")
    print(f"{city}: {float(ts.mean()):.2f} K")
```

### 4.3 Level/Pressure Selection (3D data)

```python
# For pressure-level data (time, level, lat, lon)
ds_pl = xr.open_dataset("ERA5_pressure_levels.nc")
u = ds_pl["u"]   # Zonal wind (time, pressure, lat, lon)

# Select 850 hPa level
u850 = u.sel(pressure_level=850)

# Select multiple levels
u_upper = u.sel(pressure_level=[850, 500, 200])

# Vertical profile at a point
u_profile = u.sel(lat=33.73, lon=73.09, method="nearest")
# Now shape: (time, pressure)
```

---

## 📁 PART 5 — Operations on NC Data

### 5.1 Unit Conversion

```python
# Temperature: Kelvin → Celsius
da_C = ds["t2m"] - 273.15
da_C.attrs["units"] = "°C"
da_C.attrs["long_name"] = "2 metre temperature"
da_C.name = "t2m"

# Precipitation: ERA5 is m/hour → mm/day
# ERA5 precip units: m (accumulated over 1 hour)
da_pr_mmday = ds["tp"] * 1000 * 24   # m → mm, hourly → daily
da_pr_mmday.attrs["units"] = "mm day-1"
```

### 5.2 Spatial Aggregation

```python
da = ds["t2m"] - 273.15   # In Celsius

# Simple spatial mean (NOT area-weighted)
global_ts = da.mean(["lat", "lon"])   # Time series, simple mean

# CORRECT: Latitude-weighted spatial mean (area-weighted)
weights = np.cos(np.deg2rad(da.lat))
weights.name = "weights"
global_ts_weighted = da.weighted(weights).mean(["lat", "lon"])

# Regional mean (Pakistan, 23–37°N, 60–80°E)
pak_ts_weighted = da.sel(lat=slice(37, 23), lon=slice(60, 80)) \
                    .weighted(weights) \
                    .mean(["lat", "lon"])

# Zonal mean (average over all longitudes)
zonal_mean = da.mean("lon")   # Shape: (time, lat)
```

### 5.3 Temporal Aggregation

```python
# Annual mean
annual = da.resample(time="1Y").mean()
print(annual.dims)   # ('year', 'lat', 'lon')

# Monthly mean from daily data
monthly = da.resample(time="1MS").mean()

# Seasonal mean
seasonal = da.resample(time="QS-DEC").mean()   # DJF, MAM, JJA, SON

# Long-term monthly climatology (e.g., 1981-2010 normal)
base = da.sel(time=slice("1981", "2010"))
clim = base.groupby("time.month").mean("time")
print(clim.dims)   # ('month', 'lat', 'lon') — 12 values

# Monthly anomalies
anomaly = da.groupby("time.month") - clim
print(anomaly.dims)   # ('time', 'lat', 'lon') — same as original
```

### 5.4 Statistical Operations

```python
# All-time statistics (spatial maps)
temp_mean = da.mean("time")           # Mean map
temp_std  = da.std("time")            # Variability map
temp_max  = da.max("time")            # All-time maximum
temp_min  = da.min("time")            # All-time minimum
temp_p95  = da.quantile(0.95, "time") # 95th percentile map

# Annual cycle
annual_cycle = da.groupby("time.month").mean("time")  # 12-month climatology

# Trend at each grid point (using apply_ufunc)
from scipy.stats import linregress

def get_slope(arr):
    x = np.arange(len(arr))
    valid = ~np.isnan(arr)
    if valid.sum() < 10:
        return np.nan
    slope, *_ = linregress(x[valid], arr[valid])
    return slope

slope_map = xr.apply_ufunc(
    get_slope,
    da,
    input_core_dims=[["time"]],
    vectorize=True,
    dask="parallelized",
    output_dtypes=[float],
)
```

### 5.5 Masking & Filtering

```python
# Mask values (set to NaN)
da_land = da.where(land_mask)       # NaN over ocean (land_mask=True over land)
da_hot = da.where(da > 303.15)      # Keep only values > 30°C
da_pak = da.where((da.lat >= 23) & (da.lat <= 37) &
                  (da.lon >= 60) & (da.lon <= 80))  # Box mask

# Clip values
da_clipped = da.clip(min=200, max=330)   # Physical bounds

# Drop NaN-only slices
da_clean = da.dropna("lat", how="all")   # Drop lat rows that are all NaN
```

### 5.6 Combining Datasets

```python
# Merge two datasets (different variables)
merged = xr.merge([ds_temperature, ds_precipitation])

# Concatenate along time
combined = xr.concat([ds_2019, ds_2020, ds_2021], dim="time")

# Join datasets that are on different grids
# (need to regrid first — see Module-9_salem_rioxarray)
```

---

## 📁 PART 6 — Saving Results

### 6.1 Save to NetCDF

```python
# Simple save
result.to_netcdf("output.nc")

# With encoding (compression, precision)
result.to_netcdf(
    "output.nc",
    encoding={
        "t2m": {
            "dtype": "float32",      # Save as 32-bit float (halves size vs float64)
            "zlib": True,            # Enable compression
            "complevel": 4,          # Compression level (1=fast, 9=small)
            "_FillValue": -9999.0,   # Missing value code
        },
        "time": {"dtype": "float64"},
    }
)

# Save Dataset with multiple variables
ds_result = xr.Dataset({
    "slope": slope_map,
    "pvalue": pval_map,
    "annual_mean": annual_mean,
})
ds_result.attrs = {
    "title": "ERA5 Trend Analysis 1979-2023",
    "institution": "Quaid-i-Azam University",
    "author": "Rashid",
    "history": "Created 2024-01-01",
    "Conventions": "CF-1.8",
}
ds_result.to_netcdf("trend_analysis.nc")
```

### 6.2 Save to CSV (for station-like data)

```python
# Time series → CSV
pak_ts.to_series().to_csv("pakistan_time_series.csv",
                           header=["temperature_C"],
                           index=True)

# Annual mean per grid point → CSV (for small domains)
annual_mean.to_dataframe().reset_index().to_csv("annual_mean_grid.csv", index=False)
```

### 6.3 Save to GeoTIFF (for GIS)

```python
import rioxarray

result = result.rio.set_spatial_dims(x_dim="lon", y_dim="lat")
result = result.rio.write_crs("EPSG:4326")
result.rio.to_raster("output_trend_map.tif", driver="GTiff",
                     compress="LZW")
```

---

## 📁 PART 7 — Complete End-to-End Workflow

```python
"""
Complete workflow: Load ERA5 NC file → clean → analyze → save
Example: Annual temperature trend over Pakistan (1979-2023)
"""

import xarray as xr
import numpy as np
import pymannkendall as mk
from dask.diagnostics import ProgressBar

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# STEP 1: LOAD DATA
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
print("Loading ERA5 data...")
ds = xr.open_mfdataset(
    "ERA5_t2m_19*.nc ERA5_t2m_20*.nc".split(),
    combine="by_coords",
    chunks={"time": 100}
)

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# STEP 2: CLEAN & PREPARE
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# Rename dimensions
ds = ds.rename({"longitude": "lon", "latitude": "lat"})

# Select variable and convert units
da = ds["t2m"] - 273.15   # K → °C
da.attrs["units"] = "°C"
da.attrs["long_name"] = "2 metre temperature"

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# STEP 3: SUBSET TO PAKISTAN
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
da_pak = da.sel(lat=slice(37, 23), lon=slice(60, 80))
print(f"Pakistan domain: {da_pak.shape}")

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# STEP 4: COMPUTE CLIMATOLOGY & ANOMALY
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# Monthly climatology (1981-2010 base period)
base = da_pak.sel(time=slice("1981", "2010"))
clim = base.groupby("time.month").mean("time")

# Monthly anomalies
anomaly = da_pak.groupby("time.month") - clim

# Annual mean anomaly
annual_anomaly = anomaly.resample(time="1Y").mean()

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# STEP 5: TREND ANALYSIS
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
def mk_analysis(arr):
    valid = arr[~np.isnan(arr)]
    if len(valid) < 10:
        return np.nan, np.nan
    result = mk.hamed_rao_modification_test(valid)
    return result.slope, result.p

def get_slope(arr): return mk_analysis(arr)[0]
def get_pval(arr): return mk_analysis(arr)[1]

kw = dict(input_core_dims=[["time"]], vectorize=True,
          dask="parallelized", output_dtypes=[float])

slope_map = xr.apply_ufunc(get_slope, annual_anomaly, **kw)
pval_map  = xr.apply_ufunc(get_pval, annual_anomaly, **kw)

print("Computing trend maps...")
with ProgressBar():
    slope_map = slope_map.compute()
    pval_map  = pval_map.compute()

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# STEP 6: AREA-WEIGHTED REGIONAL MEAN
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
weights = np.cos(np.deg2rad(da_pak.lat))
pak_ts = da_pak.weighted(weights).mean(["lat", "lon"])
pak_annual_ts = pak_ts.resample(time="1Y").mean()
with ProgressBar():
    pak_annual_ts = pak_annual_ts.compute()

# Pakistan-wide trend
pak_result = mk.hamed_rao_modification_test(pak_annual_ts.values)
print(f"\nPakistan-wide trend: {pak_result.slope:.3f} °C/year")
print(f"p-value: {pak_result.p:.4f}")
print(f"Significant: {pak_result.h}")

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# STEP 7: SAVE ALL RESULTS
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# Save gridded results
xr.Dataset({"slope": slope_map, "pvalue": pval_map}).to_netcdf(
    "pakistan_trend_1979_2023.nc",
    encoding={
        "slope": {"dtype": "float32", "zlib": True},
        "pvalue": {"dtype": "float32", "zlib": True},
    }
)

# Save time series to CSV
pak_annual_ts.to_series().to_csv("pakistan_annual_temperature.csv")

# Save climatology
clim.to_netcdf("pakistan_climatology_1981_2010.nc")

print("\nAll results saved!")
print("Files created:")
print("  pakistan_trend_1979_2023.nc    — Trend maps")
print("  pakistan_annual_temperature.csv — Time series")
print("  pakistan_climatology_1981_2010.nc — 30-year normals")
```

---

## 📋 NC → xarray Quick Reference

| Task | Code |
|------|------|
| Open single file | `xr.open_dataset("file.nc")` |
| Open multiple files | `xr.open_mfdataset("*.nc", combine="by_coords")` |
| Open with chunking | `xr.open_mfdataset("*.nc", chunks={"time": 100})` |
| View structure | `print(ds)` |
| Check variable | `print(ds["t2m"])` |
| K → °C | `da - 273.15` |
| Rename dims | `ds.rename({"longitude": "lon"})` |
| Select by coord | `da.sel(lat=33.73, lon=73.09, method="nearest")` |
| Select region | `da.sel(lat=slice(37,23), lon=slice(60,80))` |
| Select year | `da.sel(time="2020")` |
| Annual mean | `da.resample(time="1Y").mean()` |
| Climatology | `da.groupby("time.month").mean("time")` |
| Anomaly | `da.groupby("time.month") - clim` |
| Weighted mean | `da.weighted(weights).mean(["lat","lon"])` |
| Apply function | `xr.apply_ufunc(func, da, input_core_dims=[["time"]])` |
| Compute (Dask) | `da.compute()` |
| Save NC | `da.to_netcdf("output.nc", encoding={...})` |

---

## ⚠️ Common Pitfalls & Solutions

| Problem | Cause | Solution |
|---------|-------|---------|
| Time decoding fails | Non-standard calendar (360-day) | `decode_times=False`, then use `cftime` |
| Latitude reversed | ERA5 goes 90→-90 | Use `slice(37, 23)` for N→S selection |
| Lon is 0-360 | Some models use 0-360 | `ds.assign_coords(lon=(ds.lon+180)%360-180)` |
| Out of memory | File too large | Add `chunks={"time": 100}` |
| `isel` vs `sel` | Confusion | `isel` = integer index, `sel` = coordinate value |
| Slow computation | No Dask | Always use `chunks=` for multi-file datasets |
| Missing `_FillValue` | Not auto-applied | Check `mask_and_scale=True` (default) |

---

## 📚 Resources

- [xarray Documentation](https://docs.xarray.dev)
- [xarray NC4 Guide](https://docs.xarray.dev/en/stable/user-guide/io.html)
- [Pangeo Tutorial](https://tutorial.xarray.dev)
- [CF Conventions](https://cfconventions.org)
