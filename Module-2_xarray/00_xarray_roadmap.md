# 🌐 xarray — Complete Learning Roadmap
## Climate & Geoscience Data Analysis

---

## 📚 What is xarray?

xarray is a Python library designed for working with **labeled, multi-dimensional arrays** — the standard format for climate, weather, and earth science data. It builds on top of NumPy and Pandas, adding dimension names, coordinate labels, and metadata awareness.

**Why xarray for climate science?**
- Handles NetCDF (.nc) files natively — the universal format for climate data
- Keeps track of lat, lon, time coordinates automatically
- Supports lazy loading (Dask integration) for files too large to fit in memory
- Works with CF (Climate and Forecast) conventions out of the box

---

## 🗺️ Roadmap Overview

```
Level 1: Foundations          → Data structures, loading NC files, indexing
Level 2: Core Operations      → Selection, arithmetic, resampling
Level 3: GroupBy & Rolling    → Climatologies, anomalies, moving averages
Level 4: apply_ufunc          → Custom functions on xarray objects
Level 5: Dask Integration     → Large file handling (multi-GB datasets)
Level 6: Advanced Workflows   → Real climate analysis pipelines
```

---

## 📁 LEVEL 1 — Foundations

### 1.1 Core Data Structures

| Object | Description | Climate Analogy |
|--------|-------------|-----------------|
| `xr.DataArray` | N-dimensional labeled array | Single variable: temperature |
| `xr.Dataset` | Dict-like container of DataArrays | A full NC file (T, P, U, V, ...) |
| `dims` | Dimension names (time, lat, lon) | Axis labels |
| `coords` | Coordinate values | Actual lat/lon/time values |
| `attrs` | Metadata dictionary | Units, source, history |

```python
import xarray as xr
import numpy as np

# --- Create a DataArray manually ---
temp = xr.DataArray(
    data=np.random.rand(12, 90, 180),
    dims=["time", "lat", "lon"],
    coords={
        "time": xr.cftime_range("2000-01", periods=12, freq="MS"),
        "lat": np.linspace(-89, 89, 90),
        "lon": np.linspace(-179, 179, 180),
    },
    attrs={"units": "K", "long_name": "Surface Air Temperature"},
    name="temperature"
)

print(temp)
print(temp.dims)       # ('time', 'lat', 'lon')
print(temp.shape)      # (12, 90, 180)
print(temp.coords)     # Shows coordinate arrays
print(temp.attrs)      # {'units': 'K', ...}
```

### 1.2 Loading NetCDF Files

```python
# --- Single file ---
ds = xr.open_dataset("ERA5_temperature_2020.nc")
print(ds)                      # Overview of all variables
print(ds.data_vars)            # List of variables
print(ds["t2m"])               # Access variable as DataArray
print(ds.t2m)                  # Same, dot notation

# --- Multiple files (temporal concatenation) ---
ds = xr.open_mfdataset(
    "ERA5_temperature_20*.nc",
    combine="by_coords",       # Automatically joins by coordinates
    chunks={"time": 100}       # Enable Dask chunking
)

# --- Control decode behavior ---
ds = xr.open_dataset(
    "data.nc",
    decode_times=True,         # Decode CF time units
    mask_and_scale=True,       # Apply scale_factor, add_offset, _FillValue
    engine="netcdf4"           # or "scipy", "cfgrib", "zarr"
)
```

### 1.3 Inspecting Datasets

```python
# Basic inspection
print(ds)                      # Full dataset summary
print(ds.dims)                 # Dimension sizes
print(ds.coords)               # Coordinates
print(ds.data_vars)            # Data variables
print(ds.attrs)                # Global attributes

# Specific variable
var = ds["precipitation"]
print(var.dims)
print(var.dtype)
print(var.attrs)               # units, long_name, missing_value
print(var.min().values)        # Actual minimum value
print(var.max().values)
```

### 1.4 Saving to NetCDF

```python
# Save DataArray
da.to_netcdf("output_temperature.nc")

# Save Dataset
ds.to_netcdf("output_dataset.nc", encoding={
    "temperature": {"dtype": "float32", "zlib": True, "complevel": 4}
})

# Save with specific unlimited dimension
ds.to_netcdf("output.nc", unlimited_dims=["time"])
```

---

## 📁 LEVEL 2 — Core Operations

### 2.1 Indexing & Selection

```python
# --- Label-based selection (isel = integer, sel = label) ---
# Select by position (like NumPy)
da.isel(time=0)                      # First time step
da.isel(lat=slice(10, 50))           # Lat rows 10–49
da.isel(time=0, lat=5, lon=10)       # Scalar selection

# Select by coordinate value (like Pandas)
da.sel(time="2020-01")               # First month of 2020
da.sel(lat=30.5, lon=70.2, method="nearest")   # Nearest grid point
da.sel(lat=slice(20, 40), lon=slice(60, 80))   # Bounding box

# --- Advanced: where() ---
da_filtered = da.where(da > 0)       # Mask values ≤ 0 (NaN)
da_positive = da.where(da > 280, other=280)   # Clip below 280

# --- Drop coordinates ---
da_clean = da.drop_vars("level")
```

### 2.2 Arithmetic & Broadcasting

```python
# Element-wise arithmetic (auto-aligns on coordinates)
temp_celsius = ds["t2m"] - 273.15
anomaly = ds["t2m"] - ds["t2m"].mean("time")

# Operations with broadcasting
lat_weights = np.cos(np.deg2rad(ds.lat))
weighted_mean = (ds["t2m"] * lat_weights).mean(["lat", "lon"])

# Dataset arithmetic (applies to all variables)
ds_celsius = ds - 273.15

# Combining datasets
merged = xr.merge([ds1, ds2])
concat = xr.concat([ds_2019, ds_2020], dim="time")
```

### 2.3 Reducing / Aggregation

```python
# Reduce along dimensions
mean_temp = da.mean("time")             # Spatial mean map
zonal_mean = da.mean("lon")            # Zonal mean (lat profile)
time_series = da.mean(["lat", "lon"])  # Area-averaged time series

# Other reductions
da.sum("time")
da.std("time")
da.var("time")
da.min("time")
da.max("time")
da.median("time")
da.quantile(0.95, dim="time")          # 95th percentile

# Keep dimensions
da.mean("time", keepdims=True)
```

### 2.4 Resampling (Time Operations)

```python
# --- Resample: change temporal frequency ---
# Monthly → Annual
annual = da.resample(time="1Y").mean()

# Daily → Monthly
monthly = da.resample(time="1MS").mean()   # "MS" = Month Start
monthly_sum = da.resample(time="1MS").sum()  # for precipitation

# Monthly → Seasonal (3-month means)
seasonal = da.resample(time="QS-DEC").mean()

# Hourly → Daily
daily = da.resample(time="1D").mean()

# --- Interpolate: upsample ---
daily_interp = monthly.resample(time="1D").interpolate("linear")
```

---

## 📁 LEVEL 3 — GroupBy & Rolling Windows

### 3.1 GroupBy — Seasonal & Monthly Climatologies

```python
# --- Monthly climatology (30-year normal) ---
climatology = da.groupby("time.month").mean("time")
# Result: DataArray with 12 values (Jan–Dec average)
print(climatology)  # dims: (month, lat, lon)

# --- Seasonal climatology ---
seasonal_clim = da.groupby("time.season").mean("time")
# Seasons: DJF, MAM, JJA, SON

# --- Annual cycle ---
annual_cycle = da.groupby("time.dayofyear").mean("time")

# --- GroupBy year ---
yearly = da.groupby("time.year").mean()

# --- GroupBy custom bins ---
# Example: group by temperature quintiles
bins = np.percentile(da.values, [0, 20, 40, 60, 80, 100])
da.groupby_bins("lat", bins=5).mean()
```

### 3.2 Anomalies (Deseasonalization)

```python
# Method 1: GroupBy subtract (standard approach)
climatology = da.groupby("time.month").mean("time")
anomaly = da.groupby("time.month") - climatology
# Each Jan minus Jan climatology, each Feb minus Feb climatology, etc.

# Method 2: Seasonal anomaly
seasonal_clim = da.groupby("time.season").mean("time")
seasonal_anomaly = da.groupby("time.season") - seasonal_clim

# Method 3: Annual anomaly
annual_mean = da.mean("time")
annual_anomaly = da - annual_mean

# Standardized anomaly (z-score)
std_anomaly = anomaly / da.groupby("time.month").std("time")
```

### 3.3 Rolling Windows (Moving Averages)

```python
# --- Simple rolling mean ---
da_rolling = da.rolling(time=5, center=True).mean()
# 5-time-step centered moving average

# --- Rolling with minimum periods (handles NaN at edges) ---
da_rolling = da.rolling(time=12, min_periods=6).mean()

# --- Rolling std (for detecting extreme events) ---
da_rolling_std = da.rolling(time=30).std()

# --- Multi-dimensional rolling ---
da_smooth = da.rolling(lat=3, lon=3, center=True).mean()

# --- Rolling sum (e.g., cumulative rainfall) ---
rainfall_cumsum = precip.rolling(time=30).sum()

# --- Construct rolling object for custom operations ---
roller = da.rolling(time=5, center=True)
roller.mean()       # Mean
roller.sum()        # Sum
roller.std()        # Standard deviation
roller.min()        # Minimum
roller.max()        # Maximum
roller.construct("window_dim")  # Get raw windowed DataArray

# Example: Calculate rolling Pearson correlation
def pearson_corr(x, y):
    # custom correlation over window
    pass

rolling = da.rolling(time=90)
# Apply custom function via construct + reduce
windows = rolling.construct("w")  # (time, lat, lon, w)
```

### 3.4 GroupBy + Rolling Combined

```python
# Example: 5-year rolling mean of annual temperature
annual = da.resample(time="1Y").mean()
rolling_annual = annual.rolling(time=5, center=True).mean()

# Example: Monthly anomaly smoothed with 3-month rolling average
clim = da.groupby("time.month").mean("time")
anomaly = da.groupby("time.month") - clim
smooth_anomaly = anomaly.rolling(time=3, center=True).mean()
```

---

## 📁 LEVEL 4 — apply_ufunc

### 4.1 What is apply_ufunc?

`xr.apply_ufunc` allows you to apply **arbitrary NumPy/SciPy functions** to xarray objects while preserving coordinate labels. It's the bridge between xarray and the scientific Python ecosystem.

```
When to use apply_ufunc:
  ✅ Applying scipy functions (stats, signal processing)
  ✅ Custom vectorized operations not available in xarray
  ✅ Functions that work on 1D arrays (applied along a dim)
  ✅ Wrapping functions from other libraries (e.g., pymannkendall)
  ❌ Simple element-wise math → use arithmetic directly
  ❌ Reductions along dims → use .mean(), .std(), etc.
```

### 4.2 Basic apply_ufunc

```python
import scipy.stats as stats

# --- Example 1: Apply zscore along time ---
def zscore(arr):
    return (arr - arr.mean()) / arr.std()

normalized = xr.apply_ufunc(
    zscore,                          # Function to apply
    da,                              # Input DataArray
    vectorize=True,                  # Apply function element-wise
    input_core_dims=[["time"]],      # Dims consumed by the function
    output_core_dims=[["time"]],     # Dims produced by the function
    exclude_dims=set(["time"]),      # Dims that change size
)

# --- Example 2: Linear detrending along time ---
from scipy import signal

detrended = xr.apply_ufunc(
    signal.detrend,
    da,
    input_core_dims=[["time"]],
    output_core_dims=[["time"]],
    vectorize=True,
    kwargs={"axis": -1},
)
```

### 4.3 apply_ufunc with Multiple Outputs

```python
# --- Mann-Kendall trend test for every grid point ---
import pymannkendall as mk

def mk_trend(arr):
    result = mk.original_test(arr)
    return result.slope, result.p  # Return 2 values

slopes, pvalues = xr.apply_ufunc(
    mk_trend,
    da,
    input_core_dims=[["time"]],
    output_core_dims=[[], []],       # Both outputs are scalars per grid point
    vectorize=True,
    dask="parallelized",
    output_dtypes=[float, float],
)

# slopes and pvalues are now (lat, lon) DataArrays
```

### 4.4 apply_ufunc with Dask

```python
# With chunked (Dask) arrays
slopes = xr.apply_ufunc(
    mk_trend,
    da.chunk({"lat": 10, "lon": 10}),   # Pre-chunk
    input_core_dims=[["time"]],
    output_core_dims=[[]],
    vectorize=True,
    dask="parallelized",                 # Parallelize over chunks
    output_dtypes=[float],
)

# Trigger computation
slopes = slopes.compute()
```

### 4.5 Practical apply_ufunc Recipes

```python
# --- Percentile calculation ---
p95 = xr.apply_ufunc(
    np.percentile,
    da,
    input_core_dims=[["time"]],
    kwargs={"q": 95, "axis": -1},
    dask="parallelized",
    output_dtypes=[float],
)

# --- Linear trend (slope) at each grid point ---
from scipy.stats import linregress

def get_slope(y):
    x = np.arange(len(y))
    slope, *_ = linregress(x, y)
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

---

## 📁 LEVEL 5 — Dask Integration

### 5.1 Why Dask?

| Without Dask | With Dask |
|-------------|-----------|
| Loads entire file into RAM | Loads only what's needed |
| Crashes on 50GB ERA5 files | Handles 500GB+ datasets |
| Sequential processing | Parallel processing |
| Immediate computation | Lazy evaluation |

### 5.2 Chunking Strategy

```python
# --- Open with chunking (creates Dask arrays) ---
ds = xr.open_mfdataset(
    "ERA5_*.nc",
    chunks={
        "time": 100,       # Process 100 time steps at once
        "lat": 90,         # Half of global lat
        "lon": 180,        # Half of global lon
    }
)

# Check chunk sizes
print(ds["t2m"].chunks)    # ((100, 100, 65), (90, 90), (180, 180))

# Rechunk if needed
da_rechunked = da.chunk({"time": 365, "lat": -1, "lon": -1})
# -1 means "don't chunk this dimension" (entire axis in one chunk)
```

### 5.3 Dask Workflows

```python
import dask
from dask.distributed import Client

# Start a local Dask cluster
client = Client(n_workers=4, threads_per_worker=2, memory_limit="4GB")
print(client.dashboard_link)   # Browser dashboard

# --- Lazy computation (nothing runs yet) ---
ds = xr.open_mfdataset("ERA5_*.nc", chunks={"time": 100})
annual_mean = ds["t2m"].groupby("time.year").mean()  # Lazy!
print(annual_mean)  # Shows graph, not result

# --- Trigger computation ---
result = annual_mean.compute()     # Now it actually runs
# OR
result = annual_mean.load()        # Loads into memory

# --- Persist in memory (keep for repeated access) ---
persisted = annual_mean.persist()  # Keeps distributed in workers

# --- Progress bar ---
from dask.diagnostics import ProgressBar
with ProgressBar():
    result = annual_mean.compute()
```

### 5.4 Chunking Best Practices for Climate Data

```python
# GOOD chunking (chunks along time only, full spatial grid per chunk)
chunks = {"time": 365, "lat": -1, "lon": -1}

# GOOD chunking for spatial operations (full time, chunked spatially)
chunks = {"time": -1, "lat": 30, "lon": 60}

# BAD: Too many tiny chunks (high overhead)
chunks = {"time": 1, "lat": 1, "lon": 1}

# Recommended chunk size: 100MB–500MB per chunk
# ERA5 hourly, global: ~1.4M points per time step at float32 = ~5.5MB
# So 100 time steps ≈ 550MB → good chunk

# Rule of thumb:
# - For time-axis operations (trends, climatology): chunk time, keep lat/lon whole
# - For spatial operations (interpolation, regridding): chunk lat/lon, keep time whole
```

### 5.5 Dask + apply_ufunc Pipeline

```python
# Full large-dataset workflow example
from dask.distributed import Client
import xarray as xr
import pymannkendall as mk

client = Client(n_workers=4)

# Load ERA5 (lazy)
ds = xr.open_mfdataset(
    "ERA5_t2m_1979_2023.nc",
    chunks={"time": 100}
)
da = ds["t2m"]

# Define trend function
def mk_slope(arr):
    try:
        result = mk.original_test(arr)
        return np.float32(result.slope)
    except:
        return np.float32(np.nan)

# Apply across grid points (parallelized via Dask)
slope_map = xr.apply_ufunc(
    mk_slope,
    da,
    input_core_dims=[["time"]],
    vectorize=True,
    dask="parallelized",
    output_dtypes=[np.float32],
)

# Compute and save
with ProgressBar():
    slope_map.compute().to_netcdf("mk_slope_ERA5.nc")

client.close()
```

---

## 📁 LEVEL 6 — Advanced Workflows

### 6.1 Spatial Interpolation & Regridding

```python
# Interpolate to a new grid
new_lat = np.arange(-90, 91, 1.0)   # 1° grid
new_lon = np.arange(-180, 181, 1.0)

da_regrid = da.interp(lat=new_lat, lon=new_lon, method="linear")

# Fill NaN after interpolation
da_regrid = da_regrid.interpolate_na(dim="lon", method="linear")
```

### 6.2 Weighted Area Means (Correct Climate Averaging)

```python
# Grid cells at equator are larger than at poles!
# Weight by cosine of latitude for correct area averaging

weights = np.cos(np.deg2rad(da.lat))
weights.name = "weights"

# Apply weights
weighted_mean = da.weighted(weights).mean(["lat", "lon"])
# Time series: area-weighted global/regional mean
```

### 6.3 Masking with regionmask / Custom Masks

```python
import regionmask

# Create country mask
countries = regionmask.defined_regions.natural_earth_v5_0_0.countries_110
mask = countries.mask(da)

# Extract Pakistan (region index ~61 for Pakistan)
pak_mask = mask == 61
pak_data = da.where(pak_mask)

# Area-weighted mean over Pakistan
weights = np.cos(np.deg2rad(da.lat))
pak_mean = pak_data.weighted(weights).mean(["lat", "lon"])
```

### 6.4 Complete Climate Analysis Pipeline

```python
import xarray as xr
import numpy as np
import pymannkendall as mk

# 1. Load data
ds = xr.open_mfdataset("ERA5_t2m_*.nc", chunks={"time": 100})
da = ds["t2m"] - 273.15   # Convert K to °C

# 2. Compute monthly climatology
clim = da.groupby("time.month").mean("time")

# 3. Compute monthly anomalies
anomaly = da.groupby("time.month") - clim

# 4. Annual mean anomaly
annual_anomaly = anomaly.resample(time="1Y").mean()

# 5. Smooth with 5-year rolling average
smooth = annual_anomaly.rolling(time=5, center=True).mean()

# 6. Trend detection
def slope(arr):
    result = mk.original_test(arr[~np.isnan(arr)])
    return result.slope

slope_map = xr.apply_ufunc(
    slope, annual_anomaly,
    input_core_dims=[["year"]],
    vectorize=True,
    dask="parallelized",
    output_dtypes=[float]
)

# 7. Save results
slope_map.to_netcdf("temperature_trend.nc")
```

---

## 📋 xarray Quick Reference Card

| Operation | Code |
|-----------|------|
| Open file | `xr.open_dataset("file.nc")` |
| Open multiple | `xr.open_mfdataset("*.nc", combine="by_coords")` |
| Select label | `da.sel(time="2020-01", method="nearest")` |
| Select index | `da.isel(time=0, lat=slice(10,50))` |
| Mask values | `da.where(condition)` |
| Mean over dim | `da.mean("time")` |
| Groupby month | `da.groupby("time.month").mean()` |
| Anomaly | `da.groupby("time.month") - clim` |
| Rolling mean | `da.rolling(time=12).mean()` |
| Resample | `da.resample(time="1Y").mean()` |
| Apply function | `xr.apply_ufunc(func, da, ...)` |
| Dask chunking | `xr.open_mfdataset(..., chunks={...})` |
| Compute | `da.compute()` |
| Save NC | `da.to_netcdf("output.nc")` |
| Weighted mean | `da.weighted(weights).mean(["lat","lon"])` |

---

## 📚 Resources

- [xarray Documentation](https://docs.xarray.dev)
- [xarray Tutorial (Official)](https://tutorial.xarray.dev)
- [Pangeo Project](https://pangeo.io) — Big data climate workflows
- [Project Pythia](https://projectpythia.org) — Earth science Python tutorials
