# 🌍 Climate Data Science — Master Learning Index
## Complete Roadmap Collection for Python Climate Analysis

> **Author**: Rashid | **Institution**: Quaid-i-Azam University
> **Created**: July 2024 | **Focus**: Climate change, South Asia, Pakistan

---

## 📂 Folder Structure

```
Learning/
│
├── 📄 MASTER_INDEX.md                          ← You are here
│
├── 📁 Module-2_xarray/
│   └── 00_xarray_roadmap.md                   → xarray complete guide
│
├── 📁 Module-3_pandas/
│   └── 00_pandas_roadmap.md                   → Station/tabular data
│
├── 📁 Module-4_matplotlib_cartopy/
│   └── 00_matplotlib_cartopy_roadmap.md       → Maps & projections
│
├── 📁 Module-5_numpy_scipy/
│   └── 00_numpy_scipy_roadmap.md              → Arrays & statistics
│
├── 📁 Module-6_pymannkendall/
│   └── 00_pymannkendall_roadmap.md            → Trend detection
│
├── 📁 Module-7_xclim/
│   └── 00_xclim_roadmap.md                   → Climate indices
│
├── 📁 Module-8_regionmask_geopandas/
│   └── 00_regionmask_geopandas_roadmap.md    → Spatial masking
│
├── 📁 Module-9_salem_rioxarray/
│   └── 00_salem_rioxarray_roadmap.md         → Raster operations
│
├── 📁 Module-10_NetCDF_Commands/
│   └── 00_netcdf_commands_guide.md           → CDO, NCO, ncdump
│
└── 📁 Module-11_NC_to_xarray/
    └── 00_nc_to_xarray_complete_guide.md     → NC files → xarray workflow
```

---

## 🗺️ Learning Path Recommendations

### 🟢 Beginner Path (Start Here)

```
Week 1-2:  Module-11  → NC_to_xarray      (Understand the data format)
Week 3-4:  Module-2   → xarray            (Core tool for gridded data)
Week 5-6:  Module-3   → pandas            (Station data)
Week 7-8:  Module-5   → NumPy + SciPy     (Array math, statistics)
```

### 🟡 Intermediate Path

```
Week 9-10:  Module-4  → matplotlib+cartopy  (Visualize everything)
Week 11-12: Module-6  → pymannkendall       (Trend detection)
Week 13-14: Module-8  → regionmask+geopandas (Extract Pakistan data)
```

### 🔴 Advanced Path

```
Week 15-16: Module-7  → xclim          (Climate indices, bias correction)
Week 17-18: Module-9  → salem+rioxarray (Raster operations, reprojection)
Week 19-20: Module-10 → NetCDF commands (CDO/NCO for batch processing)
```

---

## 📋 Module Summary Table

| Module | Library | Main Purpose | Difficulty | Time |
|--------|---------|-------------|-----------|------|
| [Module-11](Module-11_NC_to_xarray/00_nc_to_xarray_complete_guide.md) | xarray + NetCDF | Loading .nc files, first operations | ⭐⭐ | 2 weeks |
| [Module-2](Module-2_xarray/00_xarray_roadmap.md) | xarray | Gridded climate data (ERA5, CMIP6) | ⭐⭐⭐ | 4 weeks |
| [Module-3](Module-3_pandas/00_pandas_roadmap.md) | pandas | Station data, time series | ⭐⭐ | 2 weeks |
| [Module-4](Module-4_matplotlib_cartopy/00_matplotlib_cartopy_roadmap.md) | matplotlib + cartopy | Spatial maps, projections | ⭐⭐⭐ | 3 weeks |
| [Module-5](Module-5_numpy_scipy/00_numpy_scipy_roadmap.md) | NumPy + SciPy | Array math, distribution fitting | ⭐⭐ | 2 weeks |
| [Module-6](Module-6_pymannkendall/00_pymannkendall_roadmap.md) | pymannkendall | Trend detection (Mann-Kendall) | ⭐⭐ | 1 week |
| [Module-7](Module-7_xclim/00_xclim_roadmap.md) | xclim | Climate indices, bias correction | ⭐⭐⭐⭐ | 3 weeks |
| [Module-8](Module-8_regionmask_geopandas/00_regionmask_geopandas_roadmap.md) | regionmask + geopandas | Spatial masking, district data | ⭐⭐⭐ | 2 weeks |
| [Module-9](Module-9_salem_rioxarray/00_salem_rioxarray_roadmap.md) | salem + rioxarray | Raster ops, CRS handling | ⭐⭐⭐ | 2 weeks |
| [Module-10](Module-10_NetCDF_Commands/00_netcdf_commands_guide.md) | CDO, NCO, ncdump | Command-line NC tools | ⭐⭐⭐ | 2 weeks |

---

## 🔗 How the Libraries Connect

```
NetCDF File (.nc)
        │
        ▼
    CDO / NCO                    ← Command-line preprocessing
(Module-10)
        │
        ▼
    xr.open_dataset()            ← Load into Python
    (Module-11 + Module-2)
        │
        ├──────────────────────────────────────────────────┐
        ▼                                                  ▼
   xarray DataArray                              pandas DataFrame
  (gridded data)                               (station/tabular)
  (Module-2)                                    (Module-3)
        │                                              │
        ├─── regionmask ──► Mask to Pakistan           │
        │    (Module-8)                                │
        │                                              │
        ├─── rioxarray ───► Reproject, clip            │
        │    (Module-9)                                │
        │                                              │
        ├─── xclim ───────► Climate indices            │
        │    (Module-7)     Bias correction            │
        │                                              │
        ├─── NumPy ───────► Array operations     NumPy + SciPy
        │    (Module-5)     Statistics            (Module-5)
        │                        │                     │
        └─── apply_ufunc ─────► pymannkendall         MK test
             (Module-2)          (Module-6)         (Module-6)
                    │
                    ▼
             matplotlib + cartopy
               (Module-4)
             Maps & Visualizations
```

---

## ⚡ Common Task → Module Lookup

| What do you want to do? | Go to |
|--------------------------|-------|
| Open an ERA5/CMIP6 .nc file | Module-11, Module-2 |
| Extract data for Pakistan only | Module-8 (regionmask) |
| Extract data for Punjab province | Module-8 (geopandas + rioxarray) |
| Compute monthly climatology (30-yr normal) | Module-2 (GroupBy) |
| Compute temperature anomalies | Module-2 (GroupBy) |
| Detect temperature trends | Module-6 (pymannkendall) |
| Calculate climate indices (hot days, dry spells) | Module-7 (xclim) |
| Plot a spatial temperature map | Module-4 (cartopy) |
| Plot seasonal maps (4-panel) | Module-4 |
| Analyze station data (PMD) | Module-3 (pandas) |
| Handle large multi-decade files | Module-2 (Dask) |
| Reproject to UTM for area calculations | Module-9 (rioxarray) |
| Process NetCDF from command line | Module-10 (CDO/NCO) |
| Fit GEV to annual maxima | Module-5 (scipy.stats) |
| Create bias-corrected CMIP6 projections | Module-7 (xclim SDBA) |
| Create ensemble mean from multiple models | Module-7 (xclim ensembles) |

---

## 🛠️ Quick Installation

```bash
# Install everything with conda (RECOMMENDED — avoids binary dependency issues)
conda create -n climate python=3.11
conda activate climate

conda install -c conda-forge xarray dask netcdf4 scipy matplotlib cartopy
conda install -c conda-forge geopandas regionmask rioxarray rasterio
conda install -c conda-forge xclim pymannkendall salem
conda install -c conda-forge cdo nco ncdump ncview

# Or with pip (after installing gdal/geos separately):
pip install xarray dask[complete] scipy matplotlib cartopy
pip install geopandas regionmask rioxarray xclim pymannkendall salem
pip install pymannkendall cmocean
```

---

## 📊 Key Data Sources for Pakistan Climate Research

| Dataset | Resolution | Period | Best For |
|---------|-----------|--------|---------|
| ERA5 (ECMWF) | 0.25° | 1940–present | Reanalysis reference data |
| CHIRPS | 0.05° | 1981–present | Precipitation (high res) |
| CMIP6 | 1–2° | 1850–2100 | Future projections |
| CORDEX-SA | 0.22° | 1970–2100 | Regional projections |
| APHRODITE | 0.25° | 1951–2015 | Asian precipitation |
| PMD Station Data | Point | 1950–present | Ground truth |
| NOAA GHCN-D | Point | Varies | Global station network |

---

## 📚 Essential References

1. **xarray**: Hoyer & Hamman (2017), *JOSS*
2. **Mann-Kendall**: Mann (1945) + Hamed & Rao (1998)
3. **xclim**: Logan et al. (2021), *JOSS*
4. **ETCCDI Indices**: Karl et al. (1999)
5. **Cartopy**: Met Office (2010)

---

*All code examples tested with Python 3.11 and current library versions (2024)*
