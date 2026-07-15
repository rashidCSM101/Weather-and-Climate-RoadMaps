# 🔢 NumPy + SciPy — Complete Learning Roadmap
## Array Operations & Statistical Analysis for Climate Science

---

## 📚 What & Why?

- **NumPy**: The fundamental array computing library. All climate data (after loading with xarray/pandas) ultimately lives as NumPy arrays. Understanding NumPy is essential for custom calculations.
- **SciPy**: Scientific computing on top of NumPy — statistics, signal processing, interpolation, optimization. Essential for trend tests, distribution fitting, and hypothesis testing.

---

## 🗺️ Roadmap Overview

```
NUMPY:
  Level 1: Array Basics        → Creation, dtypes, indexing, slicing
  Level 2: Array Operations    → Broadcasting, vectorization, masking
  Level 3: Statistical Ops     → Mean, std, percentiles, correlations
  Level 4: Climate-Specific    → Weighted means, spatial statistics

SCIPY:
  Level 5: scipy.stats         → Distributions, tests, regression
  Level 6: Trend Tests         → Mann-Kendall setup, linear regression
  Level 7: Distribution Fitting→ Fitting GEV, Gamma for extremes
  Level 8: Signal Processing   → Filtering, FFT, wavelet basics
```

---

## 📁 NUMPY — LEVEL 1: Foundations

### 1.1 Array Creation

```python
import numpy as np

# --- From lists ---
arr = np.array([1, 2, 3, 4, 5])
arr_2d = np.array([[1, 2, 3], [4, 5, 6]])
arr_3d = np.zeros((12, 90, 180))       # 12 months, 90 lat, 180 lon

# --- Common creation functions ---
np.zeros((90, 180))              # All zeros (land mask template)
np.ones((90, 180))               # All ones
np.full((90, 180), np.nan)       # All NaN (for masked output)
np.linspace(-90, 90, 181)        # 181 evenly spaced lat values
np.arange(1950, 2024)            # Year array
np.eye(5)                        # Identity matrix

# --- Random arrays (for testing) ---
np.random.seed(42)               # Reproducibility
np.random.normal(loc=25, scale=5, size=(365, 90, 180))   # Temperature-like
np.random.exponential(scale=2, size=(365,))               # Precipitation-like

# --- From netCDF via xarray ---
import xarray as xr
da = xr.open_dataset("ERA5.nc")["t2m"]
arr = da.values         # Extract underlying NumPy array
lat = da.lat.values
lon = da.lon.values
```

### 1.2 Array Attributes & Inspection

```python
arr = np.random.rand(12, 90, 180)

arr.shape         # (12, 90, 180)
arr.ndim          # 3
arr.size          # 12 * 90 * 180 = 194400
arr.dtype         # float64
arr.itemsize      # 8 bytes per element
arr.nbytes        # Total bytes

# Reshape
arr_2d = arr.reshape(12, 90*180)      # (12, 16200)
arr_flat = arr.flatten()              # 1D array
arr_T = arr.T                         # Transpose
```

### 1.3 Indexing & Slicing

```python
# 1D indexing
arr = np.array([10, 20, 30, 40, 50])
arr[0]         # 10 (first)
arr[-1]        # 50 (last)
arr[1:4]       # [20, 30, 40]
arr[::2]       # [10, 30, 50] (every 2nd)

# 2D indexing (lat/lon grid)
grid = np.random.rand(90, 180)
grid[0, 0]           # NW corner
grid[45, 90]         # Center
grid[:, 90]          # All lat, lon=90 (meridian)
grid[40:50, 60:80]   # Subgrid (Pakistan region)
grid[-1, :]          # Last row (South Pole)

# 3D (time, lat, lon)
data = np.random.rand(365, 90, 180)
data[0]              # First time step (90×180)
data[:, 0, :]        # First lat row, all times
data[0:90, :, :]     # First 90 days (JFM)

# Boolean indexing
data[data > 300]         # Values above 300K
data[np.isnan(data)]     # NaN locations
data[~np.isnan(data)]    # Non-NaN values
```

---

## 📁 NUMPY — LEVEL 2: Array Operations

### 2.1 Broadcasting Rules

```python
# Broadcasting: arrays with different shapes can be combined
# Rule: shapes are compared from the RIGHT, and dimensions must match or be 1

# Example: multiply 3D climate array by latitude weights (1D)
data = np.random.rand(12, 90, 180)   # (time, lat, lon)
lat = np.linspace(-89, 89, 90)
weights = np.cos(np.deg2rad(lat))    # (90,) — 1D

# Broadcasting step by step:
weights_3d = weights[:, np.newaxis]  # (90, 1)
# data (12, 90, 180) × weights (90, 1) → (12, 90, 180) ✅

weighted = data * weights[:, np.newaxis]   # Efficient, no loop!

# More broadcasting examples
data_monthly = np.random.rand(120, 90, 180)  # 10 years × 12 months
clim = np.random.rand(12, 90, 180)           # 12-month climatology
# Subtract: (120, 90, 180) - (12, 90, 180) → NOT automatic!
# Need to reshape clim or tile it
```

### 2.2 Vectorized Operations (No Loops!)

```python
# BAD: Python loops (slow)
result = np.zeros((90, 180))
for i in range(90):
    for j in range(180):
        result[i, j] = data[:, i, j].mean()

# GOOD: Vectorized (fast)
result = data.mean(axis=0)   # Mean over axis 0 (time) → (90, 180)

# --- Axis parameter is crucial ---
data.mean(axis=0)           # Mean over time → (lat, lon)
data.mean(axis=(1, 2))      # Mean over lat, lon → (time,) area mean
data.mean(axis=-1)          # Mean over last axis (lon) → (time, lat)

# Common vectorized climate operations
data.mean(axis=0)           # Climatological mean map
data.std(axis=0)            # Climatological std map
data.max(axis=0)            # All-time maximum
data.min(axis=0)            # All-time minimum
(data > 300).sum(axis=0)    # Count days above 300K per grid cell
np.percentile(data, 95, axis=0)  # 95th percentile map
```

### 2.3 Masking & NaN Handling

```python
# Create masked array
data_masked = np.ma.array(data, mask=np.isnan(data))

# NaN-aware functions
np.nanmean(data, axis=0)     # Mean ignoring NaN
np.nanstd(data, axis=0)
np.nanmax(data, axis=0)
np.nanmin(data, axis=0)
np.nansum(data, axis=0)
np.nanpercentile(data, 95, axis=0)

# Apply a mask (set values to NaN)
land_mask = np.load("land_mask.npy")   # True = land
data_ocean = np.where(land_mask, np.nan, data)   # NaN over land

# Replace specific values
data[data == -9999] = np.nan   # Replace missing value code
data = np.clip(data, 200, 400)  # Clip to physically reasonable range
```

### 2.4 Efficient Array Operations

```python
# Stack arrays
data_all = np.stack([data_2019, data_2020, data_2021], axis=0)  # New axis

# Concatenate
data_concat = np.concatenate([data_2019, data_2020], axis=0)    # Existing axis

# Tiling / repeating
annual = np.random.rand(12, 90, 180)    # 1 year
decade = np.tile(annual, (10, 1, 1))    # Repeat 10 times → (120, 90, 180)

# Matrix multiplication (for EOF analysis)
A = data_2d    # (time, space)
ATA = A.T @ A  # Covariance matrix (space × space)
cov = np.cov(A.T)  # Alternative

# Sorting
data_sorted = np.sort(data, axis=0)   # Sort along time axis
idx = np.argsort(data, axis=0)        # Get sorted indices (useful for ranking)
```

---

## 📁 NUMPY — LEVEL 3: Statistical Operations

### 3.1 Descriptive Statistics

```python
data = np.random.normal(25, 5, (365, 90, 180))

# Basic stats
np.mean(data, axis=0)
np.median(data, axis=0)
np.std(data, axis=0)
np.var(data, axis=0)
np.min(data, axis=0)
np.max(data, axis=0)
np.ptp(data, axis=0)              # Peak-to-peak (max - min)
np.sum(data, axis=0)

# Percentiles
np.percentile(data, [10, 25, 50, 75, 90, 95, 99], axis=0)

# Histogram
hist, bin_edges = np.histogram(data.flatten(), bins=50)
```

### 3.2 Correlation & Covariance

```python
# 1D correlation
x = np.random.rand(100)
y = np.random.rand(100)
np.corrcoef(x, y)                 # 2×2 correlation matrix
r = np.corrcoef(x, y)[0, 1]      # Pearson r value

# Covariance
np.cov(x, y)                      # 2×2 covariance matrix

# Gridded correlation (map of correlation with index)
nino34 = np.random.rand(365)      # ENSO index
# Vectorized correlation with each grid point
def corr_map(field, index):
    # field: (time, lat, lon), index: (time,)
    field_anom = field - field.mean(axis=0)
    index_anom = index - index.mean()
    numerator = (field_anom * index_anom[:, np.newaxis, np.newaxis]).mean(axis=0)
    denominator = field_anom.std(axis=0) * index_anom.std()
    return numerator / denominator

r_map = corr_map(data, nino34)    # (lat, lon) correlation map
```

### 3.3 Weighted Statistics (Latitude Weighting)

```python
lat = np.linspace(-89, 89, 90)
weights = np.cos(np.deg2rad(lat))          # Area weights
weights /= weights.sum()                   # Normalize

# Global mean (lat-weighted)
lat_weighted_mean = (data * weights[:, np.newaxis]).sum(axis=1)  # (time, lon)
global_mean = lat_weighted_mean.mean(axis=1)  # (time,)

# Or using np.average
global_mean = np.average(
    data.mean(axis=2),             # Average over lon first
    weights=weights,               # Weight by cos(lat)
    axis=1                         # Over lat
)
```

---

## 📁 SCIPY — LEVEL 5: Statistics

### 5.1 Basic Hypothesis Tests

```python
from scipy import stats

# One-sample t-test (is mean significantly different from a value?)
t_stat, p_value = stats.ttest_1samp(sample, popmean=0)

# Two-sample t-test (are two periods different?)
t_stat, p_value = stats.ttest_ind(period1, period2, equal_var=False)  # Welch's

# Paired t-test (same stations, different periods)
t_stat, p_value = stats.ttest_rel(before, after)

# Wilcoxon test (non-parametric alternative to paired t-test)
stat, p = stats.wilcoxon(before, after)

# Mann-Whitney U (non-parametric independent samples)
stat, p = stats.mannwhitneyu(group1, group2, alternative="two-sided")

# Kolmogorov-Smirnov (distribution comparison)
stat, p = stats.ks_2samp(distribution1, distribution2)
```

### 5.2 Linear Regression & Trend

```python
from scipy.stats import linregress

# 1D time series trend
years = np.arange(1950, 2024)
temperature = np.random.rand(74) + np.linspace(0, 2, 74)  # Warming trend

result = linregress(years, temperature)
print(f"Slope: {result.slope:.4f} °C/year")
print(f"Intercept: {result.intercept:.4f}")
print(f"R²: {result.rvalue**2:.4f}")
print(f"P-value: {result.pvalue:.4f}")
print(f"Std error: {result.stderr:.4f}")

# Trend line
trend_line = result.slope * years + result.intercept

# Gridded trend map (loop approach — use xr.apply_ufunc for large data!)
slope_map = np.full((90, 180), np.nan)
pval_map = np.full((90, 180), np.nan)

for i in range(90):
    for j in range(180):
        ts = data[:, i, j]
        if not np.any(np.isnan(ts)):
            r = linregress(np.arange(len(ts)), ts)
            slope_map[i, j] = r.slope
            pval_map[i, j] = r.pvalue
```

### 5.3 Correlation Tests

```python
from scipy.stats import pearsonr, spearmanr, kendalltau

# Pearson (assumes normal distribution)
r, p = pearsonr(x, y)

# Spearman (rank-based, non-parametric)
rho, p = spearmanr(x, y)

# Kendall's tau (rank-based, better for small samples)
tau, p = kendalltau(x, y)

print(f"Pearson r = {r:.3f}, p = {p:.3f}")
print(f"Spearman ρ = {rho:.3f}, p = {p:.3f}")
print(f"Kendall τ = {tau:.3f}, p = {p:.3f}")
```

---

## 📁 SCIPY — LEVEL 6: Distribution Fitting (Extreme Events)

### 6.1 Fitting Distributions

```python
from scipy import stats

# Fit a Gamma distribution to precipitation
precip = np.random.exponential(5, 1000)  # Example data

# Fit: returns shape parameters
shape, loc, scale = stats.gamma.fit(precip, floc=0)  # Fix location=0

# Generate PDF
x = np.linspace(0, 50, 200)
pdf = stats.gamma.pdf(x, shape, loc=loc, scale=scale)

# Return level (extreme value)
# What precipitation amount is exceeded with 1% probability (100-year event)?
return_100yr = stats.gamma.ppf(0.99, shape, loc=loc, scale=scale)
print(f"100-year rainfall: {return_100yr:.1f} mm")

# --- GEV (Generalized Extreme Value) for annual maxima ---
annual_max_precip = np.random.rand(74) * 100 + 50

c, loc, scale = stats.genextreme.fit(annual_max_precip)  # c = shape parameter
print(f"GEV shape (ξ): {c:.3f}, loc (μ): {loc:.3f}, scale (σ): {scale:.3f}")

# Return periods
return_periods = [2, 5, 10, 25, 50, 100]
for rp in return_periods:
    quantile = 1 - 1/rp
    level = stats.genextreme.ppf(quantile, c, loc, scale)
    print(f"{rp}-year event: {level:.1f} mm")

# --- Normal distribution fit ---
mu, sigma = stats.norm.fit(temperature)
```

---

## 📁 SCIPY — LEVEL 7: Signal Processing

### 7.1 Filtering (Removing Noise)

```python
from scipy import signal

# --- Butterworth low-pass filter (remove high-frequency noise) ---
# Retain periods > 10 years (low-pass)
fs = 1.0            # 1 sample/year
cutoff = 1/10       # 1/10 year = retain 10+ year signals
order = 4           # Filter order

b, a = signal.butter(order, cutoff / (fs/2), btype="low")
filtered = signal.filtfilt(b, a, temperature_ts)  # Zero-phase filter

# --- Running mean (simple low-pass) ---
def running_mean(x, N):
    return np.convolve(x, np.ones(N)/N, mode="same")

smooth = running_mean(temperature_ts, 10)

# --- FFT (spectral analysis — find dominant periods) ---
ts = temperature_ts - temperature_ts.mean()   # Remove mean
fft = np.fft.fft(ts)
freqs = np.fft.fftfreq(len(ts), d=1)         # d=1 for annual data
power = np.abs(fft)**2 / len(ts)

# Find dominant period
dominant_freq = freqs[np.argmax(power[1:len(ts)//2])+1]
dominant_period = 1 / dominant_freq
print(f"Dominant period: {dominant_period:.1f} years")
```

---

## 📋 NumPy Quick Reference

| Operation | Code |
|-----------|------|
| Create zeros | `np.zeros((12, 90, 180))` |
| Create range | `np.arange(1950, 2024)` |
| Linspace | `np.linspace(-90, 90, 181)` |
| Mean over time | `np.nanmean(data, axis=0)` |
| Std over time | `np.nanstd(data, axis=0)` |
| Percentile | `np.nanpercentile(data, 95, axis=0)` |
| Boolean mask | `data[data > 0]` |
| Replace NaN | `np.where(np.isnan(data), 0, data)` |
| Lat weights | `np.cos(np.deg2rad(lat))` |
| Weighted mean | `np.average(data, weights=weights, axis=1)` |
| Correlation | `np.corrcoef(x, y)[0,1]` |

## 📋 SciPy Quick Reference

| Test | Code |
|------|------|
| Linear trend | `stats.linregress(x, y)` |
| Pearson r | `stats.pearsonr(x, y)` |
| Spearman ρ | `stats.spearmanr(x, y)` |
| t-test | `stats.ttest_ind(a, b)` |
| Fit Gamma | `stats.gamma.fit(data, floc=0)` |
| Fit GEV | `stats.genextreme.fit(maxima)` |
| Return level | `stats.genextreme.ppf(0.99, c, loc, scale)` |

---

## 📚 Resources

- [NumPy Documentation](https://numpy.org/doc/)
- [SciPy Documentation](https://docs.scipy.org)
- [SciPy Statistics](https://docs.scipy.org/doc/scipy/reference/stats.html)
- [NumPy for Climate Science](https://numpy.org/numpy-tutorials/)
