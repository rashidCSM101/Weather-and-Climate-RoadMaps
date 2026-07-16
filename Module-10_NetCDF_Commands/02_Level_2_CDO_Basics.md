# ⚙️ LEVEL 2: CDO Basics (Climate Data Operators)
## The Swiss Army Knife of Climate Science

Welcome to **Level 2** of Module 10! 

**CDO (Climate Data Operators)** is a collection of over 600 command-line tools designed specifically for manipulating climate data. It is often much faster and more memory-efficient than Python for routine tasks. Many scientists use CDO to "prep" their data before bringing it into Python.

Every CDO command follows this basic syntax:
```bash
cdo [operator] [input_file.nc] [output_file.nc]
```

---

## ⏱️ 2.1 Temporal Operations (Time Averaging)

### `yearmean` (Annual Mean)
**Syntax:** `cdo yearmean input.nc output.nc`
- **Explanation:** Takes data (e.g., daily or monthly) and calculates the average for each year. If you feed it 365 daily files for 10 years, it will output exactly 10 values per pixel (one for each year).

🌍 **Real-Life Climate Use Case:**
You downloaded 45 years of *daily* Minimum Temperature (tasmin) data for the globe. Your computer struggles to open it. Before using Python, you run `cdo yearmean tasmin_global_daily_gridded_obs_cpc_1979-2024.nc annual_tasmin.nc`. You now have a tiny file with only 45 time steps, ready to plot a decadal warming trend!

### `monmean` (Monthly Mean)
**Syntax:** `cdo monmean input.nc output.nc`
- **Explanation:** Averages daily data into monthly chunks (Jan 1990, Feb 1990... Jan 1991...).

### `ymonmean` (Long-term Monthly Climatology / Normal)
**Syntax:** `cdo ymonmean input.nc output.nc`
- **Parameters:** Notice the `y` at the front (Year-Month-Mean). It averages all Januaries together, all Februarys together, across all years in your file. The output will have exactly 12 time steps.

🌍 **Real-Life Climate Use Case:**
You want to know the "normal" rainfall for the monsoon season (JJA). You take 30 years of precipitation data (1991-2020) and run `cdo ymonmean precip_1991_2020.nc climatology.nc`. You now have the 30-year expected baseline for each month.

### `ymonsub` (Monthly Anomaly)
**Syntax:** `cdo ymonsub input.nc climatology.nc anomaly.nc`
- **Explanation:** Subtracts the 12-month climatology file from the full input file. 
- **Parameters:** This command requires TWO input files. 1) The full data, 2) The 12-month baseline.

🌍 **Real-Life Climate Use Case:**
You want to study the 2010 Pakistan Floods. You run `cdo ymonsub precip_full.nc climatology.nc precip_anomalies.nc`. Now, every pixel shows exactly how many mm of rainfall fell *above the historical average*.

---

## 🗺️ 2.2 Spatial Operations (Subsetting & Regridding)

### `sellonlatbox` (Crop to a Bounding Box)
**Syntax:** `cdo sellonlatbox,lon1,lon2,lat1,lat2 input.nc output.nc`
- **Parameters:** 
  - `lon1,lon2`: Western and Eastern longitudes.
  - `lat1,lat2`: Southern and Northern latitudes.

🌍 **Real-Life Climate Use Case:**
You downloaded a **Global** CMIP6 model file (2GB). You only care about Pakistan. You run `cdo sellonlatbox,60,78,23,37 global_cmip6.nc pakistan_cmip6.nc`. Your file drops from 2GB to 5MB instantly! Now it is effortless to load in Python.

### `fldmean` (Field Mean / Spatial Average)
**Syntax:** `cdo fldmean input.nc output.nc`
- **Explanation:** Takes the average over all Latitudes and Longitudes. It turns a 3D map (time, lat, lon) into a 1D time series (time). 
- *Crucially, CDO automatically applies latitude-weighting (grid cells near the poles are smaller), which is physically accurate.*

🌍 **Real-Life Climate Use Case:**
After cropping to Pakistan using `sellonlatbox`, you run `cdo fldmean pakistan_tasmin.nc pak_timeseries.nc`. You now have a single time series representing the average minimum temperature of the entire country over time, perfect for a line graph.

### `remapbil` (Bilinear Interpolation / Regridding)
**Syntax:** `cdo remapbil,target_grid.txt input.nc output.nc`
- **Parameters:** `target_grid` is either a text file defining a grid, or simply the name of another NetCDF file you want to match! (e.g., `cdo remapbil,ERA5.nc CMIP6.nc CMIP6_on_ERA5_grid.nc`).

🌍 **Real-Life Climate Use Case:**
You want to compare CMIP6 model output (which has a coarse 1.0° resolution) with ERA5 observation data (which has a fine 0.25° resolution). You cannot subtract them if they have different grids! You run `cdo remapbil,ERA5.nc CMIP6.nc CMIP6_high_res.nc`. Now the CMIP6 data matches the ERA5 grid exactly, and you can calculate the model bias.

---

## 🔗 2.3 Multi-File Operations

### `mergetime` (Combine Files Over Time)
**Syntax:** `cdo mergetime file1.nc file2.nc file3.nc output.nc`
- *Pro-tip Syntax:* `cdo mergetime tasmin_*.nc tasmin_all.nc`
- **Explanation:** Concatenates multiple files chronologically. 

🌍 **Real-Life Climate Use Case:**
You downloaded CPC data decade-by-decade because the server wouldn't allow a single massive download. You have files named `tasmin_1980s.nc`, `tasmin_1990s.nc`, etc. You run `cdo mergetime tasmin_*.nc full_dataset.nc` to stitch them all into one single file for Python to read.

---

## ⛓️ 2.4 Chaining Commands (The True Power of CDO)

You don't have to save intermediate files. You can chain CDO commands together by adding a minus sign (`-`) before the operator. CDO evaluates from **Right to Left**.

**Syntax:**
```bash
cdo [Operator_3] -[Operator_2] -[Operator_1] input.nc output.nc
```

🌍 **Real-Life Climate Use Case:**
You have a global daily dataset. You want the **Annual Mean** over **Pakistan** only.
Instead of doing it in two steps, you do:
```bash
cdo yearmean -sellonlatbox,60,78,23,37 tasmin_global_daily_gridded_obs_cpc_1979-2024.nc pak_annual_tasmin.nc
```
*How it reads (Right to Left):* First, take `tasmin_global_daily_gridded_obs_cpc_1979-2024.nc` and crop it to Pakistan (`sellonlatbox`). THEN, take that result and calculate the annual average (`yearmean`). Finally, save it to `pak_annual_tasmin.nc`.

---

## 🎯 Summary

CDO is perfect for bulk preprocessing. Before opening Jupyter Notebook, use CDO to:
1. Crop the data to your region (`sellonlatbox`).
2. Merge multiple downloaded files into one (`mergetime`).
3. Downsample daily data to monthly/annual if you don't need daily (`monmean` / `yearmean`).

---
### 🚦 Next Steps

You now know how to manipulate standard climate files from the terminal!

Are you ready to move on to **Level 3: CDO ECA Indices (Climate Extremes)**? This is where we will learn exactly how to use CDO to calculate things like **TXx (Hottest day of the year)**, **TNn (Coldest night)**, **Rx1day (Maximum 1-day rainfall)**, and **Rx5day**!
