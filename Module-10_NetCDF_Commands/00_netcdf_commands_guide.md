# 🗄️ NetCDF Commands — Complete Reference Guide
## CDO, NCO, ncdump, ncview & Python NetCDF Tools

---

## 📚 What is NetCDF?

NetCDF (Network Common Data Form) is the **universal file format for climate data**. Nearly every major climate dataset — ERA5, CMIP6, CORDEX, GHCN — uses it.

**Structure:**
```
NetCDF File (.nc):
├── Global Attributes     → Source, history, Conventions
├── Dimensions            → time, lat, lon, level
├── Variables
│   ├── Coordinate vars   → time(:), lat(:), lon(:)
│   └── Data vars         → temperature(time,lat,lon)
│       ├── Attributes    → units, long_name, _FillValue, scale_factor
│       └── Data          → Actual numeric array
```

---

## 🗺️ Roadmap Overview

```
Section 1: ncdump — Inspect NC files from command line
Section 2: ncview  — Quick visual inspection
Section 3: CDO    — Climate Data Operators (most powerful CLI tool)
Section 4: NCO    — NetCDF Operators (lower-level manipulation)
Section 5: CDO ECA — Climate Extremes Indices (TXx, TNn, Rx1day, Rx5day)
Section 6: Python  — netCDF4 library (raw access)
Section 7: Python  — xarray (recommended approach)
Section 8: Recipes — Common real-world operations
```

---

## 📁 SECTION 1 — ncdump: Inspect NetCDF Files

### 1.1 Installation

```bash
# On Windows (via conda — RECOMMENDED)
conda install -c conda-forge netcdf4 nco cdo

# Or via pip (Python only, no CLI tools)
pip install netCDF4
```

### 1.2 ncdump Commands

```bash
# View file header (metadata only, no data)
ncdump -h ERA5_t2m_2020.nc

# Output:
# netcdf ERA5_t2m_2020 {
# dimensions:
#     longitude = 1440 ;
#     latitude = 721 ;
#     time = 12 ;
# variables:
#     float longitude(longitude) ;
#         longitude:units = "degrees_east" ;
#     float latitude(latitude) ;
#         latitude:units = "degrees_north" ;
#     int time(time) ;
#         time:units = "hours since 1900-01-01 00:00:00.0" ;
#         time:calendar = "gregorian" ;
#     short t2m(time, latitude, longitude) ;
#         t2m:scale_factor = 0.00056 ;
#         t2m:add_offset = 266.7 ;
#         t2m:_FillValue = -32767 ;
#         t2m:missing_value = -32767 ;
#         t2m:units = "K" ;
#         t2m:long_name = "2 metre temperature" ;
# // global attributes:
#         :Conventions = "CF-1.6" ;
#         :history = "2020-01-01 00:00:00 ..." ;
# }

# View specific variable data
ncdump -v t2m ERA5.nc | head -100

# View with time values decoded
ncdump -t ERA5.nc | grep "time ="

# Count variables and dimensions
ncdump -h ERA5.nc | grep -E "^(float|double|int|short|char)" | wc -l

# Show all global attributes only
ncdump -g ERA5.nc
```

---

## 📁 SECTION 2 — ncview: Quick Visual Inspection

```bash
# Install (Linux/Mac; Windows needs WSL or conda)
conda install -c conda-forge ncview

# Open file for visual inspection
ncview ERA5_t2m_2020.nc

# Features:
# - Click variable → see spatial map
# - Drag time slider → animate through time
# - Click grid cell → see time series
# - Zoom, colormap control
```

---

## 📁 SECTION 3 — CDO: Climate Data Operators

CDO is the **most powerful command-line tool** for climate data processing. It can do in one line what would take 20 lines of Python.

### 3.1 Installation

```bash
conda install -c conda-forge cdo
```

### 3.2 Basic Information

```bash
# File info (like ncdump -h but cleaner)
cdo info ERA5.nc
cdo sinfo ERA5.nc         # Detailed info
cdo showname ERA5.nc      # Variable names only
cdo showlevel ERA5.nc     # Pressure levels
cdo showtimestamp ERA5.nc # All timestamps

# File size and grid
cdo griddes ERA5.nc       # Grid description
cdo zaxisdes ERA5.nc      # Vertical axis description
```

### 3.3 Temporal Operations

```bash
# Compute temporal mean (all time → single value)
cdo timmean ERA5.nc ERA5_mean.nc

# Annual mean from daily/monthly data
cdo yearmean ERA5_daily.nc ERA5_annual.nc

# Monthly mean from daily data
cdo monmean ERA5_daily.nc ERA5_monthly.nc

# Seasonal mean (DJF, MAM, JJA, SON)
cdo seasmean ERA5_monthly.nc ERA5_seasonal.nc

# Long-term monthly mean (climatology, 30-year normal)
cdo ymonmean ERA5_monthly.nc ERA5_climatology.nc

# Monthly anomalies (subtract climatology)
cdo ymonsub ERA5_monthly.nc ERA5_climatology.nc ERA5_anomaly.nc

# Select specific years
cdo selyear,1981/2010 ERA5_full.nc ERA5_base.nc
cdo selyear,1979/2023 ERA5_full.nc ERA5_study.nc

# Select specific months (e.g., JJA = June, July, August)
cdo selmon,6,7,8 ERA5_monthly.nc ERA5_JJA.nc

# Select specific season
cdo selseas,JJA ERA5_monthly.nc ERA5_JJA.nc
cdo selseas,DJF ERA5_monthly.nc ERA5_DJF.nc

# Select date range
cdo seldate,1979-01-01,2023-12-31 ERA5.nc ERA5_subset.nc

# Maximum/minimum over time
cdo timmax ERA5.nc ERA5_max.nc
cdo timmin ERA5.nc ERA5_min.nc

# Standard deviation
cdo timstd ERA5.nc ERA5_std.nc
```

### 3.4 Spatial Operations

```bash
# Spatial mean (area average — NOT latitude-weighted!)
cdo fldmean ERA5.nc ERA5_spatial_mean.nc

# Latitude-weighted spatial mean (correct for sphere!)
cdo fldmean -sellonlatbox,60,80,20,40 ERA5.nc pakistan_mean.nc

# Select spatial region (bounding box: lon1,lon2,lat1,lat2)
cdo sellonlatbox,60,80,20,40 ERA5.nc ERA5_Pakistan.nc
cdo sellonlatbox,-180,180,-90,90 ERA5.nc ERA5_global.nc  # Full globe

# Regrid to different resolution
# Bilinear interpolation to 1° grid
cdo remapbil,r360x180 ERA5_025.nc ERA5_1deg.nc

# Nearest neighbor
cdo remapnn,r360x180 ERA5.nc ERA5_1deg_nn.nc

# Conservative remapping (preserves area integrals, RECOMMENDED for precip)
cdo remapcon,r360x180 ERA5_pr.nc ERA5_pr_1deg.nc

# Define target grid from file
cdo remapbil,target_grid.nc source.nc output.nc

# Flip latitudes (some models have reversed lat)
cdo invertlat ERA5.nc ERA5_flipped.nc

# Select specific level (pressure levels)
cdo sellevel,850 ERA5_pl.nc ERA5_850hPa.nc
cdo sellevel,850,500,200 ERA5_pl.nc ERA5_levels.nc
```

### 3.5 Arithmetic Operations

```bash
# Convert Kelvin to Celsius
cdo subc,273.15 ERA5_K.nc ERA5_C.nc

# Scale precipitation (m → mm)
cdo mulc,1000 ERA5_pr_m.nc ERA5_pr_mm.nc

# Add two files
cdo add file1.nc file2.nc output.nc

# Subtract (anomaly)
cdo sub ERA5.nc ERA5_clim.nc ERA5_anomaly.nc

# Multiply
cdo mul ERA5.nc weights.nc ERA5_weighted.nc

# Power
cdo sqr ERA5.nc ERA5_squared.nc     # Square
cdo sqrt ERA5.nc ERA5_sqrt.nc       # Square root

# Set missing values
cdo setmissval,-9999 ERA5.nc ERA5_clean.nc
cdo setmisstoc,0 ERA5.nc ERA5_zeros.nc   # Replace NaN with 0
```

### 3.6 Multi-File Operations

```bash
# Merge (concatenate along time)
cdo mergetime ERA5_2019.nc ERA5_2020.nc ERA5_2021.nc ERA5_merged.nc
cdo mergetime ERA5_20*.nc ERA5_all.nc

# Merge along variables
cdo merge ERA5_temperature.nc ERA5_precip.nc ERA5_combined.nc

# Select variable from multi-variable file
cdo selvar,t2m ERA5.nc ERA5_t2m_only.nc
cdo selvar,t2m,tp ERA5.nc ERA5_t2m_tp.nc

# Split by year (create one file per year)
cdo splityear ERA5_full.nc ERA5_year_

# Split by month
cdo splitmon ERA5_full.nc ERA5_month_

# Split by variable
cdo splitvar ERA5_combined.nc ERA5_
```

### 3.7 Chaining Operations (Piping)

```bash
# Chain multiple operators (piping with -)
# Example: Select Pakistan, compute annual mean, convert to Celsius
cdo subc,273.15 -yearmean -sellonlatbox,60,80,20,40 ERA5.nc ERA5_Pak_annual_C.nc

# Example: Monthly anomaly for Pakistan JJA
cdo ymonsub -selmon,6,7,8 -sellonlatbox,60,80,20,40 ERA5.nc \
    -selmon,6,7,8 ERA5_clim.nc ERA5_Pak_JJA_anomaly.nc

# Example: Detrended time series
cdo detrend ERA5_ts.nc ERA5_detrended.nc
```

### 3.8 Statistics & Trend Analysis

```bash
# Linear trend (slope at each grid point)
cdo trend ERA5.nc ERA5_intc.nc ERA5_slope.nc
# ERA5_slope.nc contains trend slope (units per timestep)

# Remove trend
cdo detrend ERA5.nc ERA5_detrended.nc

# Running mean (5-year)
cdo runmean,5 ERA5_annual.nc ERA5_5yr_running.nc

# Percentile
cdo timselpercentile,90 ERA5.nc ERA5_p90.nc

# Standard deviation over time
cdo timstd ERA5.nc ERA5_std.nc

# Correlation between two fields
cdo timcor ERA5_t2m.nc ERA5_precip.nc correlation.nc
```

---

## 📁 SECTION 4 — NCO: NetCDF Operators

NCO is for lower-level NetCDF manipulation — renaming variables, editing attributes, changing dimensions.

### 4.1 ncks — Slice & Dice

```bash
# Extract specific variable
ncks -v t2m ERA5.nc ERA5_t2m.nc

# Extract multiple variables
ncks -v t2m,tp,u10,v10 ERA5.nc ERA5_subset.nc

# Extract spatial subset
ncks -d lat,20.0,40.0 -d lon,60.0,80.0 ERA5.nc ERA5_Pakistan.nc

# Extract time subset
ncks -d time,0,11 ERA5.nc ERA5_first12.nc    # First 12 time steps

# Extract with coordinate values
ncks -d latitude,20.0,40.0 -d longitude,60.0,80.0 ERA5.nc output.nc

# Combine: variable + spatial subset
ncks -v t2m -d lat,20.0,40.0 -d lon,60.0,80.0 ERA5.nc ERA5_Pak_t2m.nc
```

### 4.2 ncrename — Rename Variables/Dimensions

```bash
# Rename variable
ncrename -v t2m,temperature ERA5.nc ERA5_renamed.nc

# Rename dimension
ncrename -d longitude,lon ERA5.nc ERA5_renamed.nc
ncrename -d latitude,lat ERA5.nc ERA5_renamed.nc

# Rename multiple at once
ncrename -v t2m,tas -d longitude,lon -d latitude,lat ERA5.nc ERA5_cmip_names.nc
```

### 4.3 ncatted — Edit Attributes

```bash
# Change units attribute
ncatted -a units,t2m,o,c,"K" ERA5.nc ERA5_units.nc

# Add long_name
ncatted -a long_name,t2m,o,c,"2 metre temperature" ERA5.nc output.nc

# Change missing value
ncatted -a _FillValue,t2m,o,f,-9999.0 ERA5.nc output.nc

# Delete attribute
ncatted -a scale_factor,t2m,d,, ERA5.nc output.nc

# Edit global attribute
ncatted -a history,global,a,c,"\nProcessed by Rashid 2024" ERA5.nc output.nc
```

### 4.4 ncra & ncea — Averaging

```bash
# Average all files in time (record average)
ncra ERA5_2019.nc ERA5_2020.nc ERA5_2021.nc ERA5_average.nc

# Ensemble average (average along file axis, not time)
ncea model1.nc model2.nc model3.nc ensemble_mean.nc
```

### 4.5 ncap2 — Arithmetic Processing

```bash
# Convert K to Celsius
ncap2 -s "t2m=t2m-273.15" ERA5.nc ERA5_C.nc

# Compute new variable
ncap2 -s "DTR=tasmax-tasmin" ERA5.nc ERA5_DTR.nc

# Multiple operations
ncap2 -s "tas=t2m-273.15; pr=pr*86400" ERA5.nc ERA5_processed.nc
```

### 4.6 ncbo — Binary Operations (Two Files)

```bash
# Subtract (anomaly)
ncbo --op_typ=sbt ERA5.nc ERA5_clim.nc ERA5_anomaly.nc

# Add files
ncbo --op_typ=add file1.nc file2.nc output.nc
```

### 4.7 ncdiff — Difference

```bash
# Difference between two files (future - historical)
ncdiff future_2081_2100.nc historical_1981_2000.nc change.nc
```

---

## 📁 SECTION 5 — Python: netCDF4 Library (Raw Access)

### 5.1 Reading Files

```python
from netCDF4 import Dataset
import numpy as np

# Open file
nc = Dataset("ERA5_t2m_2020.nc", "r")   # "r" = read mode

# View structure
print(nc)                              # Full summary
print(nc.variables.keys())             # Variable names: dict_keys(['lon','lat','time','t2m'])
print(nc.dimensions)                   # Dimension info

# Access variables
lat  = nc.variables["latitude"][:]    # NumPy masked array
lon  = nc.variables["longitude"][:]
time = nc.variables["time"][:]
t2m  = nc.variables["t2m"][:]         # Shape: (time, lat, lon)

# Access attributes
print(nc.variables["t2m"].units)       # "K"
print(nc.variables["t2m"].long_name)   # "2 metre temperature"
print(nc.variables["t2m"]._FillValue)  # -32767
print(nc.variables["t2m"].scale_factor)
print(nc.variables["t2m"].add_offset)

# Decode time
import cftime
time_var = nc.variables["time"]
dates = cftime.num2date(time_var[:], time_var.units, time_var.calendar)
print(dates[0])   # e.g., datetime(2020, 1, 1, 0, 0, 0)

# Close file (important!)
nc.close()

# Better: use context manager
with Dataset("ERA5.nc", "r") as nc:
    t2m = nc.variables["t2m"][:]
    # File auto-closes after block
```

### 5.2 Writing NetCDF Files

```python
from netCDF4 import Dataset
import numpy as np

# Create output file
with Dataset("output.nc", "w", format="NETCDF4") as nc:

    # Create dimensions
    nc.createDimension("time", None)      # None = unlimited
    nc.createDimension("lat", 90)
    nc.createDimension("lon", 180)

    # Create coordinate variables
    times = nc.createVariable("time", "f8", ("time",))
    lats  = nc.createVariable("lat", "f4", ("lat",))
    lons  = nc.createVariable("lon", "f4", ("lon",))

    # Set coordinate attributes
    times.units = "days since 1979-01-01"
    times.calendar = "standard"
    lats.units = "degrees_north"
    lons.units = "degrees_east"

    # Create data variable with compression
    temp = nc.createVariable(
        "temperature", "f4", ("time", "lat", "lon"),
        zlib=True,            # Enable compression
        complevel=4,          # Compression level (1-9)
        least_significant_digit=2,  # Round to 2 decimal places
        fill_value=np.nan,
        chunksizes=(1, 90, 180)     # Chunk for time access
    )

    # Set data variable attributes
    temp.units = "K"
    temp.long_name = "Near-surface Air Temperature"
    temp.standard_name = "air_temperature"
    temp.missing_value = -9999.0
    temp.coordinates = "lat lon"

    # Global attributes
    nc.Conventions = "CF-1.8"
    nc.title = "ERA5 Surface Temperature"
    nc.history = "Created 2024-01-01"
    nc.institution = "Quaid-i-Azam University"
    nc.source = "ERA5 Reanalysis"
    nc.contact = "rashid@qau.edu.pk"

    # Write coordinate data
    from netCDF4 import date2num
    import datetime
    date_list = [datetime.datetime(2020, m, 1) for m in range(1, 13)]
    times[:] = date2num(date_list, times.units)
    lats[:]  = np.linspace(-89, 89, 90)
    lons[:]  = np.linspace(-179, 179, 180)

    # Write actual data
    temp[:] = np.random.rand(12, 90, 180) + 280   # Fake temperature data
```

---

## 📁 SECTION 6 — Python: xarray (RECOMMENDED)

See **Module-2_xarray** for the full guide. Quick reference for NC operations:

```python
import xarray as xr

# Open single file
ds = xr.open_dataset("ERA5.nc")

# Open multiple files (automatic concatenation along time)
ds = xr.open_mfdataset("ERA5_*.nc", combine="by_coords")

# Open with Dask (lazy loading for large files)
ds = xr.open_mfdataset("ERA5_*.nc", chunks={"time": 100})

# Inspect
print(ds)                    # Dataset overview
print(ds.data_vars)          # Variables
print(ds.coords)             # Coordinates
print(ds.attrs)              # Global metadata
print(ds["t2m"].attrs)       # Variable metadata

# Access variable
da = ds["t2m"]
print(da.shape)
print(da.dims)

# Convert Kelvin to Celsius
da_C = da - 273.15
da_C.attrs["units"] = "°C"

# Select region (Pakistan)
da_pak = da.sel(
    lat=slice(37, 23),        # Note: ERA5 lat goes N→S
    lon=slice(60, 80)
)

# Compute
annual_mean = da.resample(time="1Y").mean()

# Save
annual_mean.to_netcdf("ERA5_annual_mean.nc", encoding={
    "t2m": {"dtype": "float32", "zlib": True, "complevel": 4}
})
```

---

## 📁 SECTION 7 — Real-World Recipe Collection

### Recipe 1: Download + Subset ERA5

```bash
# 1. Download ERA5 for Pakistan (using CDS API — Python)
# (see cdsapi documentation)

# 2. Subset to Pakistan
cdo sellonlatbox,60,80,23,37 ERA5_global.nc ERA5_Pakistan.nc

# 3. Annual means
cdo yearmean ERA5_Pakistan.nc ERA5_Pakistan_annual.nc

# 4. Convert K to °C
cdo subc,273.15 ERA5_Pakistan_annual.nc ERA5_Pakistan_annual_C.nc
```

### Recipe 2: Compute Climatology & Anomaly

```bash
# Step 1: Compute climatology (1981-2010 monthly mean)
cdo selyear,1981/2010 ERA5_monthly.nc ERA5_base.nc
cdo ymonmean ERA5_base.nc ERA5_climatology.nc

# Step 2: Compute anomaly (full period - climatology)
cdo ymonsub ERA5_monthly.nc ERA5_climatology.nc ERA5_anomaly.nc
```

### Recipe 3: Multi-Model Ensemble Mean

```bash
# Average 5 models
cdo ensmean model1.nc model2.nc model3.nc model4.nc model5.nc ensemble_mean.nc

# Ensemble standard deviation (spread)
cdo ensstd model1.nc model2.nc model3.nc ensemble_std.nc
```

### Recipe 4: Extract Time Series for a Point

```bash
# Nearest-neighbor interpolation at Islamabad (73.09E, 33.73N)
cdo remapnn,lon=73.09/lat=33.73 ERA5.nc ERA5_Islamabad.nc

# Or using ncks
ncks -d lat,33.73 -d lon,73.09 ERA5.nc ERA5_Islamabad.nc
```

### Recipe 5: Regrid ERA5 to CMIP6 Grid

```bash
# Get target grid from a CMIP6 file
cdo griddes CMIP6_model.nc > cmip6_grid.txt

# Regrid ERA5 to CMIP6 grid (bilinear)
cdo remapbil,cmip6_grid.txt ERA5.nc ERA5_on_CMIP6_grid.nc
```

### Recipe 6: Check and Fix Time Axis

```python
import xarray as xr
import pandas as pd

# Open file
ds = xr.open_dataset("ERA5.nc", decode_times=False)   # Don't auto-decode

# Inspect raw time values
print(ds.time)
print(ds.time.attrs)   # Check units and calendar

# Fix and re-decode
ds["time"].attrs["units"] = "hours since 1900-01-01 00:00:00"
ds["time"].attrs["calendar"] = "standard"
ds = xr.decode_cf(ds)   # Now decode

print(ds.time)   # Should show correct dates
```

---

## 📋 CDO Quick Reference

| Task | Command |
|------|---------|
| File info | `cdo sinfo file.nc` |
| Annual mean | `cdo yearmean in.nc out.nc` |
| Monthly mean | `cdo monmean in.nc out.nc` |
| Climatology | `cdo ymonmean in.nc clim.nc` |
| Anomaly | `cdo ymonsub in.nc clim.nc anom.nc` |
| Select region | `cdo sellonlatbox,W,E,S,N in.nc out.nc` |
| Select variable | `cdo selvar,t2m in.nc out.nc` |
| Select years | `cdo selyear,1979/2023 in.nc out.nc` |
| Select season | `cdo selseas,JJA in.nc out.nc` |
| Convert K→°C | `cdo subc,273.15 in.nc out.nc` |
| Regrid bilinear | `cdo remapbil,r360x180 in.nc out.nc` |
| Merge files | `cdo mergetime *.nc merged.nc` |
| Ensemble mean | `cdo ensmean m1.nc m2.nc ... out.nc` |
| Trend | `cdo trend in.nc intc.nc slope.nc` |

## 📋 NCO Quick Reference

| Task | Command |
|------|---------|
| Extract variable | `ncks -v t2m in.nc out.nc` |
| Spatial subset | `ncks -d lat,S,N -d lon,W,E in.nc out.nc` |
| Rename variable | `ncrename -v old_name,new_name in.nc out.nc` |
| Rename dimension | `ncrename -d longitude,lon in.nc out.nc` |
| Edit attribute | `ncatted -a units,t2m,o,c,"K" in.nc out.nc` |
| Convert K→°C | `ncap2 -s "t2m=t2m-273.15" in.nc out.nc` |
| Difference | `ncdiff future.nc historical.nc change.nc` |

---

## 📚 Resources

- [CDO Documentation](https://code.mpimet.mpg.de/projects/cdo/wiki)
- [CDO Operators Reference](https://code.mpimet.mpg.de/projects/cdo/embedded/cdo.pdf)
- [NCO Documentation](https://nco.sourceforge.net)
- [netCDF4 Python](https://unidata.github.io/netcdf4-python/)
- [CF Conventions](https://cfconventions.org)
- [Unidata NetCDF](https://www.unidata.ucar.edu/software/netcdf/)
