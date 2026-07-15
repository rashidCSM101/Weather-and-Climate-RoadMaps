# 🌡️ xclim — Complete Learning Roadmap
## Climate Indices, Bias Correction & Ensemble Analysis

---

## 📚 What is xclim?

xclim is a Python library built on xarray, developed specifically for **climate science workflows**. It provides:
- **500+ climate indices** (ETCCDI, Climdex standards) — just one line of code each
- **Bias correction methods** — making raw climate model output usable
- **Ensemble analysis** — handling multi-model/multi-run ensembles
- **Statistical testing** — built-in significance tests
- **CF-compliant output** — automatically adds correct attributes/units

---

## 🗺️ Roadmap Overview

```
Level 1: Installation & Setup    → Requirements, CF conventions
Level 2: Climate Indices         → Temperature, Precipitation, Wind indices
Level 3: ETCCDI Extreme Indices  → Standard climate extreme indices
Level 4: Index Calculations      → From raw ERA5/CMIP data
Level 5: Bias Correction         → Quantile mapping, delta method, DQM
Level 6: Ensemble Analysis       → Multi-model spreads, percentiles
Level 7: Statistical Testing     → Change detection, significance
Level 8: Complete Workflow       → End-to-end CMIP projection workflow
```

---

## 📁 LEVEL 1 — Setup & Requirements

### 1.1 Installation

```bash
pip install xclim
```

### 1.2 CF Convention Requirements

xclim REQUIRES data to follow Climate and Forecast (CF) conventions:

```python
import xarray as xr
import xclim

# Data MUST have:
# 1. Standard units attribute (e.g., "K" not "Kelvin" or "kelvin")
# 2. Correct cell_methods (added automatically by ERA5/CMIP)
# 3. Coordinate names: "time", "lat"/"latitude", "lon"/"longitude"

# Load ERA5 data
ds = xr.open_dataset("ERA5_t2m_daily.nc")
print(ds["t2m"].attrs["units"])   # Must be "K"

# Fix units if needed (xclim is strict about units)
ds["t2m"].attrs["units"] = "K"                    # If missing
ds["t2m"].attrs["cell_methods"] = "time: mean"    # Time averaging method

# Convert Kelvin to Celsius
ds["tasmax"] = ds["t2m"] - 273.15
ds["tasmax"].attrs["units"] = "degC"             # xclim accepts "degC"
```

### 1.3 Variable Naming

```python
# xclim uses CMIP6 standard variable names:
# tas    = Near-surface air temperature (daily mean, K or °C)
# tasmax = Near-surface maximum temperature (daily max, K or °C)  
# tasmin = Near-surface minimum temperature (daily min, K or °C)
# pr     = Precipitation (kg m-2 s-1 or mm/day)
# prsn   = Snowfall precipitation flux
# sfcWind = Near-surface wind speed (m/s)
# hurs   = Near-surface relative humidity (%)
# rsds   = Surface downwelling shortwave radiation (W m-2)

# Rename ERA5 variables to CMIP6 names
ds = ds.rename({
    "t2m": "tas",
    "mx2t": "tasmax",
    "mn2t": "tasmin",
    "tp": "pr",
})

# Fix precipitation units (ERA5 is m/day, need mm/day or kg m-2 s-1)
ds["pr"] = ds["pr"] * 1000          # m → mm
ds["pr"].attrs["units"] = "mm d-1"  # Or "kg m-2 s-1"
```

---

## 📁 LEVEL 2 — Climate Indices

### 2.1 Temperature Indices

```python
import xclim
from xclim.indices import *
import xarray as xr

ds = xr.open_dataset("ERA5_daily.nc")
tas    = ds["tas"]     # Daily mean temperature
tasmax = ds["tasmax"]  # Daily maximum temperature
tasmin = ds["tasmin"]  # Daily minimum temperature

# Make sure units are set
tasmax.attrs["units"] = "degC"
tasmin.attrs["units"] = "degC"

# --- HEAT-RELATED INDICES ---

# TX90p: Days with Tmax > 90th percentile
# First compute percentile threshold
tx90 = xclim.indices.tx90p(tasmax=tasmax, t90=None)  # Auto-compute threshold

# Hot days: Days with Tmax > threshold
hot_days = xclim.indices.tx_days_above(
    tasmax=tasmax,
    thresh="35 degC",   # Threshold with units
    freq="YS"           # Annual frequency
)
print(hot_days)

# Tropical Nights: Days where Tmin > 20°C
tropical_nights = xclim.indices.tropical_nights(
    tasmin=tasmin,
    thresh="20 degC",
    freq="YS"
)

# Heat Wave Duration Index (HWDI)
# Consecutive days where Tmax > 90th percentile
hwdi = xclim.indices.heat_wave_max_length(
    tasmax=tasmax,
    tasmin=tasmin,
    thresh_tasmax="36 degC",
    thresh_tasmin="20 degC",
    window=3,           # At least 3 consecutive days
    freq="YS"
)

# Growing Degree Days
gdd = xclim.indices.growing_degree_days(
    tas=tas,
    thresh="10 degC",   # Base temperature
    freq="YS"
)

# --- COLD-RELATED INDICES ---

# Frost Days: Days with Tmin < 0°C
frost = xclim.indices.frost_days(tasmin=tasmin, freq="YS")

# Ice Days: Days with Tmax < 0°C
ice_days = xclim.indices.ice_days(tasmax=tasmax, freq="YS")

# Cold Spell Duration Index
csdi = xclim.indices.cold_spell_duration_index(
    tasmin=tasmin,
    thresh_tasmin=None,  # Uses 10th percentile
    window=6,
    freq="YS"
)

# Diurnal Temperature Range
dtr = xclim.indices.daily_temperature_range(
    tasmax=tasmax,
    tasmin=tasmin,
    freq="MS"   # Monthly
)
```

### 2.2 Precipitation Indices

```python
pr = ds["pr"]
pr.attrs["units"] = "mm d-1"

# --- WET/DRY DAY INDICES ---

# Wet Days: Days with precip >= 1mm
wet_days = xclim.indices.wetdays(
    pr=pr,
    thresh="1 mm/day",
    freq="YS"
)

# Maximum consecutive dry days (CDD — Consecutive Dry Days)
cdd = xclim.indices.maximum_consecutive_dry_days(
    pr=pr,
    thresh="1 mm/day",
    freq="YS"
)

# Maximum consecutive wet days (CWD)
cwd = xclim.indices.maximum_consecutive_wet_days(
    pr=pr,
    thresh="1 mm/day",
    freq="YS"
)

# --- PRECIPITATION AMOUNTS ---

# Annual total precipitation on wet days (PRCPTOT)
prcptot = xclim.indices.wet_precip_accumulation(
    pr=pr,
    thresh="1 mm/day",
    freq="YS"
)

# Very Wet Days: Precip > 95th percentile (R95p)
r95p = xclim.indices.r95p(pr=pr, freq="YS")

# Extremely Wet Days: Precip > 99th percentile (R99p)
r99p = xclim.indices.r99p(pr=pr, freq="YS")

# Max 1-day precipitation (RX1day)
rx1day = xclim.indices.max_1day_precipitation_amount(pr=pr, freq="YS")

# Max 5-day precipitation (RX5day)
rx5day = xclim.indices.max_n_day_precipitation_amount(
    pr=pr, window=5, freq="YS"
)

# Simple Precipitation Intensity Index (SDII)
sdii = xclim.indices.sdii(
    pr=pr,
    thresh="1 mm/day",
    freq="YS"
)

# Heavy Precipitation Days (R10mm, R20mm)
r10mm = xclim.indices.days_over_precip_thresh(
    pr=pr,
    thresh="10 mm/day",
    freq="YS"
)
r20mm = xclim.indices.days_over_precip_thresh(
    pr=pr,
    thresh="20 mm/day",
    freq="YS"
)
```

### 2.3 Frequency Control

```python
# freq parameter controls output time resolution:
# "YS"  = Annual (Year Start)
# "MS"  = Monthly (Month Start)
# "QS-DEC" = Seasonal (DJF, MAM, JJA, SON)
# "D"   = Daily (rare for indices)

# Monthly hot days
hot_days_monthly = xclim.indices.tx_days_above(
    tasmax=tasmax,
    thresh="35 degC",
    freq="MS"   # Monthly — gives 12 values per year
)

# Seasonal precipitation
rx5day_seasonal = xclim.indices.max_n_day_precipitation_amount(
    pr=pr, window=5, freq="QS-DEC"
)
```

---

## 📁 LEVEL 3 — ETCCDI Extreme Indices

### 3.1 All 27 ETCCDI Indices

```python
# The Expert Team on Climate Change Detection and Indices (ETCCDI)
# defines 27 standard extreme climate indices used globally

# Temperature-based ETCCDI indices:
tn10p = xclim.indices.tn10p(tasmin=tasmin)     # Cold nights (<10th pctl)
tn90p = xclim.indices.tn90p(tasmin=tasmin)     # Warm nights (>90th pctl)
tx10p = xclim.indices.tx10p(tasmax=tasmax)     # Cool days (<10th pctl)
tx90p = xclim.indices.tx90p(tasmax=tasmax)     # Hot days (>90th pctl)

txx = xclim.indices.txx(tasmax=tasmax, freq="YS")  # Annual max of Tmax
txn = xclim.indices.txn(tasmax=tasmax, freq="YS")  # Annual min of Tmax
tnx = xclim.indices.tnx(tasmin=tasmin, freq="YS")  # Annual max of Tmin
tnn = xclim.indices.tnn(tasmin=tasmin, freq="YS")  # Annual min of Tmin

gsl = xclim.indices.growing_season_length(tas=tas, freq="YS")

# Precipitation-based ETCCDI indices:
rx1day  = xclim.indices.max_1day_precipitation_amount(pr=pr, freq="YS")
sdii    = xclim.indices.sdii(pr=pr, freq="YS")
r10mm   = xclim.indices.days_over_precip_thresh(pr=pr, thresh="10 mm/d", freq="YS")
r20mm   = xclim.indices.days_over_precip_thresh(pr=pr, thresh="20 mm/d", freq="YS")
cdd     = xclim.indices.maximum_consecutive_dry_days(pr=pr, freq="YS")
cwd     = xclim.indices.maximum_consecutive_wet_days(pr=pr, freq="YS")
r95p    = xclim.indices.r95p(pr=pr, freq="YS")
r99p    = xclim.indices.r99p(pr=pr, freq="YS")
prcptot = xclim.indices.wet_precip_accumulation(pr=pr, freq="YS")
```

### 3.2 Using Indicators (High-Level API)

```python
# xclim also has an "Indicators" API that auto-checks units and CF compliance

# Build indicator from index
from xclim.core.indicator import build_indicator_cls

# Or use pre-built indicators
hot_days_indicator = xclim.atmos.tx_days_above

result = hot_days_indicator(
    tasmax=tasmax,
    thresh="35 degC",
    freq="YS"
)
# Automatically adds CF-compliant attributes to output
print(result.attrs)   # Standard name, long name, units all filled in
```

---

## 📁 LEVEL 4 — Batch Index Calculation

### 4.1 Calculate All Indices at Once

```python
def calculate_all_indices(ds, freq="YS"):
    """Calculate a standard set of climate indices from daily data."""
    tasmax = ds["tasmax"]
    tasmin = ds["tasmin"]
    tas    = ds["tas"]
    pr     = ds["pr"]

    indices = {}

    # Temperature
    indices["TXx"]  = xclim.indices.txx(tasmax=tasmax, freq=freq)
    indices["TNn"]  = xclim.indices.tnn(tasmin=tasmin, freq=freq)
    indices["HD35"] = xclim.indices.tx_days_above(tasmax=tasmax, thresh="35 degC", freq=freq)
    indices["FD"]   = xclim.indices.frost_days(tasmin=tasmin, freq=freq)
    indices["CDD_T"]= xclim.indices.maximum_consecutive_dry_days(pr=pr, freq=freq)
    indices["DTR"]  = xclim.indices.daily_temperature_range(tasmax=tasmax, tasmin=tasmin, freq=freq)

    # Precipitation
    indices["Rx1day"] = xclim.indices.max_1day_precipitation_amount(pr=pr, freq=freq)
    indices["Rx5day"] = xclim.indices.max_n_day_precipitation_amount(pr=pr, window=5, freq=freq)
    indices["SDII"]   = xclim.indices.sdii(pr=pr, freq=freq)
    indices["R10mm"]  = xclim.indices.days_over_precip_thresh(pr=pr, thresh="10 mm/d", freq=freq)
    indices["CDD"]    = xclim.indices.maximum_consecutive_dry_days(pr=pr, freq=freq)
    indices["CWD"]    = xclim.indices.maximum_consecutive_wet_days(pr=pr, freq=freq)
    indices["PRCPTOT"]= xclim.indices.wet_precip_accumulation(pr=pr, freq=freq)

    return xr.Dataset(indices)

# Calculate and save
indices_ds = calculate_all_indices(ds, freq="YS")
indices_ds.to_netcdf("climate_indices.nc")
```

---

## 📁 LEVEL 5 — Bias Correction

### 5.1 Why Bias Correction?

```
Raw GCM/RCM output has SYSTEMATIC ERRORS compared to observations.
Example: ERA5 shows Tmax = 38°C in Pakistan
         CMIP6 model shows Tmax = 32°C (6°C cold bias)
         
After bias correction: CMIP6 adjusted to match ERA5 statistics
→ Makes model output usable for impact studies
```

### 5.2 Quantile Mapping (Standard Method)

```python
from xclim.sdba import adjustment

# Setup:
# - obs: Observed reference (ERA5 or station) — "training period"
# - sim: Model output (historical + future)
# - hist: Model historical period (same as obs period)
# - scen: Model future period to be corrected

# Load data
obs  = xr.open_dataset("ERA5_1980_2014.nc")["tasmax"]   # Observation
hist = xr.open_dataset("CMIP6_hist_1980_2014.nc")["tasmax"]  # Model historical
scen = xr.open_dataset("CMIP6_ssp585_2015_2100.nc")["tasmax"]  # Model future

# Set units
for da in [obs, hist, scen]:
    da.attrs["units"] = "degC"

# --- Quantile Mapping ---
QM = adjustment.QuantileDeltaMapping.train(
    ref=obs,            # Reference (observations)
    hist=hist,          # Model historical
    nquantiles=100,     # Number of quantiles
    kind="+",           # Additive bias ("+" for T, "*" for P)
    group="time.month", # Monthly grouping (account for seasonality)
)

# Apply correction to future scenario
scen_corrected = QM.adjust(scen, extrapolation="constant")

print("Original future mean:", float(scen.mean()))
print("Corrected future mean:", float(scen_corrected.mean()))
```

### 5.3 Other Bias Correction Methods

```python
# --- Delta Additive Method (simple, good for temperature) ---
DM = adjustment.Scaling.train(
    ref=obs, hist=hist,
    kind="+",
    group="time.month"
)
scen_delta = DM.adjust(scen)

# --- LOCI (Local Intensity Scaling for precipitation) ---
LOCI = adjustment.LOCI.train(
    ref=obs, hist=hist,
    thresh="1 mm/d",   # Wet day threshold
    group="time.month"
)
pr_corrected = LOCI.adjust(pr_scen)

# --- Equidistant Quantile Matching ---
EDQM = adjustment.EQM.train(
    ref=obs, hist=hist,
    nquantiles=50,
    kind="+"
)
scen_edqm = EDQM.adjust(scen)
```

---

## 📁 LEVEL 6 — Ensemble Analysis

### 6.1 Multi-Model Ensemble

```python
from xclim.ensemble import create_ensemble, ensemble_percentiles

# Create ensemble from multiple models
models = {
    "ACCESS": xr.open_dataset("ACCESS_tasmax.nc")["tasmax"],
    "MPI-ESM": xr.open_dataset("MPI_tasmax.nc")["tasmax"],
    "CanESM5": xr.open_dataset("CanESM5_tasmax.nc")["tasmax"],
    "IPSL": xr.open_dataset("IPSL_tasmax.nc")["tasmax"],
}

# Align to common grid if needed (regrid to coarsest)
# Then stack:
ens = xr.concat(list(models.values()), dim="model")
ens["model"] = list(models.keys())

# Ensemble statistics
ens_mean = ens.mean("model")
ens_std  = ens.std("model")

# Ensemble percentiles
ens_p10 = ens.quantile(0.10, dim="model")   # 10th percentile (low end)
ens_p50 = ens.quantile(0.50, dim="model")   # Median
ens_p90 = ens.quantile(0.90, dim="model")   # 90th percentile (high end)

# Model agreement (% of models agreeing on sign of change)
change = ens - ens.sel(time=slice("1980","2010")).mean("time")
agreement = (change > 0).mean("model") > 0.8   # >80% models agree
```

### 6.2 xclim Ensemble Tools

```python
from xclim import ensembles

# Create ensemble Dataset
ens_ds = ensembles.create_ensemble(
    [path1, path2, path3, path4],  # List of NetCDF paths
    mf_flag=False
)

# Ensemble percentiles
perc_ds = ensembles.ensemble_percentiles(
    ens_ds,
    values=[10, 25, 50, 75, 90],
    split=False
)

# Ensemble robustness (agreement)
robustness = ensembles.change_significance(
    fut=ens_future,
    ref=ens_historical,
    sig_vals=[0.8, 0.9]
)
```

---

## 📋 xclim Quick Reference

| Index | Code | Units |
|-------|------|-------|
| Hot days (Tmax > 35°C) | `xclim.indices.tx_days_above(tasmax, thresh="35 degC")` | days/year |
| Frost days | `xclim.indices.frost_days(tasmin)` | days/year |
| Max annual Tmax | `xclim.indices.txx(tasmax)` | °C |
| Rx1day | `xclim.indices.max_1day_precipitation_amount(pr)` | mm |
| CDD (dry spell) | `xclim.indices.maximum_consecutive_dry_days(pr)` | days |
| SDII | `xclim.indices.sdii(pr)` | mm/wet day |
| GDD | `xclim.indices.growing_degree_days(tas, thresh="10 degC")` | °C·days |

---

## 📚 Resources

- [xclim Documentation](https://xclim.readthedocs.io)
- [xclim GitHub](https://github.com/Ouranosinc/xclim)
- [ETCCDI Indices](https://www.wcrp-climate.org/etccdi)
- [Bias Correction Guide (xclim)](https://xclim.readthedocs.io/en/stable/sdba.html)
