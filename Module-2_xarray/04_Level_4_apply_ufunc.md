# 🚀 LEVEL 4: `apply_ufunc`
## Custom Functions and Wrapping External Libraries

Welcome to **Level 4**! This is arguably the most powerful (and slightly complex) function in `xarray`.

`xarray` has built-in functions for simple things (like `.mean()` or `.std()`). But what if you want to apply a complex statistical model—like a **linear regression**, a **Mann-Kendall trend test**, or a custom climate index—to *every single pixel* on your map?

This is where `xr.apply_ufunc` (Apply Universal Function) comes in. It takes a function designed for simple 1D NumPy arrays and applies it across the multi-dimensional xarray structure while preserving your coordinates (lat, lon, time).

---

## 🛠️ 4.1 The Syntax & Parameters of `apply_ufunc`

The syntax can look intimidating at first. Here is the structure:

```python
import xarray as xr

result = xr.apply_ufunc(
    func,                     # 1. The custom function you want to apply
    *args,                    # 2. The xarray DataArrays you are feeding into the function
    input_core_dims=None,     # 3. Which dimensions the function consumes
    output_core_dims=None,    # 4. Which new dimensions the function creates (if any)
    vectorize=False,          # 5. Should xarray loop over the grid for you?
    dask="forbidden",         # 6. How to handle parallel processing (important for large data)
    output_dtypes=None,       # 7. The data type of the output (e.g., float)
    kwargs=None               # 8. Extra arguments to pass to your custom function
)
```

### 🧠 Parameter Breakdown

1. **`func`**: The Python function you want to run. It must accept NumPy arrays, not xarray objects.
2. **`*args`**: The `DataArray`(s) you are processing (e.g., your `tmax` variable).
3. **`input_core_dims`**: A list of lists. If your function calculates a trend over time, the core dimension is `"time"`. You are telling xarray: *"Give my function 1D slices of time for each lat/lon pixel."* (e.g., `[["time"]]`).
4. **`output_core_dims`**: If your function returns a single number per pixel (like a trend slope), this is empty `[[]]`. If it returns a time series, it's `[["time"]]`.
5. **`vectorize`**: Set this to `True` if your `func` only knows how to handle 1D arrays (like `scipy.stats.linregress`). xarray will automatically loop over all latitudes and longitudes for you!
6. **`dask`**: Set to `"parallelized"` if you want it to run on multiple CPU cores simultaneously (highly recommended for climate maps).
7. **`output_dtypes`**: The data type you expect back (e.g., `[float]`). Required if using Dask.

---

## 📈 4.2 Calculating Linear Warming Trends (scipy.stats)

🌍 **Real-Life Climate Use Case:**
You have 40 years of Annual Maximum Temperature (`tmax`) for Pakistan. You want to create a map showing the **Warming Trend (°C per year)** for every single district. 

To do this, we need to run a linear regression (from `scipy.stats`) on the time series of *every single pixel*.

### Step 1: Write the 1D function
First, write a function that calculates the trend for a simple 1D NumPy array (one pixel's history).

```python
from scipy.stats import linregress
import numpy as np

def get_warming_trend(time_series_1d):
    """Takes a 1D array of temperatures and returns the slope (°C/year)."""
    # Create an X-axis (0, 1, 2, ..., N years)
    x = np.arange(len(time_series_1d))
    
    # Run the linear regression
    result = linregress(x, time_series_1d)
    
    # We only care about the slope (rate of warming)
    return result.slope
```

### Step 2: Apply it to the whole map using `apply_ufunc`

```python
# Assume annual_tmax is your DataArray of shape (time: 40, lat: 100, lon: 100)

warming_map = xr.apply_ufunc(
    get_warming_trend,               # The function we just wrote
    annual_tmax,                     # The DataArray we are feeding in
    input_core_dims=[["time"]],      # The function needs the 'time' dimension
    output_core_dims=[[]],           # The output is a single number (slope), so no core dims
    vectorize=True,                  # Loop over lat/lon automatically!
    dask="parallelized",             # Run in parallel to make it fast
    output_dtypes=[float]            # The slope is a float
)

print(warming_map)
# Result: A 2D DataArray (lat, lon) showing the warming trend per pixel!
```

---

## ⚖️ 4.3 Standardized Anomalies (Z-Scores)

🌍 **Real-Life Climate Use Case:**
You want to identify Extreme Drought or Extreme Heat periods. A standard way to do this is a **Z-Score** (Standardized Anomaly). It tells you how many standard deviations a value is away from the mean. 

If Z > 2, it's exceptionally hot. If Z < -2, it's exceptionally cold. 

We can use `scipy.stats.zscore`.

### The Code

```python
from scipy.stats import zscore

# Let's say you have monthly tmax data over 30 years
# shape: (time: 360, lat: 100, lon: 100)

zscore_map = xr.apply_ufunc(
    zscore,                         # The scipy function
    monthly_tmax,                   # The input DataArray
    input_core_dims=[["time"]],     # We want to calculate the z-score across time
    output_core_dims=[["time"]],    # The output ALSO has a time dimension (it gives a Z-score for every month!)
    vectorize=True,                 # Not strictly needed for zscore, but safe.
    dask="parallelized",
    output_dtypes=[float]
)

# Result: A DataArray of shape (time, lat, lon) where every value is now a Z-Score.
# You can now easily mask extreme events:
extreme_heat_months = monthly_tmax.where(zscore_map > 2.0)
```

---

## 🚦 4.4 Multiple Outputs (Returning Slope AND P-Value)

Often, statistical tests return more than one thing. For example, a linear regression returns a `slope` (the trend) AND a `pvalue` (is the trend statistically significant?). 

🌍 **Real-Life Climate Use Case:**
When publishing a climate paper, you cannot just show a map of warming trends. You must also show **Statistical Significance** (usually p < 0.05). Let's extract both at the same time.

### Step 1: The 1D Function returning 2 things

```python
def get_trend_and_pvalue(time_series_1d):
    # Handle NaN values (e.g., if the pixel is over the ocean and you masked it)
    if np.isnan(time_series_1d).all():
        return np.nan, np.nan
        
    x = np.arange(len(time_series_1d))
    result = linregress(x, time_series_1d)
    
    # Return TWO variables
    return result.slope, result.pvalue
```

### Step 2: Applying it

Notice the syntax changes slightly for `output_core_dims` and `output_dtypes`.

```python
# apply_ufunc will return a TUPLE of DataArrays
slope_map, pvalue_map = xr.apply_ufunc(
    get_trend_and_pvalue,
    annual_tmax,
    input_core_dims=[["time"]],
    
    # We are returning 2 scalars, so we need two empty lists!
    output_core_dims=[[], []],       
    
    vectorize=True,
    dask="parallelized",
    
    # We are returning 2 floats
    output_dtypes=[float, float]     
)

# Now you can easily mask insignificant trends!
# Keep the slope only where p-value is less than 0.05 (95% confidence)
significant_warming = slope_map.where(pvalue_map < 0.05)
```

---

## 🎯 Summary of Level 4

1. **`apply_ufunc`** is your bridge between simple Python/SciPy functions and massive multi-dimensional climate datasets.
2. **`input_core_dims=[["time"]]`**: Tells xarray to collapse the time dimension into a 1D array to feed to your function.
3. **`vectorize=True`**: The magic parameter that loops your 1D function over every latitude and longitude point.
4. **`output_core_dims`**: If returning a single statistic (like a trend), use `[[]]`. If returning a time series (like z-scores), use `[["time"]]`.

---
### 🚦 Next Steps

You have now completed the core functionality of xarray! 

In **Level 5: Dask Integration**, we will cover how to handle massive climate files (like a 50GB global ERA5 file) without crashing your computer's RAM. 

Are you ready to move on to Level 5, or do you have any questions about how `apply_ufunc` maps functions across dimensions?
