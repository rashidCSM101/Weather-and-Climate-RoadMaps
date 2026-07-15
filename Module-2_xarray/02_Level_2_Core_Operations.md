# 🛠️ LEVEL 2: Core Operations (Deep Dive)
## Indexing, Selection, and Resampling

Welcome to **Level 2**! Since you mentioned you are working with a dataset containing a `tmax` (Maximum Temperature) variable, we will tailor all the examples in this module around `tmax`. 

By the end of this module, you will know how to slice your data geographically, extract specific time periods, and calculate monthly or annual averages.

---

## 🎯 2.1 Indexing & Selection

In `xarray`, there are two main ways to select data:
1. **`isel()`**: Integer selection (like NumPy). "Give me the 5th latitude and 10th longitude."
2. **`sel()`**: Label selection (like Pandas). "Give me the data for 2020-05-15 at Latitude 33.73 and Longitude 73.09."

For climate science, **`sel()`** is almost always the better and safer choice.

### 📍 Selecting by Exact Labels (`sel`)

```python
import xarray as xr

# Let's assume you've loaded your dataset
ds = xr.open_dataset("your_tmax_dataset.nc")
tmax = ds["tmax"]  # Extract the DataArray

# 1. Select a specific date
tmax_july_1 = tmax.sel(time="2020-07-01")

# 2. Select a specific month (all days in July 2020)
tmax_july_all = tmax.sel(time="2020-07")

# 3. Select a specific year (all 365 days)
tmax_2020 = tmax.sel(time="2020")

# 4. Nearest Neighbor Selection (Crucial for Station Data!)
# You want the Tmax for Islamabad, but your grid might not land exactly on 33.73N.
# method="nearest" finds the closest grid cell automatically.
islamabad_tmax = tmax.sel(lat=33.73, lon=73.09, method="nearest")
print(islamabad_tmax) # This will be a 1D time series!
```

### ✂️ Slicing a Region (`slice`)

If you want to crop your global or regional dataset down to a specific bounding box (e.g., Pakistan), you use `slice`.

```python
# Slicing Pakistan (Approx: Lat 23 to 37, Lon 60 to 78)
# IMPORTANT: Check your latitude direction! 
# ERA5 latitudes go from 90 to -90 (North to South). 
# If so, your slice MUST go from North to South as well.

# For standard South-to-North latitudes:
pakistan_tmax = tmax.sel(lat=slice(23, 37), lon=slice(60, 78))

# For ERA5 (North-to-South latitudes):
pakistan_tmax_era5 = tmax.sel(lat=slice(37, 23), lon=slice(60, 78))

# Slicing time
tmax_decade = tmax.sel(time=slice("2010-01-01", "2019-12-31"))
```

### 🎭 Masking with `where()`

Sometimes you want to keep the shape of the data but mask out certain values (e.g., mask out the ocean, or mask out temperatures below freezing).

```python
# Keep only values where Tmax > 40°C (Extreme heat days). 
# All other values become NaN (Not a Number).
extreme_heat = tmax.where(tmax > 40.0)

# Replace values instead of putting NaN
# E.g., if Tmax > 50°C (maybe a sensor error), clip it to 50.
cleaned_tmax = tmax.where(tmax <= 50.0, other=50.0)
```

---

## 🧮 2.2 Arithmetic & Broadcasting

Because `xarray` labels your axes, you can do math without worrying about array shapes. It automatically aligns the data based on coordinates!

```python
# 1. Simple element-wise math
# Convert Kelvin to Celsius (if your tmax is in Kelvin)
tmax_celsius = tmax - 273.15

# Update the metadata so you don't forget!
tmax_celsius.attrs["units"] = "degC"

# 2. Calculating Anomalies (Subtracting a mean)
# Subtract the long-term average from the daily data
mean_tmax = tmax.mean(dim="time")
tmax_anomaly = tmax - mean_tmax

# 3. Diurnal Temperature Range (If you have tmin)
# Because they share 'time', 'lat', and 'lon', xarray does this pixel-by-pixel perfectly.
# dtr = ds["tmax"] - ds["tmin"]
```

---

## 📉 2.3 Reducing & Aggregation

Reductions collapse a dimension. If you have `(time, lat, lon)` and you take the `.mean("time")`, you are left with just `(lat, lon)` — a spatial map!

```python
# 1. Spatial Map (Climatology)
# Average over all time. Leaves lat and lon.
average_map = tmax.mean(dim="time")

# 2. Area-Averaged Time Series
# Average over space. Leaves time.
# NOTE: This is a simple mean. In climate science, grid cells near the poles are smaller than at the equator.
global_timeseries = tmax.mean(dim=["lat", "lon"])

# 3. CORRECT Area-Weighted Mean (Crucial!)
# Weight by the cosine of latitude
import numpy as np
weights = np.cos(np.deg2rad(tmax.lat))
weights.name = "weights"

weighted_timeseries = tmax.weighted(weights).mean(dim=["lat", "lon"])

# 4. Other Reductions
max_ever_recorded = tmax.max(dim="time") # Map of the hottest it has ever been at each pixel
temperature_variability = tmax.std(dim="time") # Map of standard deviation
```

---

## ⏳ 2.4 Resampling (Time Operations)

`resample()` is used when you want to change the **frequency** of your time series (e.g., going from Daily data to Monthly or Annual data).

The syntax is `resample(time="FREQUENCY").method()`.
Common frequencies:
- `"1MS"`: Monthly (Month Start)
- `"1YS"`: Annual (Year Start)
- `"QS-DEC"`: Seasonal (Quarter Start, anchored to December for DJF, MAM, JJA, SON)

```python
# Assume 'tmax' is DAILY data.

# 1. Calculate Monthly Mean Tmax
monthly_tmax = tmax.resample(time="1MS").mean()

# 2. Calculate Annual Maximum (Hottest day of each year)
annual_hottest_day = tmax.resample(time="1YS").max()

# 3. Calculate Seasonal Mean
seasonal_tmax = tmax.resample(time="QS-DEC").mean()

# How many heatwave days per year? (Tmax > 40°C)
heat_days = (tmax > 40.0) # Creates a boolean array (True/False)
annual_heat_days = heat_days.resample(time="1YS").sum() # Counts the 'True' values per year
```

---

## 🎯 Summary of Level 2

1. **`sel()`**: Slice and dice your data using real coordinate labels (dates, exact latitudes).
2. **`method="nearest"`**: The best way to extract a single station's time series from a grid.
3. **`mean(dim=...)`**: Collapse a dimension. Mean over time gives a map. Mean over lat/lon gives a time series.
4. **`resample()`**: Downsample your data from daily to monthly, seasonal, or annual resolutions.

---
### 🚦 Next Steps
In **Level 3: GroupBy & Rolling Windows**, we will cover how to create 30-year climatological normals (e.g., "What is the average January temperature?"), calculate anomalies properly, and apply moving averages.

Does the `resample()` or `sel()` logic make sense for your `tmax` dataset, or would you like to see a specific calculation applied to your data?
