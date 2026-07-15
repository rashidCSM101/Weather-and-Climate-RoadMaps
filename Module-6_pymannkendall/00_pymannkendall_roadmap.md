# 📈 pymannkendall — Complete Learning Roadmap
## Non-Parametric Trend Detection for Climate Analysis

---

## 📚 What is pymannkendall?

The Mann-Kendall test is the **most widely used method for detecting trends in climate data**. It is:
- **Non-parametric**: Does NOT assume data follows a normal distribution
- **Robust**: Works even with skewed data (like precipitation) or outliers
- **Rank-based**: Only considers whether values go up or down, not their magnitude
- **Widely accepted**: Recommended by WMO for climate trend analysis

The `pymannkendall` library provides 8 variants of the Mann-Kendall test in Python, making it the essential tool for climate trend analysis.

---

## 🗺️ Roadmap Overview

```
Level 1: Statistical Background    → Why MK? What does it test?
Level 2: pymannkendall Basics      → Installation, basic usage, interpreting results
Level 3: All 8 MK Variants        → When to use each type
Level 4: Gridded Data Analysis     → Apply to every lat/lon point (climate maps)
Level 5: Station Data Workflow     → Multiple stations, batch analysis
Level 6: Trend Significance Maps   → Visualizing MK results
Level 7: Combining with Sen's Slope→ Magnitude + direction of trends
Level 8: Complete Climate Pipeline → Full workflow
```

---

## 📁 LEVEL 1 — Statistical Background

### 1.1 Why Mann-Kendall?

| Method | Assumption | Climate Use |
|--------|-----------|------------|
| Linear regression (OLS) | Normal distribution, constant variance | Only for normally distributed data |
| **Mann-Kendall** | None (non-parametric) | ✅ Temperature, precipitation, humidity, any climate variable |
| Spearman correlation | Rank-based | Good alternative but less common |

**Key properties:**
- H₀ (null hypothesis): **No monotonic trend** in the data
- H₁ (alternative): **Monotonic upward or downward trend** exists
- p < 0.05 → Trend is statistically significant

### 1.2 MK Statistic (S) — The Concept

```
For a time series [x₁, x₂, x₃, ..., xₙ]:

For every pair (xᵢ, xⱼ) where i < j:
  - If xⱼ > xᵢ: score = +1 (upward tendency)
  - If xⱼ < xᵢ: score = -1 (downward tendency)
  - If xⱼ = xᵢ: score = 0 (no change)

S = Σ all scores

- S >> 0: Upward trend
- S << 0: Downward trend
- S ≈ 0: No trend

Sen's Slope = Median of all (xⱼ - xᵢ) / (j - i)
  → Actual magnitude of trend (units per time step)
```

---

## 📁 LEVEL 2 — pymannkendall Basics

### 2.1 Installation

```bash
pip install pymannkendall
```

### 2.2 Basic Usage

```python
import pymannkendall as mk
import numpy as np
import pandas as pd

# Example: Annual temperature series
temp = np.array([
    24.1, 24.3, 24.0, 24.5, 24.8, 25.1, 24.9, 25.3,
    25.5, 25.2, 25.8, 26.0, 25.9, 26.2, 26.5
])

# Run original Mann-Kendall test
result = mk.original_test(temp)
print(result)

# --- UNDERSTANDING THE RESULT OBJECT ---
print(result.trend)       # "increasing", "decreasing", or "no trend"
print(result.h)           # True if trend is significant (p < alpha)
print(result.p)           # p-value
print(result.z)           # Normalized test statistic (z-score)
print(result.Tau)         # Kendall's Tau (rank correlation, -1 to +1)
print(result.s)           # Mann-Kendall statistic (S)
print(result.var_s)       # Variance of S
print(result.slope)       # Sen's slope (trend per time step)
print(result.intercept)   # Intercept of Sen's slope line

# --- Interpretation ---
print(f"\nTrend direction: {result.trend}")
print(f"Significant (α=0.05)? {result.h}")
print(f"p-value: {result.p:.4f}")
print(f"Sen's slope: {result.slope:.4f} °C/year")
print(f"Kendall's τ: {result.Tau:.4f}")

# Example output:
# Trend direction: increasing
# Significant (α=0.05)? True
# p-value: 0.0023
# Sen's slope: 0.0421 °C/year
# Kendall's τ: 0.619
```

### 2.3 Custom Alpha Level

```python
# Change significance level (default: alpha=0.05)
result = mk.original_test(temp, alpha=0.01)   # 99% confidence
result = mk.original_test(temp, alpha=0.10)   # 90% confidence
```

### 2.4 Result Summary Function

```python
def summarize_mk(result, variable_name="Variable"):
    """Print a clean summary of Mann-Kendall results."""
    stars = "***" if result.p < 0.001 else "**" if result.p < 0.01 else "*" if result.p < 0.05 else "ns"
    print(f"\n{'='*50}")
    print(f"Mann-Kendall Trend Test: {variable_name}")
    print(f"{'='*50}")
    print(f"  Trend:        {result.trend.upper()}")
    print(f"  Significant:  {'YES' if result.h else 'NO'} ({stars})")
    print(f"  p-value:      {result.p:.4f}")
    print(f"  z-statistic:  {result.z:.4f}")
    print(f"  Kendall's τ:  {result.Tau:.4f}")
    print(f"  Sen's slope:  {result.slope:.5f} per time step")
    print(f"  Intercept:    {result.intercept:.4f}")
    print(f"{'='*50}\n")

summarize_mk(result, "Annual Temperature (Islamabad)")
```

---

## 📁 LEVEL 3 — All 8 Mann-Kendall Variants

### 3.1 Overview of All Tests

| Test | Function | Best For |
|------|----------|---------|
| Original | `original_test()` | Basic trend, no autocorrelation |
| Hamed & Rao | `hamed_rao_modification_test()` | **Recommended** — handles autocorrelation |
| Yue & Wang | `yue_wang_modification_test()` | Handles both autocorrelation & seasonality |
| Mann-Kendall Modified | `modified_mann_kendall_test()` | Variance correction for autocorrelated data |
| Seasonal | `seasonal_test()` | Seasonal data (monthly time series) |
| Regional | `regional_test()` | Multiple stations, regional trend |
| Correlated | `correlated_multivariate_test()` | Multiple variables simultaneously |
| Partial | `partial_tau_test()` | Remove influence of a covariate |

### 3.2 Original Mann-Kendall

```python
# USE WHEN: Data has no autocorrelation (independent observations)
# COMMON FOR: Annual means, decadal values

result = mk.original_test(temp_annual)
print(result.trend, result.p, result.slope)
```

### 3.3 Hamed & Rao Modified (RECOMMENDED for Climate)

```python
# USE WHEN: Data has autocorrelation (most climate time series do!)
# WHY: Most daily/monthly temperature/precipitation series are autocorrelated
#      The original MK test OVERESTIMATES significance in autocorrelated data
# THIS IS THE DEFAULT CHOICE FOR CLIMATE TREND ANALYSIS

result_hr = mk.hamed_rao_modification_test(temp_annual)
print("Hamed-Rao (autocorrelation-corrected):")
print(f"  p = {result_hr.p:.4f}, slope = {result_hr.slope:.4f}")

# Compare with original
result_orig = mk.original_test(temp_annual)
print("\nOriginal (uncorrected):")
print(f"  p = {result_orig.p:.4f}")

# The Hamed-Rao p-value is typically LARGER (more conservative = more correct)
```

### 3.4 Seasonal Mann-Kendall

```python
# USE WHEN: Monthly time series (accounts for seasonality)
# WHY: Monthly data has strong seasonal cycle; comparing Jan to Jun is meaningless
#      Seasonal MK compares each month only to the same month in other years

monthly_temp = np.random.rand(480)   # 40 years × 12 months
result_seasonal = mk.seasonal_test(monthly_temp, period=12)  # period=12 months

print("Seasonal MK test (for monthly data):")
print(f"  Trend: {result_seasonal.trend}")
print(f"  p-value: {result_seasonal.p:.4f}")
print(f"  Sen's slope: {result_seasonal.slope:.5f} per month")
```

### 3.5 Yue & Wang Modification

```python
# USE WHEN: Data has both autocorrelation AND trend influence on variance
# This pre-whitens the series to remove trend before testing autocorrelation

result_yw = mk.yue_wang_modification_test(temp_series)
print(f"Yue-Wang: trend={result_yw.trend}, p={result_yw.p:.4f}")
```

### 3.6 Regional Mann-Kendall

```python
# USE WHEN: Testing regional trend from multiple correlated stations
# Input: 2D array (time × stations)

station_data = np.column_stack([temp_isb, temp_khi, temp_lhr, temp_psh])
result_regional = mk.regional_test(station_data)
print(f"Regional trend: {result_regional.trend}, p={result_regional.p:.4f}")
```

---

## 📁 LEVEL 4 — Gridded Data Analysis (Climate Maps)

### 4.1 Apply MK to Every Grid Point

```python
import xarray as xr
import numpy as np
import pymannkendall as mk

# Load annual mean temperature (time, lat, lon)
da = xr.open_dataset("ERA5_t2m_annual.nc")["t2m"]

# --- Method 1: Using xr.apply_ufunc (RECOMMENDED) ---
def mk_slope(arr):
    """Extract Sen's slope from MK test."""
    try:
        valid = arr[~np.isnan(arr)]
        if len(valid) < 10:
            return np.nan
        result = mk.hamed_rao_modification_test(valid)
        return result.slope
    except Exception:
        return np.nan

def mk_pvalue(arr):
    """Extract p-value from MK test."""
    try:
        valid = arr[~np.isnan(arr)]
        if len(valid) < 10:
            return np.nan
        result = mk.hamed_rao_modification_test(valid)
        return result.p
    except Exception:
        return np.nan

# Apply slope function
slope_map = xr.apply_ufunc(
    mk_slope,
    da,
    input_core_dims=[["time"]],
    vectorize=True,
    dask="parallelized",
    output_dtypes=[float],
)

# Apply p-value function
pval_map = xr.apply_ufunc(
    mk_pvalue,
    da,
    input_core_dims=[["time"]],
    vectorize=True,
    dask="parallelized",
    output_dtypes=[float],
)

# Compute (triggers Dask computation if chunked)
slope_map = slope_map.compute()
pval_map = pval_map.compute()

# Save results
slope_map.to_netcdf("mk_slope_map.nc")
pval_map.to_netcdf("mk_pvalue_map.nc")
```

### 4.2 Combined Slope + p-value (Efficient)

```python
def mk_both(arr):
    """Return slope and p-value together."""
    try:
        valid = arr[~np.isnan(arr)]
        if len(valid) < 10:
            return np.nan, np.nan
        result = mk.hamed_rao_modification_test(valid)
        return result.slope, result.p
    except:
        return np.nan, np.nan

# Apply with two outputs
slopes, pvalues = xr.apply_ufunc(
    mk_both,
    da,
    input_core_dims=[["time"]],
    output_core_dims=[[], []],
    vectorize=True,
    output_dtypes=[float, float],
)

slopes = slopes.compute()
pvalues = pvalues.compute()

# Significant trend mask (p < 0.05)
sig_mask = pvalues < 0.05
print(f"Fraction of grid points with significant trend: {float(sig_mask.mean()):.1%}")
```

---

## 📁 LEVEL 5 — Station Data Analysis

### 5.1 Single Station Analysis

```python
import pandas as pd
import pymannkendall as mk

# Load station data
df = pd.read_csv("islamabad_1950_2020.csv", parse_dates=["date"], index_col="date")

# Annual means
annual_tmax = df["tmax"].resample("YE").mean()
annual_tmin = df["tmin"].resample("YE").mean()
annual_precip = df["precip"].resample("YE").sum()

variables = {
    "Annual Tmax": annual_tmax,
    "Annual Tmin": annual_tmin,
    "Annual Precip": annual_precip,
}

# Run MK for each variable
results = {}
for name, series in variables.items():
    clean = series.dropna()
    result = mk.hamed_rao_modification_test(clean.values)
    results[name] = {
        "trend": result.trend,
        "p_value": result.p,
        "slope": result.slope,
        "slope_per_decade": result.slope * 10,
        "significant": result.h,
        "tau": result.Tau
    }

# Print results table
print(f"\n{'Variable':<20} {'Trend':<12} {'p-value':<10} {'Slope/decade':<15} {'Sig?'}")
print("-" * 65)
for name, r in results.items():
    sig = "✓" if r["significant"] else "✗"
    print(f"{name:<20} {r['trend']:<12} {r['p_value']:<10.4f} "
          f"{r['slope_per_decade']:<15.4f} {sig}")
```

### 5.2 Batch Analysis (Multiple Stations)

```python
import glob
import pandas as pd
import pymannkendall as mk

station_files = glob.glob("stations/*.csv")
all_results = []

for filepath in station_files:
    station_name = filepath.split("/")[-1].replace(".csv", "")
    df = pd.read_csv(filepath, parse_dates=["date"], index_col="date")
    annual = df["tmax"].resample("YE").mean().dropna()

    if len(annual) < 20:  # Need at least 20 years
        continue

    result = mk.hamed_rao_modification_test(annual.values)
    all_results.append({
        "station": station_name,
        "lat": df["lat"].iloc[0],
        "lon": df["lon"].iloc[0],
        "trend": result.trend,
        "p_value": result.p,
        "slope_per_year": result.slope,
        "slope_per_decade": result.slope * 10,
        "significant_05": result.h,
        "tau": result.Tau,
        "n_years": len(annual),
    })

# Create results DataFrame
results_df = pd.DataFrame(all_results)
results_df.to_csv("mk_results_all_stations.csv", index=False)
print(results_df.sort_values("slope_per_decade", ascending=False))
```

---

## 📁 LEVEL 6 — Visualizing MK Results

### 6.1 Trend Map with Stippling

```python
import matplotlib.pyplot as plt
import cartopy.crs as ccrs
import cartopy.feature as cfeature
import numpy as np

fig, ax = plt.subplots(
    figsize=(12, 8),
    subplot_kw={"projection": ccrs.LambertConformal(
        central_longitude=70, central_latitude=30)}
)
ax.set_extent([55, 85, 20, 45], crs=ccrs.PlateCarree())
ax.add_feature(cfeature.COASTLINE.with_scale("10m"), linewidth=0.6)
ax.add_feature(cfeature.BORDERS.with_scale("10m"), linewidth=0.5)

# Plot Sen's slope (°C/decade)
slope_decade = slope_map.values * 10
levels = np.linspace(-0.5, 0.5, 21)

cf = ax.contourf(
    slope_map.lon.values, slope_map.lat.values, slope_decade,
    transform=ccrs.PlateCarree(),
    levels=levels, cmap="RdBu_r", extend="both"
)

# Stipple significant grid points (p < 0.05)
lon2d, lat2d = np.meshgrid(slope_map.lon.values, slope_map.lat.values)
sig = pval_map.values < 0.05

ax.scatter(
    lon2d[sig][::2], lat2d[sig][::2],
    transform=ccrs.PlateCarree(),
    s=1, c="black", marker=".", alpha=0.5,
    label="p < 0.05"
)

cbar = plt.colorbar(cf, ax=ax, orientation="horizontal", pad=0.05, shrink=0.8)
cbar.set_label("Temperature Trend (°C/decade)", fontsize=11)
ax.set_title("Mann-Kendall Trend (Hamed-Rao)\n1979–2023, ERA5 Tmax",
             fontsize=13, fontweight="bold")
plt.savefig("mk_trend_map.png", dpi=300, bbox_inches="tight")
```

---

## 📁 LEVEL 7 — Interpreting Sen's Slope

```python
# Sen's slope = Median of all pairwise slopes
# It is the ROBUST ESTIMATE of the RATE OF CHANGE

# Example interpretation:
result = mk.hamed_rao_modification_test(annual_temp)

slope_per_year = result.slope           # °C per year
slope_per_decade = result.slope * 10    # °C per decade
slope_over_period = result.slope * 45   # °C over 45-year period (1979–2023)

print(f"Rate of warming: {slope_per_year:.4f} °C/year")
print(f"                = {slope_per_decade:.3f} °C/decade")
print(f"                = {slope_over_period:.2f} °C over the study period")

# Reconstruct trend line (for plotting)
years = np.arange(1979, 2024)
trend_line = result.slope * (years - years[0]) + result.intercept
```

---

## 📁 LEVEL 8 — Complete Workflow Template

```python
"""
Complete Mann-Kendall trend analysis workflow:
- ERA5 annual temperature trend map
- Significant trends stippled
- Station comparison
"""

import xarray as xr
import numpy as np
import pandas as pd
import pymannkendall as mk
import matplotlib.pyplot as plt
import cartopy.crs as ccrs
import cartopy.feature as cfeature
from dask.diagnostics import ProgressBar

# ---- 1. LOAD DATA ----
ds = xr.open_mfdataset("ERA5_t2m_*.nc", chunks={"time": 100})
da_annual = (ds["t2m"] - 273.15).resample(time="1Y").mean()

# ---- 2. MK TEST AT EVERY GRID POINT ----
def mk_results(arr):
    valid = arr[~np.isnan(arr)]
    if len(valid) < 10:
        return np.nan, np.nan, np.nan
    try:
        r = mk.hamed_rao_modification_test(valid)
        return r.slope, r.p, r.Tau
    except:
        return np.nan, np.nan, np.nan

def get_slope(arr): return mk_results(arr)[0]
def get_pval(arr): return mk_results(arr)[1]

kw = dict(input_core_dims=[["time"]], vectorize=True,
          dask="parallelized", output_dtypes=[float])

slope_map = xr.apply_ufunc(get_slope, da_annual, **kw)
pval_map  = xr.apply_ufunc(get_pval, da_annual, **kw)

print("Computing trend maps...")
with ProgressBar():
    slope_map = slope_map.compute()
    pval_map  = pval_map.compute()

# ---- 3. SAVE RESULTS ----
xr.Dataset({"slope": slope_map, "pvalue": pval_map}).to_netcdf("mk_ERA5_trend.nc")

# ---- 4. PLOT ----
fig, ax = plt.subplots(
    subplot_kw={"projection": ccrs.Robinson()}, figsize=(14, 7))
ax.set_global()
ax.add_feature(cfeature.COASTLINE, linewidth=0.5)

cf = ax.contourf(
    slope_map.lon, slope_map.lat, slope_map.values * 10,
    transform=ccrs.PlateCarree(),
    levels=np.linspace(-0.8, 0.8, 33), cmap="RdBu_r", extend="both"
)

# Stippling
lon2d, lat2d = np.meshgrid(slope_map.lon, slope_map.lat)
sig = pval_map.values < 0.05
ax.scatter(lon2d[sig][::5], lat2d[sig][::5], transform=ccrs.PlateCarree(),
           s=0.5, c="k", alpha=0.5)

plt.colorbar(cf, ax=ax, orientation="horizontal", pad=0.04, shrink=0.6,
             label="Temperature Trend (°C/decade)")
ax.set_title("ERA5 Temperature Trends (1979–2023)\nMann-Kendall (Hamed-Rao), p<0.05 stippled",
             fontsize=12)
plt.savefig("ERA5_MK_trend_global.png", dpi=300, bbox_inches="tight")
print("Done!")
```

---

## 📋 pymannkendall Quick Reference

| Function | Use Case |
|----------|---------|
| `mk.original_test(x)` | Basic test, no autocorrelation |
| `mk.hamed_rao_modification_test(x)` | ⭐ RECOMMENDED — handles autocorrelation |
| `mk.seasonal_test(x, period=12)` | Monthly time series |
| `mk.yue_wang_modification_test(x)` | Autocorrelation + trend in variance |
| `mk.regional_test(X)` | Multiple stations (2D array) |
| `mk.partial_tau_test(x, covariate)` | Remove covariate influence |

**Result attributes:**
| Attribute | Meaning |
|-----------|---------|
| `.trend` | "increasing" / "decreasing" / "no trend" |
| `.h` | True if significant at alpha level |
| `.p` | p-value |
| `.z` | Normalized z-statistic |
| `.Tau` | Kendall's τ (-1 to +1) |
| `.slope` | Sen's slope (per time step) |
| `.intercept` | Sen's intercept |

---

## 📚 Resources

- [pymannkendall GitHub](https://github.com/mmhs013/pyMannKendall)
- [pymannkendall Documentation](https://pypi.org/project/pymannkendall/)
- Mann, H.B. (1945) — Original paper
- Sen, P.K. (1968) — Sen's slope estimator
- Hamed & Rao (1998) — Autocorrelation correction
