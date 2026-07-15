# 🧠 LEVEL 5: Dask Integration
## Handling Massive Climate Datasets (Out-of-Core Processing)

Welcome to **Level 5**. If you have ever tried to open 40 years of daily global climate data and your Python script froze or crashed because you ran out of RAM, **Dask** is the solution.

Dask works seamlessly under the hood of `xarray` to provide **Lazy Evaluation** and **Parallel Processing**. 

---

## 🏗️ 5.1 What is Dask and "Lazy Evaluation"?

When you load a file with Dask, `xarray` doesn't actually read the data into your computer's memory (RAM). Instead, it just reads the metadata (coordinates, dimension sizes) and creates a "blueprint" or "task graph." 

It only actually reads the data and does the math when you explicitly ask for the final answer (e.g., when you plot it or save it to a new file).

### 📖 The Syntax: Activating Dask via `chunks`

You activate Dask simply by passing the `chunks` parameter when opening a dataset.

```python
import xarray as xr

# WITHOUT DASK (Will crash if file is larger than your RAM)
# ds = xr.open_mfdataset("data/ERA5_tmax_*.nc") 

# WITH DASK (Instant! Uses almost 0 RAM)
ds = xr.open_mfdataset(
    "data/ERA5_tmax_*.nc",
    chunks={"time": 365, "lat": -1, "lon": -1} 
)
```

### 🧠 Parameter Breakdown: The `chunks` dictionary
The `chunks` parameter tells Dask how to cut the massive dataset into smaller, bite-sized blocks.
- **`"time": 365`**: Cut the data into blocks of 365 days. Dask will process one year at a time.
- **`"lat": -1` and `"lon": -1`**: The `-1` means "Do NOT chunk this dimension." Keep the entire globe intact for each time step.

🌍 **Real-Life Climate Use Case:**
You downloaded 50 years of ERA5 hourly temperature data globally. That's over 1 Terabyte of data! Your laptop only has 16GB of RAM. By using `chunks={"time": 24}`, xarray will only load one day of data (24 hours) into RAM at a time, calculate the daily maximum (`tmax`), save the result, throw the raw data out of RAM, and move to the next day. You just processed 1TB of data on a standard laptop!

---

## 🗂️ 5.2 Chunking Strategies (How to chunk correctly)

Chunking incorrectly can actually make your code *slower*. The rule of thumb is: **Your chunk size should be between 10MB and 100MB.**

You can check your chunk sizes by printing your DataArray:
```python
tmax = ds["tmax"]
print(tmax.data)  # Will say "dask.array<chunksize=(365, 721, 1440), meta=np.ndarray>"
```

### ⚙️ Syntax: Re-chunking existing data

Sometimes you need to change the chunks after loading. You use `.chunk()`.

```python
# Re-chunk to process smaller spatial blocks but all of time.
tmax_rechunked = tmax.chunk({"time": -1, "lat": 100, "lon": 100})
```

### 🌍 Real-Life Use Cases for Different Chunking Strategies:

**Scenario A: Calculating an Annual Climatology**
*You are calculating `mean(dim='time')` over 50 years.*
* **Strategy**: You want all latitudes and longitudes together, but you can process time in pieces.
* **Chunks**: `chunks={"time": 365, "lat": -1, "lon": -1}` (Chunk along time).

**Scenario B: Calculating a Linear Trend Map**
*You are applying a linear regression to the time series of every single pixel independently.*
* **Strategy**: You need the ENTIRE time series available at once for a pixel to calculate a trend. You cannot chunk time! Instead, chunk space.
* **Chunks**: `chunks={"time": -1, "lat": 50, "lon": 50}` (Process 50x50 pixel blocks at a time, but give them the full time history).

---

## 🚦 5.3 Triggering Computation (`compute`, `load`, `persist`)

Because Dask is "lazy", doing `mean_map = tmax.mean("time")` happens instantly, but `mean_map` contains no actual numbers yet! It's just a plan. 

You must trigger the computation.

### The Syntax

1. **`.compute()`**: Runs the math and returns a normal (in-memory) xarray DataArray.
2. **`.load()`**: Runs the math and replaces the Dask array inside your existing object with real data.
3. **`.persist()`**: Runs the math and keeps it distributed in Dask memory (useful if you are connected to a cluster of computers).

```python
# 1. Setup the lazy computation
annual_mean = tmax.resample(time="1YS").mean()

# 2. Add a progress bar (Highly Recommended!)
from dask.diagnostics import ProgressBar

with ProgressBar():
    # 3. TRIGGER THE MATH!
    real_annual_mean = annual_mean.compute()

# Now real_annual_mean has actual numpy arrays in it.
```

🌍 **Real-Life Climate Use Case:**
You want to plot the `annual_mean` map. If you try to run `annual_mean.plot()`, xarray will automatically trigger `.compute()` for you under the hood! However, if you plan to plot it, save it, AND print the max value, you should manually `.compute()` it *first*. Otherwise, Dask will read the 50 years of data 3 separate times!

---

## 🚀 5.4 Using Dask with `apply_ufunc`

In Level 4, we learned `apply_ufunc`. If your data is chunked with Dask, you MUST pass specific parameters to `apply_ufunc` to tell it how to handle the parallel processing.

### The Syntax

```python
from scipy.stats import linregress

def get_trend(y):
    x = np.arange(len(y))
    return linregress(x, y).slope

# Assume 'tmax' is Dask-chunked as {"time": -1, "lat": 50, "lon": 50}

trend_map_lazy = xr.apply_ufunc(
    get_trend,
    tmax,
    input_core_dims=[["time"]],
    output_core_dims=[[]],
    vectorize=True,
    
    # --- DASK SPECIFIC PARAMETERS ---
    dask="parallelized",   # Tell xarray to map this function across the Dask chunks in parallel!
    output_dtypes=[float]  # Dask REQUIRES you to tell it what data type the result will be, since it can't run it to find out!
)

# Trigger the calculation across all your CPU cores!
with ProgressBar():
    final_trend_map = trend_map_lazy.compute()
```

### 🧠 Parameter Breakdown for Dask in `apply_ufunc`:
- **`dask="parallelized"`**: This is the magic switch. It tells Dask to distribute the `get_trend` function across all your CPU cores. If you chunked your map into 100 blocks, Dask will send block 1 to Core A, block 2 to Core B, etc.
- **`output_dtypes=[float]`**: Because Dask doesn't run the code immediately, it doesn't know if `linregress` will return an integer, a float, or a string. You must explicitly tell Dask what to expect so it can allocate the right amount of memory for the final output.

---

## 🎯 Summary of Level 5

1. **`chunks={}`**: Pass this to `open_mfdataset` to keep data out of RAM and activate Dask.
2. **Chunking Rule**: Chunk the dimension you don't need all at once. (e.g., for spatial maps, chunk time. For time-series trends, chunk space).
3. **`.compute()`**: The command that says "Stop planning and actually do the math now!"
4. **`dask="parallelized"`**: Allows you to run custom Python functions over massive datasets simultaneously on all your CPU cores.

---
### 🚦 Next Steps

You have now conquered big data processing in Python! 

In our final module, **Level 6: Advanced Workflows**, we will put everything together into a complete pipeline. We will cover things like **Spatial Regridding** (interpolating data to a new grid) and **Masking with Shapefiles** (extracting Pakistan from the global grid). 

Are you ready to move to Level 6?
