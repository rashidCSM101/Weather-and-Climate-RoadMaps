# 🌩️ Climate Extremes Analysis Plan
## Using CDO and Python for Precipitation, Tasmax, and Tasmin

Since you have daily **Precipitation (pr)**, **Maximum Temperature (tasmax)**, and **Minimum Temperature (tasmin)**, you have the "Holy Trinity" of climate data. With these three variables, you can calculate the standard **ETCCDI Climate Change Indices** (Expert Team on Climate Change Detection and Indices).

Our strategy will be:
1. **Calculation**: Use **CDO (Climate Data Operators)**. CDO has built-in `eca_*` (European Climate Assessment) commands that calculate these extremes instantly.
2. **Analysis & Plotting**: Use **Python (xarray + matplotlib/cartopy)** to load the CDO outputs and create beautiful maps and time series.

---

## 🌡️ 1. Temperature Extremes (from `tasmax` and `tasmin`)

### Absolute Extremes (The absolute hottest/coldest days)
* **TXx (Hottest Day)**: The absolute maximum value of `tasmax` each month or year. (e.g., The peak of a summer heatwave).
* **TNn (Coldest Night)**: The absolute minimum value of `tasmin` each month or year. (e.g., The peak of a winter cold wave).
* **TNx (Warmest Night)**: The highest value of `tasmin`. (Important for human health, as nights don't cool down).
* **TXn (Coldest Day)**: The lowest value of `tasmax`.

### Threshold Indices (Counting days above/below a strict limit)
* **SU25 (Summer Days)**: Number of days in a year where `tasmax` > 25°C.
* **TR20 (Tropical Nights)**: Number of days in a year where `tasmin` > 20°C.
* **FD (Frost Days)**: Number of days in a year where `tasmin` < 0°C. (Crucial for agriculture).

### Percentile Indices (Relative to local climate)
* **TX90p (Warm Days)**: Percentage of days when `tasmax` > 90th percentile of the historical baseline.
* **TN10p (Cold Nights)**: Percentage of days when `tasmin` < 10th percentile.

---

## 🌧️ 2. Precipitation Extremes (from `pr`)

### Absolute Rainfall Amounts
* **Rx1day (Max 1-day rainfall)**: The highest amount of rain that fell in a single day each year/month. (Indicator for flash floods).
* **Rx5day (Max 5-day rainfall)**: The highest amount of rain that fell over 5 consecutive days. (Indicator for prolonged flooding / river overflow).

### Spell Indices (Consecutive days)
* **CDD (Consecutive Dry Days)**: The maximum number of consecutive days with less than 1mm of rain. (Indicator for Drought).
* **CWD (Consecutive Wet Days)**: The maximum number of consecutive days with >= 1mm of rain.

### Threshold Indices
* **R10mm (Heavy Rain Days)**: Number of days where precipitation >= 10mm.
* **R20mm (Very Heavy Rain Days)**: Number of days where precipitation >= 20mm.
* **SDII (Simple Daily Intensity Index)**: Average rainfall amount per wet day. (Total rain / Number of wet days).

---

## 💻 3. Our Workflow (CDO + Python)

We will do this step-by-step. For each index, the workflow will be:

**Step A (Terminal):**
We run a CDO command to do the heavy lifting.
Example: `cdo eca_txx tasmax_daily.nc txx_annual.nc`

**Step B (Jupyter/Python):**
We load `txx_annual.nc` in `xarray`, calculate the trend using `apply_ufunc`, and plot a map to see which regions are experiencing hotter maximums.

---
### 🚦 Next Step

Which extreme would you like to calculate first? I highly recommend starting with one of these three as they are the easiest to interpret and very powerful for Pakistan:

1. **TXx (Hottest Day)** using `tasmax`.
2. **Rx1day (Max 1-Day Rainfall - Flash Floods)** using `pr`.
3. **CDD (Consecutive Dry Days - Drought)** using `pr`.

Let me know which one you pick, and we will write the exact CDO command and Python code for it!
