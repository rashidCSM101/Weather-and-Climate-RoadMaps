# 🗺️ matplotlib + cartopy — Complete Learning Roadmap
## Spatial Maps & Projections for Climate Science

---

## 📚 What is cartopy?

cartopy is the standard Python library for **geographic projections and spatial mapping**. Combined with matplotlib, it enables:
- Plotting climate variables on realistic map backgrounds
- Choosing the right projection for your study region
- Adding coastlines, country borders, ocean/land masks
- Overlaying multiple data layers (observations + model)
- Publication-quality spatial figures

---

## 🗺️ Roadmap Overview

```
Level 1: matplotlib Foundations    → Figures, axes, subplots, styling
Level 2: cartopy Basics            → GeoAxes, projections, features
Level 3: Key Projections           → PlateCarree, Robinson, LambertConformal
Level 4: Plotting Climate Data     → Contour, pcolor, scatter on maps
Level 5: Colormaps & Colorbars     → Scientific color conventions
Level 6: Multi-panel Figures       → Seasonal maps, ensemble spreads
Level 7: Advanced Techniques       → Stippling, cross-hatching, quiver plots
Level 8: Publication-Ready Figures → Labels, annotations, saving
```

---

## 📁 LEVEL 1 — matplotlib Foundations

### 1.1 Figure Anatomy

```python
import matplotlib.pyplot as plt
import numpy as np

# --- Basic figure components ---
fig = plt.figure(figsize=(10, 6), dpi=150)         # Create figure
ax = fig.add_subplot(1, 1, 1)                       # Add axes

# OR shorthand:
fig, ax = plt.subplots(figsize=(10, 6), dpi=150)

# Figure elements
ax.set_title("Monthly Mean Temperature", fontsize=14, fontweight="bold", pad=10)
ax.set_xlabel("Year", fontsize=12)
ax.set_ylabel("Temperature (°C)", fontsize=12)
ax.set_xlim(1950, 2024)
ax.set_ylim(-5, 50)
ax.grid(True, linestyle="--", alpha=0.5)
ax.legend(loc="upper left", fontsize=10)

plt.tight_layout()
plt.savefig("figure.png", dpi=300, bbox_inches="tight")
plt.show()
```

### 1.2 Common Plot Types for Climate

```python
# Time series (station data)
ax.plot(years, tmax, color="firebrick", linewidth=1.5, label="Tmax")
ax.plot(years, tmin, color="steelblue", linewidth=1.5, label="Tmin")

# Filled uncertainty
ax.fill_between(years, tmax_low, tmax_high, alpha=0.3, color="firebrick",
                label="90% CI")

# Scatter plot (two variables)
ax.scatter(precip, temp, c=year, cmap="viridis", s=30, alpha=0.7)

# Bar chart (seasonal precipitation)
ax.bar(months, precip, color="steelblue", edgecolor="navy", alpha=0.8)

# Box plots (interannual variability)
ax.boxplot([jan_data, feb_data, mar_data], labels=["Jan", "Feb", "Mar"])

# Histogram (distribution)
ax.hist(tmax, bins=30, density=True, color="salmon", alpha=0.7)

# Heatmap-style (annual cycle)
im = ax.imshow(annual_cycle_matrix, aspect="auto", cmap="RdBu_r",
               vmin=-3, vmax=3)
plt.colorbar(im, label="Anomaly (°C)")
```

### 1.3 Subplots & Multi-panel

```python
# --- Grid of subplots ---
fig, axes = plt.subplots(2, 2, figsize=(12, 8))
axes[0, 0].plot(...)      # Top-left
axes[0, 1].plot(...)      # Top-right
axes[1, 0].plot(...)      # Bottom-left
axes[1, 1].plot(...)      # Bottom-right

# --- Unequal subplots (for maps with colorbars) ---
fig = plt.figure(figsize=(14, 10))
ax1 = fig.add_subplot(2, 2, 1)
ax2 = fig.add_subplot(2, 2, 2)
ax3 = fig.add_subplot(2, 1, 2)   # Wide bottom panel

# --- GridSpec for flexible layouts ---
import matplotlib.gridspec as gridspec
fig = plt.figure(figsize=(14, 10))
gs = gridspec.GridSpec(2, 3, hspace=0.4, wspace=0.3,
                       width_ratios=[1, 1, 0.05])  # Last col = colorbar
ax1 = fig.add_subplot(gs[0, 0])
ax2 = fig.add_subplot(gs[0, 1])
cax = fig.add_subplot(gs[:, 2])    # Colorbar spanning both rows
```

---

## 📁 LEVEL 2 — cartopy Basics

### 2.1 Setting Up GeoAxes

```python
import cartopy.crs as ccrs
import cartopy.feature as cfeature
import matplotlib.pyplot as plt
import numpy as np

# --- Basic map ---
fig, ax = plt.subplots(
    figsize=(12, 6),
    subplot_kw={"projection": ccrs.PlateCarree()}    # Set projection HERE
)

# Add geographic features
ax.add_feature(cfeature.COASTLINE, linewidth=0.8, edgecolor="black")
ax.add_feature(cfeature.BORDERS, linewidth=0.5, linestyle="--", edgecolor="gray")
ax.add_feature(cfeature.OCEAN, color="lightblue", alpha=0.3)
ax.add_feature(cfeature.LAND, color="lightgray", alpha=0.3)
ax.add_feature(cfeature.LAKES, color="lightblue", alpha=0.5)
ax.add_feature(cfeature.RIVERS, linewidth=0.3, edgecolor="steelblue")

# Gridlines and labels
gl = ax.gridlines(crs=ccrs.PlateCarree(), draw_labels=True,
                  linewidth=0.5, color="gray", alpha=0.5, linestyle="--")
gl.top_labels = False
gl.right_labels = False
gl.xlabel_style = {"size": 10}
gl.ylabel_style = {"size": 10}

# Set extent [West, East, South, North]
ax.set_extent([60, 80, 20, 40], crs=ccrs.PlateCarree())  # Pakistan/South Asia

plt.show()
```

### 2.2 cartopy Feature Resolution

```python
import cartopy.feature as cfeature

# Resolution options: "110m" (coarse), "50m" (medium), "10m" (fine)
ax.add_feature(cfeature.COASTLINE.with_scale("10m"), linewidth=0.8)
ax.add_feature(cfeature.BORDERS.with_scale("10m"), linewidth=0.5)
ax.add_feature(cfeature.STATES.with_scale("10m"), linewidth=0.3)  # US states

# Custom Natural Earth features
from cartopy.feature import NaturalEarthFeature
provinces = NaturalEarthFeature(
    category="cultural",
    name="admin_1_states_provinces_lines",
    scale="10m",
    facecolor="none",
    edgecolor="gray"
)
ax.add_feature(provinces, linewidth=0.4)
```

---

## 📁 LEVEL 3 — Projections Deep Dive

### 3.1 PlateCarree (Equirectangular)

```python
# Best for: Global data, data already in lat/lon degrees
# Distortion: Severe at poles (Greenland looks huge)
# When to use: Quick visualization, regional maps near equator

proj = ccrs.PlateCarree()
fig, ax = plt.subplots(subplot_kw={"projection": proj}, figsize=(12, 5))
ax.set_global()   # Show full Earth
ax.add_feature(cfeature.COASTLINE)

# IMPORTANT: When plotting data, specify data transform!
ax.contourf(
    lon, lat, data,          # Data coordinates
    transform=ccrs.PlateCarree(),    # Data's coordinate system
    cmap="RdBu_r"
)
# Always use transform=ccrs.PlateCarree() when data is in lon/lat degrees
```

### 3.2 Robinson

```python
# Best for: Global overview maps (world maps)
# Properties: Compromise projection — neither area nor shape perfectly preserved
# When to use: World maps in publications, climatological overviews

proj = ccrs.Robinson()
fig, ax = plt.subplots(subplot_kw={"projection": proj}, figsize=(12, 6))
ax.set_global()
ax.add_feature(cfeature.COASTLINE, linewidth=0.6)
ax.add_feature(cfeature.LAND, color="lightgray")
ax.add_feature(cfeature.OCEAN, color="#D6EAF8")

# Plot global temperature anomaly
cf = ax.contourf(
    lon, lat, temp_anomaly,
    transform=ccrs.PlateCarree(),     # Data is in lat/lon — ALWAYS specify this!
    levels=np.linspace(-3, 3, 31),
    cmap="RdBu_r",
    extend="both"
)
plt.colorbar(cf, ax=ax, orientation="horizontal", pad=0.05,
             label="Temperature Anomaly (°C)", shrink=0.8)
ax.set_title("Global Temperature Anomaly — Robinson Projection", fontsize=13)
```

### 3.3 LambertConformal

```python
# Best for: Mid-latitude regional maps (North America, Europe, Asia)
# Properties: Preserves local shapes, good for South Asia
# When to use: Pakistan, India, regional climate studies

proj = ccrs.LambertConformal(
    central_longitude=70,       # Center of your domain (Pakistan ≈ 70°E)
    central_latitude=30,        # Center of your domain (Pakistan ≈ 30°N)
    standard_parallels=(20, 40) # Two standard parallels bracketing domain
)

fig, ax = plt.subplots(subplot_kw={"projection": proj}, figsize=(10, 8))

# Define extent in PlateCarree (then cartopy converts)
ax.set_extent([55, 85, 20, 45], crs=ccrs.PlateCarree())

ax.add_feature(cfeature.COASTLINE.with_scale("10m"), linewidth=0.8)
ax.add_feature(cfeature.BORDERS.with_scale("10m"), linewidth=0.6, edgecolor="black")
ax.add_feature(cfeature.LAND, color="#F8F9F0")
ax.add_feature(cfeature.OCEAN, color="#D6EAF8")

cf = ax.contourf(
    lon, lat, temp_data,
    transform=ccrs.PlateCarree(),     # Always!
    levels=20,
    cmap="hot_r",
    extend="both"
)
plt.colorbar(cf, ax=ax, label="Temperature (°C)", shrink=0.85)
ax.set_title("Pakistan Temperature — LambertConformal", fontsize=13)
```

### 3.4 Other Useful Projections

```python
# Mercator — Good for navigation, bad for poles
ccrs.Mercator()

# Orthographic — Globe view, beautiful for presentations
ccrs.Orthographic(central_longitude=70, central_latitude=30)

# Polar Stereographic — Arctic/Antarctic analysis
ccrs.NorthPolarStereo()
ccrs.SouthPolarStereo()

# Mollweide — Equal area world map
ccrs.Mollweide()

# Albers Equal Area — Best for area calculations
ccrs.AlbersEqualArea(central_longitude=70, central_latitude=30)

# Comparison: use the right projection!
projections = {
    "Global overview": ccrs.Robinson(),
    "Regional (Pakistan)": ccrs.LambertConformal(central_longitude=70, central_latitude=30),
    "Polar regions": ccrs.NorthPolarStereo(),
    "Equal area global": ccrs.Mollweide(),
    "Quick/simple": ccrs.PlateCarree(),
}
```

---

## 📁 LEVEL 4 — Plotting Climate Data on Maps

### 4.1 Contour Fills (Temperature Maps)

```python
import xarray as xr
import numpy as np

da = xr.open_dataset("ERA5_t2m.nc")["t2m"].mean("time") - 273.15

lon = da.lon.values
lat = da.lat.values
data = da.values

fig, ax = plt.subplots(
    figsize=(12, 6),
    subplot_kw={"projection": ccrs.Robinson()}
)
ax.set_global()
ax.add_feature(cfeature.COASTLINE, linewidth=0.5)
ax.add_feature(cfeature.BORDERS, linewidth=0.3)

# contourf: filled contours (smooth appearance)
cf = ax.contourf(
    lon, lat, data,
    transform=ccrs.PlateCarree(),
    levels=np.linspace(-30, 50, 41),
    cmap="RdYlBu_r",
    extend="both"                  # Show out-of-range values
)

# Contour lines (optional overlay)
cs = ax.contour(
    lon, lat, data,
    transform=ccrs.PlateCarree(),
    levels=[0, 20, 40],
    colors="black",
    linewidths=0.5
)
ax.clabel(cs, fmt="%d°C", fontsize=8)

# Colorbar
cbar = plt.colorbar(cf, ax=ax, orientation="horizontal",
                    pad=0.05, shrink=0.8, aspect=40)
cbar.set_label("Mean Annual Temperature (°C)", fontsize=11)
cbar.ax.tick_params(labelsize=9)

ax.set_title("ERA5 Annual Mean Temperature", fontsize=14, fontweight="bold")
plt.tight_layout()
plt.savefig("global_temperature.png", dpi=300, bbox_inches="tight")
```

### 4.2 pcolormesh (Faster for Large Grids)

```python
# pcolormesh is faster than contourf for high-resolution grids
fig, ax = plt.subplots(subplot_kw={"projection": ccrs.PlateCarree()}, figsize=(12, 6))

pm = ax.pcolormesh(
    lon, lat, data,
    transform=ccrs.PlateCarree(),
    cmap="RdBu_r",
    vmin=-3, vmax=3,               # Explicit range
    shading="auto"
)
plt.colorbar(pm, ax=ax, label="Anomaly (°C)", extend="both")
ax.coastlines()
ax.set_global()
```

### 4.3 Station Data (Scatter on Map)

```python
# Plot point observations from weather stations
fig, ax = plt.subplots(
    subplot_kw={"projection": ccrs.LambertConformal(central_longitude=70)},
    figsize=(10, 8)
)
ax.set_extent([60, 78, 23, 38], crs=ccrs.PlateCarree())
ax.add_feature(cfeature.COASTLINE.with_scale("10m"))
ax.add_feature(cfeature.BORDERS.with_scale("10m"))

sc = ax.scatter(
    station_lon,              # Station longitude
    station_lat,              # Station latitude
    c=station_trend,          # Color = trend value
    s=80,                     # Marker size
    cmap="RdBu_r",
    vmin=-0.05, vmax=0.05,
    transform=ccrs.PlateCarree(),    # ALWAYS specify!
    zorder=5,                  # Above other features
    edgecolors="black",
    linewidths=0.5
)
plt.colorbar(sc, ax=ax, label="Temperature Trend (°C/year)", shrink=0.8)

# Add station labels
for i, name in enumerate(station_names):
    ax.text(station_lon[i]+0.2, station_lat[i]+0.2, name,
            transform=ccrs.PlateCarree(), fontsize=7)
```

### 4.4 Stippling for Statistical Significance

```python
# Stipple dots where trend is NOT significant (p > 0.05)
insig_mask = pvalue > 0.05

# Create stipple grid
lon_2d, lat_2d = np.meshgrid(lon, lat)
insig_lon = lon_2d[insig_mask]
insig_lat = lat_2d[insig_mask]

ax.scatter(
    insig_lon[::3], insig_lat[::3],   # Subsample (every 3rd) for clarity
    transform=ccrs.PlateCarree(),
    s=0.5, c="black", marker=".",
    alpha=0.6
)
# Significant regions show clean color; insignificant = dotted
```

---

## 📁 LEVEL 5 — Colormaps & Scientific Color Conventions

### 5.1 Choosing the Right Colormap

```python
# DIVERGING (for anomalies: negative ↔ zero ↔ positive)
"RdBu_r"         # Temperature anomaly (red=warm, blue=cold)
"BrBG"           # Precipitation anomaly (brown=dry, green=wet)
"PuOr"           # Wind anomaly
"RdYlGn_r"       # Drought (red=dry, green=wet)

# SEQUENTIAL (for absolute values with one direction)
"YlOrRd"         # Temperature (cool=yellow, hot=red)
"Blues"          # Precipitation amount
"viridis"        # Default scientific (perceptually uniform)
"plasma"         # Alternative to viridis
"hot_r"          # Temperature (white=hot)

# QUALITATIVE (for categories)
"tab10"          # 10 distinct colors for station IDs
"Set1"           # Bold colors for small datasets
"Paired"         # Paired colors for comparison

# cmocean (ocean/climate specific)
import cmocean
"cmocean.cm.thermal"     # Temperature
"cmocean.cm.rain"        # Precipitation
"cmocean.cm.speed"       # Wind speed
"cmocean.cm.delta"       # Anomalies (diverging)

# Example: symmetric diverging colormap centered at 0
from matplotlib.colors import TwoSlopeNorm
norm = TwoSlopeNorm(vmin=-3, vcenter=0, vmax=5)  # Different scales
cf = ax.contourf(lon, lat, data, transform=ccrs.PlateCarree(),
                 norm=norm, cmap="RdBu_r")
```

### 5.2 Custom Colorbars

```python
# Horizontal colorbar below map
cbar = plt.colorbar(cf, ax=ax, orientation="horizontal",
                    pad=0.05, shrink=0.8, aspect=40,
                    extend="both")   # "min", "max", "both", "neither"
cbar.set_label("Temperature Anomaly (°C)", fontsize=11, labelpad=5)
cbar.ax.tick_params(labelsize=9)
cbar.set_ticks([-3, -2, -1, 0, 1, 2, 3])

# Shared colorbar for multi-panel
fig, axes = plt.subplots(1, 3, subplot_kw={"projection": ccrs.Robinson()},
                         figsize=(15, 4))
# Plot all panels...
plt.subplots_adjust(right=0.9)
cax = fig.add_axes([0.92, 0.15, 0.02, 0.7])   # [left, bottom, width, height]
plt.colorbar(cf, cax=cax, label="°C")
```

---

## 📁 LEVEL 6 — Multi-Panel Seasonal Maps

### 6.1 Four-Panel Seasonal Map

```python
seasons = ["DJF", "MAM", "JJA", "SON"]
season_titles = {
    "DJF": "Winter (Dec-Jan-Feb)",
    "MAM": "Spring (Mar-Apr-May)",
    "JJA": "Summer (Jun-Jul-Aug)",
    "SON": "Autumn (Sep-Oct-Nov)"
}

fig, axes = plt.subplots(
    2, 2,
    figsize=(14, 9),
    subplot_kw={"projection": ccrs.LambertConformal(
        central_longitude=70, central_latitude=30)},
    gridspec_kw={"hspace": 0.1, "wspace": 0.05}
)

levels = np.linspace(-3, 3, 31)
cmap = "RdBu_r"

for ax, season in zip(axes.flat, seasons):
    da_season = da.sel(season=season)
    cf = ax.contourf(
        lon, lat, da_season.values,
        transform=ccrs.PlateCarree(),
        levels=levels, cmap=cmap, extend="both"
    )
    ax.set_extent([55, 85, 20, 45], crs=ccrs.PlateCarree())
    ax.add_feature(cfeature.COASTLINE.with_scale("10m"), linewidth=0.5)
    ax.add_feature(cfeature.BORDERS.with_scale("10m"), linewidth=0.4)
    ax.set_title(season_titles[season], fontsize=11, fontweight="bold")

# Shared colorbar
fig.subplots_adjust(bottom=0.1)
cax = fig.add_axes([0.15, 0.05, 0.7, 0.025])
plt.colorbar(cf, cax=cax, orientation="horizontal",
             label="Temperature Anomaly (°C)", extend="both")

fig.suptitle("Seasonal Temperature Anomaly over South Asia", fontsize=14,
             fontweight="bold", y=1.01)
plt.savefig("seasonal_map.png", dpi=300, bbox_inches="tight")
```

---

## 📁 LEVEL 7 — Advanced Techniques

### 7.1 Wind Vectors (Quiver Plots)

```python
# Subsample for readable arrows
step = 5
ax.quiver(
    lon[::step], lat[::step],
    u[::step, ::step], v[::step, ::step],
    transform=ccrs.PlateCarree(),
    scale=200,          # Arrow scale (smaller = larger arrows)
    width=0.002,        # Arrow width
    color="black",
    alpha=0.7
)

# Streamlines (more aesthetic)
ax.streamplot(
    lon, lat, u, v,
    transform=ccrs.PlateCarree(),
    density=2, color="navy",
    linewidth=0.8, arrowsize=0.5
)
```

### 7.2 Cross-Hatching for Masking

```python
# Hatch over ocean/insignificant areas
ax.contourf(
    lon, lat, pvalue,
    transform=ccrs.PlateCarree(),
    levels=[0.05, 1.0],
    hatches=["///"],
    colors="none",
    linewidth=0.3
)
```

---

## 📁 LEVEL 8 — Publication-Ready Settings

### 8.1 Figure Style Setup

```python
import matplotlib as mpl
import matplotlib.pyplot as plt

# Global style settings
mpl.rcParams.update({
    "font.family": "DejaVu Sans",
    "font.size": 11,
    "axes.titlesize": 13,
    "axes.labelsize": 11,
    "xtick.labelsize": 9,
    "ytick.labelsize": 9,
    "figure.dpi": 150,
    "savefig.dpi": 300,
    "savefig.bbox": "tight",
    "axes.spines.top": False,
    "axes.spines.right": False,
})

# Publication-quality save
plt.savefig(
    "figure_01.pdf",          # PDF for journals
    dpi=300,
    bbox_inches="tight",
    format="pdf",
    transparent=False,
    facecolor="white"
)
# Also save PNG for submission
plt.savefig("figure_01.png", dpi=300, bbox_inches="tight")
```

---

## 📋 Projection Quick Reference

| Projection | Code | Best For |
|-----------|------|---------|
| PlateCarree | `ccrs.PlateCarree()` | Simple lat/lon grids, regional maps |
| Robinson | `ccrs.Robinson()` | World maps, global overview |
| LambertConformal | `ccrs.LambertConformal(central_longitude=70)` | Mid-latitude regional (South Asia) |
| Mercator | `ccrs.Mercator()` | Navigation, near-equator |
| Mollweide | `ccrs.Mollweide()` | Equal-area world maps |
| Orthographic | `ccrs.Orthographic(70, 30)` | Globe views, presentations |
| NorthPolarStereo | `ccrs.NorthPolarStereo()` | Arctic studies |

---

## 📚 Resources

- [cartopy Documentation](https://scitools.org.uk/cartopy/docs/latest/)
- [cartopy Projection List](https://scitools.org.uk/cartopy/docs/latest/reference/projections.html)
- [matplotlib Documentation](https://matplotlib.org/stable/)
- [cmocean Colormaps](https://matplotlib.org/cmocean/)
- [Scientific Colormaps (SciViz)](https://www.fabiocrameri.ch/colourmaps/)
