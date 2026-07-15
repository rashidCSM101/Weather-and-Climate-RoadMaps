# 🔄 LEVEL 3: GroupBy & Rolling Windows
## Climatologies, Anomalies, and Moving Averages

Welcome to **Level 3**. This is where `xarray` shines brightest for climate scientists. 

Almost all climate studies deal with "normals" (What is the usual temperature for July?) and "anomalies" (Was this July hotter than usual?). We also use rolling windows to smooth out noisy daily weather to see long-term trends.

Let's dive into how to do this effortlessly with `groupby` and `rolling`.

---

## 🗂️ 3.1 GroupBy: Calculating Climatologies

`groupby()` is used when you want to group your data by a specific time component (like month, season, or day of the year) and then calculate a statistic (like the mean) for each group.

### 📅 Monthly Climatology

🌍 **Real-Life Climate Use Case:** 
You have 30 years of daily `tmax` data (1991–2020) for Pakistan. You want to know the *average expected maximum temperature* for each of the 12 months. This is called a **30-year normal** or **climatology**. 

```python
import xarray as xr

# Load 30 years of data
ds = xr.open_dataset("tmax_1991_2020.nc")
tmax = ds["tmax"]

# Group all the Januarys together, all the Februarys together, etc.
# Then calculate the mean over the "time" dimension.
monthly_climatology = tmax.groupby("time.month").mean(dim="time")

print(monthly_climatology)
# The resulting DataArray will have a new dimension called 'month' (size 12)
# instead of 'time'. So shape is now (12, lat, lon).
```

### 🍂 Seasonal Climatology

🌍 **Real-Life Climate Use Case:** 
Instead of months, you want to know the average temperature for the 4 major seasons (DJF, MAM, JJA, SON) across your study period to see which regions of Pakistan have the hottest summers (JJA).

```python
# Group by season
seasonal_climatology = tmax.groupby("time.season").mean(dim="time")

print(seasonal_climatology)
# The 'season' dimension will have 4 values: ['DJF', 'JJA', 'MAM', 'SON']
```

### 🗓️ Annual Cycle (Day of Year)

🌍 **Real-Life Climate Use Case:** 
You are studying heatwaves and need to know the *exact average temperature for every single day of the year* (e.g., average temp for January 1st, January 2nd, ... July 15th). 

```python
# Group by the 365 days of the year
annual_cycle = tmax.groupby("time.dayofyear").mean(dim="time")

# The 'dayofyear' dimension will have 365 values (1 to 365)
```

---

## 📉 3.2 Anomalies: Removing the Baseline

An anomaly is simply the actual value minus the climatological average. It tells you how much hotter or colder a specific time was compared to "normal."

🌍 **Real-Life Climate Use Case:** 
In May 2022, Pakistan experienced a brutal heatwave. You want to create a map showing exactly *how many degrees hotter* May 2022 was compared to the historical average for May.

```python
# Step 1: Calculate the 30-year baseline (Climatology)
baseline = tmax.sel(time=slice("1991", "2020"))
monthly_climatology = baseline.groupby("time.month").mean(dim="time")

# Step 2: Calculate the Anomaly for the ENTIRE dataset
# xarray is incredibly smart here. It automatically matches the month!
# If it's a January day, it subtracts the January climatology.
# If it's a July day, it subtracts the July climatology.
tmax_anomaly = tmax.groupby("time.month") - monthly_climatology

# Step 3: Extract the specific heatwave month to map it
may_2022_anomaly = tmax_anomaly.sel(time="2022-05")

# Now you can plot `may_2022_anomaly` to show the heatwave!
```

---

## 🌀 3.3 Rolling Windows (Moving Averages)

Weather data is noisy. One day might be 40°C, the next 32°C. A **rolling window** smooths this out by taking the average of the surrounding days.

The syntax is `.rolling(time=WINDOW_SIZE).mean()`.

### 🌤️ 7-Day Centered Moving Average

🌍 **Real-Life Climate Use Case:** 
You want to identify prolonged hot periods, not just single hot days. A 7-day rolling average of `tmax` smooths out the daily spikes, allowing you to see the true underlying heatwave.

```python
# Calculate a 7-day rolling mean.
# center=True means the value for Wednesday is the average of (Sun, Mon, Tue, WED, Thu, Fri, Sat).
tmax_7day_smoothed = tmax.rolling(time=7, center=True).mean()
```

### 📊 Multi-Year Rolling Average (Smoothing Decadal Trends)

🌍 **Real-Life Climate Use Case:** 
You have 100 years of annual temperature data. You want to plot a line graph showing the long-term climate change trend, but the year-to-year El Niño/La Niña variations make the line too jagged. You apply a **5-year or 10-year rolling mean** to reveal the smooth, underlying global warming trend.

```python
# First, calculate the Annual Mean Temperature
annual_tmax = tmax.resample(time="1YS").mean()

# Then, apply a 10-year rolling average to smooth the interannual noise
decadal_trend_smoothed = annual_tmax.rolling(time=10, center=True).mean()
```

### 🌧️ Rolling Sums (Accumulations)

*(Bonus concept using Precipitation instead of Temperature!)*

🌍 **Real-Life Climate Use Case:** 
You are studying droughts or floods using daily precipitation (`pr`). You need to know the total rainfall over the last 30 days to see if the soil is waterlogged or parched.

```python
# Assuming you have daily precipitation data
# This calculates the total rainfall over the previous 30 days for EVERY day in the dataset.
precip_30day_total = pr.rolling(time=30).sum()
```

---

## 🔗 3.4 Combining GroupBy and Rolling (Advanced)

🌍 **Real-Life Climate Use Case:** 
You want to create a smooth annual cycle curve. If you just take the `groupby("time.dayofyear")` mean, the curve can still be jagged (e.g., historical July 15th might have been unusually cold by random chance). You apply a rolling mean *to the climatology itself* to smooth the annual curve.

```python
# 1. Get raw day-of-year average (365 values)
raw_annual_cycle = tmax.groupby("time.dayofyear").mean("time")

# 2. Smooth the 365-day curve with a 15-day rolling window
smooth_annual_cycle = raw_annual_cycle.rolling(dayofyear=15, center=True).mean()
```

---

## 🎯 Summary of Level 3

1. **`groupby("time.month")`**: The easiest way to calculate your 30-year normal climatologies.
2. **`groupby(...) - climatology`**: The standard way to calculate temperature anomalies (how far above/below normal a day was).
3. **`rolling(time=N).mean()`**: How to smooth out noisy weather data to see heatwaves (7-day) or long-term climate change trends (10-year).

---
### 🚦 Next Steps
You have now mastered the core manipulations of climate data! 

In **Level 4: `apply_ufunc`**, we will look at how to apply custom statistical functions (like scipy's linear regression to calculate warming trends) across the entire grid. 

Do the real-life examples above help clarify how you might use these tools on your `tmax` dataset? Are you ready to dive into calculating statistical trends across the map in Level 4?
