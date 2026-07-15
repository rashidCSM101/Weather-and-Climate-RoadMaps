# 🐼 pandas — Complete Learning Roadmap
## Station & Tabular Climate Data Analysis

---

## 📚 What is pandas?

pandas is the go-to Python library for **tabular data analysis** — data that fits in a table (rows and columns). In climate science, this means:
- **Station data**: daily/monthly records from weather stations (temperature, rainfall, humidity)
- **Observational records**: WMO station files, PMD CSV data
- **Time series analysis**: trend analysis, anomaly detection, seasonal decomposition
- **Data cleaning**: handling missing values, duplicates, outliers in station records

---

## 🗺️ Roadmap Overview

```
Level 1: Foundations          → DataFrame/Series, I/O, data types
Level 2: Indexing & Selection → loc/iloc, boolean indexing, MultiIndex
Level 3: Time Series          → DatetimeIndex, resampling, shifting
Level 4: Data Cleaning        → Missing values, outliers, quality control
Level 5: Groupby & Aggregation → Climatologies, seasonal summaries
Level 6: Merging & Joining    → Combining station datasets
Level 7: Statistical Analysis → Correlation, regression, trends
Level 8: Climate Applications → Real station data workflows
```

---

## 📁 LEVEL 1 — Foundations

### 1.1 Core Data Structures

```python
import pandas as pd
import numpy as np

# --- Series: 1D labeled array ---
temp = pd.Series(
    [15.2, 18.5, 22.1, 28.3, 33.4],
    index=pd.date_range("2020-01-01", periods=5, freq="MS"),
    name="temperature_C"
)
print(temp)
print(temp.dtype)      # float64
print(temp.index)      # DatetimeIndex
print(temp.name)       # 'temperature_C'

# --- DataFrame: 2D labeled table ---
df = pd.DataFrame({
    "date": pd.date_range("2020-01-01", periods=365, freq="D"),
    "station_id": "Islamabad",
    "tmax": np.random.normal(25, 5, 365),
    "tmin": np.random.normal(15, 5, 365),
    "precip": np.random.exponential(2, 365),
})
print(df.head())       # First 5 rows
print(df.tail())       # Last 5 rows
print(df.info())       # Column types, non-null counts
print(df.describe())   # Statistical summary
print(df.shape)        # (365, 5)
print(df.dtypes)       # Data types per column
print(df.columns)      # Column names
```

### 1.2 Reading Climate Data Files

```python
# --- CSV (most common station format) ---
df = pd.read_csv(
    "islamabad_station.csv",
    parse_dates=["date"],              # Auto-detect date columns
    index_col="date",                  # Set date as index
    na_values=["-999", "-9999", "M"],  # Common missing value codes
    dtype={"tmax": float, "tmin": float, "precip": float}
)

# --- Excel files (PMD data) ---
df = pd.read_excel(
    "PMD_station_data.xlsx",
    sheet_name="Islamabad",
    header=0,
    parse_dates=True,
    index_col=0
)

# --- Fixed-width format (GHCN-D, synoptic data) ---
df = pd.read_fwf(
    "station_data.txt",
    widths=[11, 4, 2, 2, 6, 6, 6, 6],
    names=["station", "year", "month", "day", "tmax", "tmin", "precip", "wind"],
    na_values=["-9999"]
)

# --- From multiple CSVs (multiple stations) ---
import glob
all_files = glob.glob("stations/*.csv")
dfs = [pd.read_csv(f, parse_dates=["date"], index_col="date") for f in all_files]
df_all = pd.concat(dfs, keys=[f.split("/")[-1] for f in all_files])  # MultiIndex

# --- Tab-separated values ---
df = pd.read_csv("data.txt", sep="\t", parse_dates=["date"])
```

### 1.3 Data Types & Memory Optimization

```python
# Convert data types
df["year"] = df["year"].astype("int16")
df["month"] = df["month"].astype("int8")
df["station"] = df["station"].astype("category")  # Saves memory for repeated strings
df["date"] = pd.to_datetime(df["date"])

# Check memory usage
print(df.memory_usage(deep=True))   # Per column
print(df.memory_usage(deep=True).sum() / 1024**2, "MB")
```

---

## 📁 LEVEL 2 — Indexing & Selection

### 2.1 loc & iloc

```python
# --- loc: label-based ---
df.loc["2020-01-01"]                    # Single date
df.loc["2020-01-01":"2020-12-31"]       # Date range
df.loc["2020", "tmax"]                  # Year + column
df.loc[df["tmax"] > 40]                 # Boolean condition

# --- iloc: integer-based ---
df.iloc[0]                              # First row
df.iloc[-1]                             # Last row
df.iloc[0:10]                           # First 10 rows
df.iloc[0:10, 0:3]                      # First 10 rows, 3 columns
df.iloc[[0, 5, 10], :]                  # Specific row indices

# --- Selecting columns ---
df["tmax"]                              # Single column → Series
df[["tmax", "tmin"]]                    # Multiple columns → DataFrame
df.tmax                                 # Dot notation (only simple names)

# --- Boolean indexing ---
df[df["tmax"] > 45]                     # Heat wave days
df[(df["month"] == 7) & (df["tmax"] > 40)]  # July days above 40°C
df[df["precip"].isna()]                 # Missing precipitation
```

### 2.2 DatetimeIndex Operations

```python
# Set DatetimeIndex
df.index = pd.to_datetime(df.index)

# Access date components
df.index.year
df.index.month
df.index.day
df.index.dayofyear
df.index.weekday
df.index.quarter
df.index.season      # Not built-in, see below

# Select by date
df["2020"]                             # Full year
df["2020-06"]                          # Full month
df["2020-06":"2020-08"]               # Summer months

# Add date component columns
df["year"] = df.index.year
df["month"] = df.index.month
df["season"] = df.index.month.map({
    12: "DJF", 1: "DJF", 2: "DJF",
    3: "MAM", 4: "MAM", 5: "MAM",
    6: "JJA", 7: "JJA", 8: "JJA",
    9: "SON", 10: "SON", 11: "SON"
})
```

### 2.3 MultiIndex (Multiple Stations)

```python
# Create MultiIndex DataFrame
df_multi = pd.DataFrame({
    "tmax": [35, 40, 38, 32, 37, 41],
    "precip": [0, 5, 12, 0, 3, 8]
}, index=pd.MultiIndex.from_tuples([
    ("Islamabad", "2020-01"), ("Islamabad", "2020-02"),
    ("Karachi", "2020-01"), ("Karachi", "2020-02"),
    ("Lahore", "2020-01"), ("Lahore", "2020-02"),
], names=["station", "date"]))

# Access levels
df_multi.loc["Islamabad"]              # All Islamabad data
df_multi.loc[("Islamabad", "2020-01")] # Single record
df_multi.xs("Islamabad", level="station")  # Cross-section

# Unstack to wide format
df_wide = df_multi["tmax"].unstack("station")  # Stations as columns
print(df_wide.head())

# Stack back to long format
df_long = df_wide.stack()
```

---

## 📁 LEVEL 3 — Time Series Operations

### 3.1 Resampling

```python
# --- Daily → Monthly ---
monthly_mean = df["tmax"].resample("MS").mean()    # Month start
monthly_sum = df["precip"].resample("ME").sum()    # Total monthly precip

# --- Daily → Seasonal ---
seasonal = df["tmax"].resample("QS-DEC").mean()    # DJF, MAM, JJA, SON

# --- Daily → Annual ---
annual = df["tmax"].resample("YE").mean()           # Annual mean
annual_max = df["tmax"].resample("YE").max()        # Annual maximum
annual_precip = df["precip"].resample("YE").sum()   # Annual total precip

# --- Monthly → Daily (upsampling) ---
daily_from_monthly = monthly_mean.resample("D").ffill()   # Forward fill
daily_interp = monthly_mean.resample("D").interpolate("linear")

# --- Custom aggregation ---
annual_stats = df["tmax"].resample("YE").agg({
    "mean": "mean",
    "max": "max",
    "min": "min",
    "std": "std"
})
```

### 3.2 Rolling Windows

```python
# 30-day moving average of temperature
df["tmax_30d"] = df["tmax"].rolling(window=30, center=True).mean()

# 90-day rolling sum (seasonal rainfall)
df["precip_90d"] = df["precip"].rolling(window=90, min_periods=60).sum()

# Rolling standard deviation (variability)
df["tmax_std"] = df["tmax"].rolling(window=365).std()

# Exponential weighted moving average (recent values weighted more)
df["tmax_ewm"] = df["tmax"].ewm(span=30).mean()
```

### 3.3 Shifting & Lagging

```python
# Lag/Lead analysis
df["tmax_prev"] = df["tmax"].shift(1)      # Previous day
df["tmax_next"] = df["tmax"].shift(-1)     # Next day
df["tmax_prev_year"] = df["tmax"].shift(365)  # Previous year

# Differencing
df["temp_change"] = df["tmax"].diff(1)     # Day-to-day change
df["annual_diff"] = df["tmax"].diff(365)   # Year-over-year change
df["precip_pct_change"] = df["precip"].pct_change()  # % change

# Autocorrelation
print(df["tmax"].autocorr(lag=1))          # Lag-1 autocorrelation
print(df["tmax"].autocorr(lag=365))        # Annual autocorrelation
```

---

## 📁 LEVEL 4 — Data Quality Control (Climate-Specific)

### 4.1 Missing Value Detection

```python
# Check missing values
print(df.isna().sum())              # Count NaN per column
print(df.isna().mean() * 100)       # % missing per column

# Visualize missing data pattern
import matplotlib.pyplot as plt
df.isna().plot(figsize=(12, 4), title="Missing Data Pattern")
plt.show()

# Percentage completeness by year/month
missing_by_year = df["tmax"].isna().groupby(df.index.year).mean() * 100
print(missing_by_year)
```

### 4.2 Missing Value Handling

```python
# Simple imputation
df["tmax"].fillna(df["tmax"].mean())       # Fill with mean (bad for climate!)
df["tmax"].fillna(method="ffill")          # Forward fill (use with caution)
df["tmax"].fillna(method="bfill")          # Backward fill

# Monthly climatology fill (recommended for climate data)
monthly_clim = df["tmax"].groupby(df.index.month).transform("mean")
df["tmax_filled"] = df["tmax"].fillna(monthly_clim)

# Interpolation for short gaps
df["tmax_interp"] = df["tmax"].interpolate(
    method="time",            # Time-aware interpolation
    limit=7,                  # Only fill gaps ≤ 7 days
    limit_direction="both"
)

# Drop rows with too many missing values
df_clean = df.dropna(subset=["tmax", "tmin"])          # Drop if both missing
df_clean = df.dropna(thresh=3)                          # Keep rows with ≥3 non-NaN
```

### 4.3 Outlier Detection (Quality Control)

```python
# --- Range check (physical limits) ---
df["tmax_qc"] = df["tmax"].copy()
df.loc[df["tmax"] > 60, "tmax_qc"] = np.nan    # >60°C impossible
df.loc[df["tmax"] < -20, "tmax_qc"] = np.nan   # <-20°C for Islamabad
df.loc[df["precip"] < 0, "precip"] = np.nan    # Negative precip

# --- Internal consistency check ---
df.loc[df["tmin"] > df["tmax"], ["tmin", "tmax"]] = np.nan  # tmin > tmax = error

# --- Z-score outlier detection (per month to account for seasonality) ---
def zscore_qc(series, threshold=3.5):
    monthly_mean = series.groupby(series.index.month).transform("mean")
    monthly_std = series.groupby(series.index.month).transform("std")
    zscore = (series - monthly_mean) / monthly_std
    return series.where(abs(zscore) < threshold)

df["tmax_qc"] = zscore_qc(df["tmax"])

# --- IQR method ---
Q1 = df["tmax"].quantile(0.25)
Q3 = df["tmax"].quantile(0.75)
IQR = Q3 - Q1
df["tmax_clean"] = df["tmax"].where(
    (df["tmax"] >= Q1 - 3 * IQR) & (df["tmax"] <= Q3 + 3 * IQR)
)
```

---

## 📁 LEVEL 5 — GroupBy & Aggregation

### 5.1 Seasonal & Monthly Climatology

```python
# --- Monthly climatology ---
monthly_clim = df.groupby(df.index.month)["tmax"].mean()
monthly_clim.index = ["Jan","Feb","Mar","Apr","May","Jun",
                       "Jul","Aug","Sep","Oct","Nov","Dec"]
print(monthly_clim)

# --- Seasonal statistics ---
df["season"] = df.index.month.map({
    12:"DJF", 1:"DJF", 2:"DJF",
    3:"MAM", 4:"MAM", 5:"MAM",
    6:"JJA", 7:"JJA", 8:"JJA",
    9:"SON", 10:"SON", 11:"SON"
})
seasonal_stats = df.groupby("season").agg({
    "tmax": ["mean", "max", "std"],
    "precip": ["mean", "sum", "max"],
})
print(seasonal_stats)

# --- Annual statistics by multiple criteria ---
df["year"] = df.index.year
annual = df.groupby("year").agg(
    tmax_mean=("tmax", "mean"),
    tmax_max=("tmax", "max"),
    tmin_mean=("tmin", "mean"),
    precip_total=("precip", "sum"),
    rainy_days=("precip", lambda x: (x >= 1).sum()),   # Days with ≥1mm
    frost_days=("tmin", lambda x: (x < 0).sum()),      # Days below 0°C
    heat_days=("tmax", lambda x: (x > 40).sum()),      # Heat stress days
).reset_index()
```

### 5.2 Percentile & Extreme Analysis

```python
# Annual extremes
annual_max_temp = df["tmax"].resample("YE").max()
annual_min_temp = df["tmin"].resample("YE").min()
annual_max_rain = df["precip"].resample("YE").max()   # Wettest day

# Percentile-based thresholds
p90 = df["tmax"].groupby(df.index.month).transform(lambda x: x.quantile(0.9))
heat_wave = df["tmax"] > p90                          # Above 90th percentile

# Count of extreme events per year
annual_hw = heat_wave.resample("YE").sum()
print("Heat wave days per year:")
print(annual_hw)
```

---

## 📁 LEVEL 6 — Merging & Reshaping

### 6.1 Merging Datasets

```python
# --- Merge two station datasets ---
df_merged = pd.merge(
    df_islamabad, df_karachi,
    on="date",
    how="outer",                     # Keep all dates from both
    suffixes=("_ISB", "_KHI")
)

# --- Join on index ---
df_joined = df1.join(df2, how="outer")

# --- Concatenate multiple stations ---
all_stations = {
    "Islamabad": df_isb,
    "Karachi": df_khi,
    "Lahore": df_lhr,
    "Peshawar": df_psh,
}
df_all = pd.concat(all_stations, names=["station", "date"])

# --- Wide to Long (melt) ---
df_wide = pd.DataFrame({
    "date": pd.date_range("2020", periods=12, freq="MS"),
    "Jan": [10]*12, "Feb": [12]*12,
})
df_long = df_wide.melt(id_vars="date", var_name="month", value_name="temp")

# --- Long to Wide (pivot) ---
df_wide = df_long.pivot_table(
    index="date", columns="station", values="tmax", aggfunc="mean"
)
```

---

## 📁 LEVEL 7 — Statistical Analysis

### 7.1 Correlation Analysis

```python
# Pearson correlation matrix
corr_matrix = df[["tmax", "tmin", "precip", "humidity"]].corr()

# Spearman (rank-based, better for non-normal climate data)
spearman_corr = df[["tmax", "precip"]].corr(method="spearman")

# Rolling correlation (teleconnection analysis)
rolling_corr = df["tmax"].rolling(365).corr(df["nino34_index"])

# Lag correlation (ENSO teleconnection)
lags = range(-24, 25)
lag_corr = [df["tmax"].corr(df["nino34"].shift(lag)) for lag in lags]
```

### 7.2 Linear Trend Analysis

```python
from scipy import stats

# Per column trend
def linear_trend(series):
    series_clean = series.dropna()
    x = np.arange(len(series_clean))
    slope, intercept, r, p, se = stats.linregress(x, series_clean)
    return pd.Series({
        "slope": slope,
        "intercept": intercept,
        "r_squared": r**2,
        "p_value": p,
        "std_err": se
    })

# Annual temperature trend
annual_tmax = df["tmax"].resample("YE").mean()
trend_result = linear_trend(annual_tmax)
print(f"Trend: {trend_result['slope']:.3f} °C/year")
print(f"P-value: {trend_result['p_value']:.4f}")

# Add trend line
years = np.arange(len(annual_tmax))
trend_line = trend_result["slope"] * years + trend_result["intercept"]
```

### 7.3 Climate Indices from Station Data

```python
# --- Growing Degree Days ---
df["gdd"] = (df["tmax"] + df["tmin"]) / 2 - 10   # Base 10°C
df["gdd"] = df["gdd"].clip(lower=0)               # No negative GDD

# --- SPI (Standardized Precipitation Index) ---
monthly_precip = df["precip"].resample("ME").sum()
monthly_mean = monthly_precip.groupby(monthly_precip.index.month).transform("mean")
monthly_std = monthly_precip.groupby(monthly_precip.index.month).transform("std")
spi = (monthly_precip - monthly_mean) / monthly_std

# --- Diurnal Temperature Range ---
df["dtr"] = df["tmax"] - df["tmin"]

# --- Consecutive Dry Days ---
def consecutive_dry_days(series, threshold=1.0):
    dry = series < threshold
    cdd = dry.groupby((~dry).cumsum()).cumsum()
    return cdd

df["cdd"] = consecutive_dry_days(df["precip"])
```

---

## 📁 LEVEL 8 — Complete Workflow

```python
import pandas as pd
import numpy as np
from scipy import stats
import matplotlib.pyplot as plt

# 1. Load station data
df = pd.read_csv(
    "islamabad_1981_2020.csv",
    parse_dates=["date"],
    index_col="date",
    na_values=["-999"]
)

# 2. Quality control
df = df[df["tmax"] < 55]         # Remove physically impossible
df = df[df["tmin"] > -10]
df = df[df["tmax"] > df["tmin"]] # Consistency check

# 3. Fill missing values
for col in ["tmax", "tmin", "precip"]:
    monthly_clim = df[col].groupby(df.index.month).transform("mean")
    df[col] = df[col].fillna(monthly_clim)

# 4. Monthly climatology (1981-2010 WMO normal)
base_period = df.loc["1981":"2010"]
monthly_clim = base_period.groupby(base_period.index.month)[["tmax", "tmin", "precip"]].mean()

# 5. Annual statistics
annual = df.resample("YE").agg({
    "tmax": "mean",
    "tmin": "mean",
    "precip": "sum"
})

# 6. Trend analysis
for var in ["tmax", "tmin", "precip"]:
    clean = annual[var].dropna()
    x = np.arange(len(clean))
    slope, _, r, p, _ = stats.linregress(x, clean)
    print(f"{var}: {slope:.3f} per year (p={p:.3f})")

# 7. Export results
annual.to_csv("islamabad_annual_stats.csv")
monthly_clim.to_excel("islamabad_climatology.xlsx")
```

---

## 📋 pandas Quick Reference Card

| Operation | Code |
|-----------|------|
| Read CSV | `pd.read_csv("file.csv", parse_dates=["date"], index_col="date")` |
| Check info | `df.info()`, `df.describe()` |
| Select column | `df["tmax"]` or `df.tmax` |
| Filter rows | `df[df["tmax"] > 40]` |
| Select by label | `df.loc["2020", "tmax"]` |
| Resample monthly | `df["tmax"].resample("MS").mean()` |
| Resample annual | `df["tmax"].resample("YE").mean()` |
| GroupBy month | `df.groupby(df.index.month)["tmax"].mean()` |
| Rolling mean | `df["tmax"].rolling(30).mean()` |
| Monthly anomaly | `df["tmax"] - monthly_clim[df.index.month].values` |
| Correlation | `df[["tmax","precip"]].corr()` |
| Count missing | `df.isna().sum()` |
| Fill missing | `df.fillna(method="ffill")` |
| Pivot | `df.pivot_table(index="year", columns="station", values="tmax")` |

---

## 📚 Resources

- [pandas Documentation](https://pandas.pydata.org/docs/)
- [pandas User Guide](https://pandas.pydata.org/docs/user_guide/index.html)
- [pandas Cheat Sheet](https://pandas.pydata.org/Pandas_Cheat_Sheet.pdf)
- [Time Series Analysis in pandas](https://pandas.pydata.org/docs/user_guide/timeseries.html)
