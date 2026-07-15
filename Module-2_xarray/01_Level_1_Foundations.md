# 🚀 LEVEL 1: Foundations of xarray (Deep Dive)
## Core Data Structures & Loading Climate Data

Welcome to **Level 1** of the xarray roadmap! In this module, we will explore the absolute building blocks of `xarray`. By the end of this, you will understand how xarray represents climate data and how to efficiently load NetCDF files.

---

## 🏗️ 1.1 Core Data Structures

In climate science, we deal with multi-dimensional data. For example, Temperature can depend on **Time**, **Latitude**, **Longitude**, and sometimes **Pressure Level** (Altitude). 

NumPy arrays are great for numbers, but they don't know what the numbers *mean*. What is axis 0? Is it time or latitude? `xarray` solves this by adding **labels** to dimensions and coordinates.

There are two main data structures in xarray:
1. **`xr.DataArray`**: Represents a *single* variable (e.g., Temperature).
2. **`xr.Dataset`**: Represents a *collection* of variables (e.g., Temperature, Precipitation, and Wind together).

### 🌡️ The `DataArray`: Your Climate Variable

Think of a `DataArray` as a single NetCDF variable. It has 4 main components:
- **`values`**: The actual numbers (a NumPy array under the hood).
- **`dims`**: The names of the axes (e.g., `['time', 'lat', 'lon']`).
- **`coords`**: The actual labels for the axes (e.g., `time = 2020-01-01`, `lat = 30.5`).
- **`attrs`**: Metadata/attributes (e.g., `units = 'Celsius'`, `long_name = 'Surface Air Temperature'`).

#### 💻 Practical Example: Creating a Mock Climate DataArray

Let's create a fake Temperature `DataArray` over Pakistan.

```python
import xarray as xr
import numpy as np
import pandas as pd

# 1. Define our coordinates
times = pd.date_range("2023-01-01", periods=12, freq="MS") # 12 months (Jan to Dec)
lats = np.linspace(23.0, 37.0, 15)  # Pakistan latitudes (approx)
lons = np.linspace(60.0, 78.0, 19)  # Pakistan longitudes (approx)

# 2. Create fake temperature data (shape: 12 time x 15 lat x 19 lon)
# We use normal distribution around 25°C
fake_temp_data = np.random.normal(loc=25, scale=5, size=(len(times), len(lats), len(lons)))

# 3. Build the DataArray
temperature = xr.DataArray(
    data=fake_temp_data,
    dims=["time", "lat", "lon"], # Dimension names MUST match the shape of the data
    coords={
        "time": times,
        "lat": lats,
        "lon": lons
    },
    attrs={
        "units": "degC",
        "long_name": "Monthly Mean Near-Surface Air Temperature",
        "standard_name": "air_temperature"
    },
    name="t2m" # Variable name
)

print(temperature)
```

**Why is this powerful?** 
If you want to know the temperature at the first time step, you don't need to remember that `time` is axis 0. You can just ask xarray for it by name!

### 📦 The `Dataset`: The Full NetCDF File

A `Dataset` is like a Python dictionary that holds multiple `DataArray` objects. In climate data, an ERA5 NetCDF file is typically loaded as a `Dataset` because it might contain both Temperature (`t2m`) and Precipitation (`tp`), sharing the same dimensions (time, lat, lon).

#### 💻 Practical Example: Building a Dataset

```python
# Let's create fake precipitation data to go with our temperature
fake_precip_data = np.random.exponential(scale=2, size=(len(times), len(lats), len(lons)))

precipitation = xr.DataArray(
    data=fake_precip_data,
    dims=["time", "lat", "lon"],
    coords={"time": times, "lat": lats, "lon": lons},
    attrs={"units": "mm/day", "long_name": "Daily Precipitation"}
)

# Combine them into a Dataset
climate_ds = xr.Dataset(
    data_vars={
        "t2m": temperature,
        "precip": precipitation
    },
    attrs={
        "description": "Fake Climate Data for Pakistan",
        "history": "Created for xarray tutorial"
    }
)

print(climate_ds)
```

---

## 📂 1.2 Loading NetCDF Files

In the real world, you rarely create data manually like above. You load it from `.nc` files.

### 📖 The `xr.open_dataset()` function

This is your workhorse for opening a single NetCDF file.

```python
# Open a single file
ds = xr.open_dataset("path/to/your/ERA5_temperature.nc")

# xarray is smart. It will automatically:
# 1. Read the dimensions (time, lat, lon).
# 2. Decode the time variable (converts "hours since 1900-01-01" to standard YYYY-MM-DD datetime).
# 3. Apply 'scale_factor' and 'add_offset' (unpacks compressed data to real Floats).
# 4. Convert '_FillValue' (missing data points) to NaN.
```

### 🧠 Under the Hood: Decoding arguments

Sometimes, climate data is messy. You might need to control how xarray opens the file:

```python
ds = xr.open_dataset(
    "messy_climate_model.nc",
    decode_times=False,   # Use this if you get "unable to decode time units" error.
    mask_and_scale=False, # Use this if you want to see the raw packed integers (not recommended unless debugging).
    drop_variables=["expver"] # Drops unnecessary variables (ERA5 sometimes has a useless 'expver' coordinate)
)
```

### 📚 The `xr.open_mfdataset()` function (Multiple Files)

Climate data is heavy. Often, you will download ERA5 data year-by-year (e.g., `era5_2020.nc`, `era5_2021.nc`). 

`open_mfdataset` (Multi-File Dataset) is arguably xarray's most magical function. It combines hundreds of files into one logical Dataset without loading them all into your RAM!

```python
# Open all files matching a pattern
ds_all = xr.open_mfdataset(
    "data/ERA5_t2m_*.nc", # The * acts as a wildcard (matches 2020, 2021, 2022, etc.)
    combine="by_coords",  # Joins files by looking at their coordinate values (e.g., lines up the time axis)
    chunks={"time": 100}  # CRITICAL for large data: Tells xarray to use Dask for lazy-loading.
)

print(ds_all)
```

**What is `chunks`? (Dask Preview)**
When you use `chunks={"time": 100}`, xarray does not load the data into RAM. It creates a "blueprint" of the data. If you have 50 years of data (approx 50GB), xarray will open it instantly. It will only actually read the data when you plot it or save it! We will cover Dask deeply in Level 5.

---

## 🎯 Summary of Level 1.1 & 1.2

1. **`DataArray`**: Holds a single variable (e.g. Temperature) + its coordinates and metadata.
2. **`Dataset`**: Holds multiple variables (e.g. Temp + Precip).
3. **`xr.open_dataset(file.nc)`**: Loads a single NetCDF file.
4. **`xr.open_mfdataset("files_*.nc")`**: Loads and stitches together multiple NetCDF files seamlessly.

---
### 🚦 Next Steps
In **Level 2 (Core Operations)**, we will learn how to slice this data (e.g., "Give me only the data for July 2020 over Islamabad"). 

Would you like to practice these concepts on some real data, or shall we move on to **Level 2.1: Indexing & Selection**?
