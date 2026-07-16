# 🔎 Level 1 & 2: Inspecting NetCDF Files
## Using `ncdump` and `ncview`

Welcome to **Module 10**. Before you write a single line of Python, the very first thing you should do when you download a new climate dataset is look at its structure. 

In the command line (Terminal/Anaconda Prompt), the two most essential tools are **ncdump** (for reading the metadata) and **ncview** (for quickly looking at the actual maps).

---

## 🛠️ 1.1 `ncdump`: Reading the File Header

`ncdump` translates a binary `.nc` file into human-readable text. The most important command you will use is `ncdump -h` (the `-h` stands for "header", which means it only shows the metadata, not the millions of data points).

### The Command
```bash
ncdump -h tasmin_global_daily_gridded_obs_cpc_1979-2024.nc
```

🌍 **Real-Life Climate Use Case:**
You downloaded `tasmin_global_daily_gridded_obs_cpc_1979-2024.nc` but you don't know:
1. What the variable is called (`tmin`, `tasmin`, or `tn`?)
2. What units it is in (Kelvin or Celsius?)
3. Are the latitudes oriented North-to-South or South-to-North?

### Example Output & How to Read It

When you run `ncdump -h tasmin_global_daily_gridded_obs_cpc_1979-2024.nc`, you will see something like this:

```text
netcdf tasmin_global_daily_gridded_obs_cpc_1979-2024 {
dimensions:
    longitude = 1440 ;          // <-- Tells you it's 0.25 degree resolution (360/1440 = 0.25)
    latitude = 721 ;            // <-- 180 degrees / 0.25 = 721 points
    time = 365 ;                // <-- 365 days in this file

variables:
    float longitude(longitude) ;
        longitude:units = "degrees_east" ;
        
    float latitude(latitude) ;
        latitude:units = "degrees_north" ;
        
    int time(time) ;
        time:units = "hours since 1900-01-01 00:00:00.0" ;   // <-- The baseline for the time axis!
        time:calendar = "gregorian" ;
        
    short tasmin(time, latitude, longitude) ;                  // <-- The variable name is tasmin! It's a short (int16).
        tasmin:scale_factor = 0.0014 ;                         // <-- IMPORTANT: Actual value = (data * scale_factor) + add_offset
        tasmin:add_offset = 270.5 ;
        tasmin:_FillValue = -32767 ;                           // <-- What it uses for missing data (like oceans if it's land-only)
        tasmin:units = "K" ;                                   // <-- It's in Kelvin!
        tasmin:long_name = "Minimum temperature at 2 metres" ;
        
// global attributes:
        :Conventions = "CF-1.6" ;
        :history = "2024-07-15 12:00:00 GMT by ERA5 download script" ;
}
```

### Other Useful `ncdump` Commands

- `ncdump -v time file.nc`: Shows the header AND prints out the actual values for the `time` variable. (Great for checking if your dates are correct).
- `ncdump -t -v time file.nc`: The `-t` tells it to translate "hours since 1900" into actual human dates (e.g., "2020-01-01").
- `ncdump -c file.nc`: Shows the header and the values of ALL coordinate variables (time, lat, lon).

---

## 🗺️ 1.2 `ncview`: Quick Visual Inspection

While `ncdump` shows you the text, `ncview` instantly opens a visual map. 

If you just downloaded a file, you should always use `ncview` to verify the data isn't corrupted, completely empty, or inverted.

### The Command
```bash
ncview tasmin_global_daily_gridded_obs_cpc_1979-2024.nc
```

🌍 **Real-Life Climate Use Case:**
You downloaded precipitation data (`pr`) from a CMIP6 model. Before spending hours writing Python code to analyze it, you type `ncview CMIP6_pr.nc`. A small graphical window pops up. 
- You click on `pr`. A map of the globe appears.
- You click the "Play" button. It starts animating the rainfall over time.
- You realize the map is upside down! (The model output had South at the top). You now know you need to fix this later in CDO or Python.

### Key Features of the `ncview` Window:
1. **Variable selection**: Buttons at the top for every variable in the file.
2. **Colormap**: You can click the colorbar to change the colors (e.g., from default to a Rainbow or Red-Blue).
3. **Time controls**: A slider and Play/Pause buttons to scrub through time.
4. **Point inspection**: If you click anywhere on the map, it will pop up a time-series graph for that exact pixel!

*(Note: `ncview` requires an X11 server if you are using Windows Subsystem for Linux (WSL). If you are using native Windows Anaconda, it might be tricky to install, in which case `xarray` plotting is your fallback).*

---

## 🎯 Summary

- Always run `ncdump -h file.nc` to check variable names, units, and `scale_factor`/`_FillValue`.
- Always run `ncview file.nc` to quickly visualize the data and catch blatant errors (like upside-down maps or all-NaN outputs).

---
### 🚦 Next Steps

You now know how to inspect your raw `.nc` files! 

In **Level 3 & 4 (CDO & NCO)**, we will learn how to manipulate these files using the **Climate Data Operators (CDO)** (like taking annual means directly from the terminal). 

After covering the basics of CDO, we will jump into the newly added **Level 5: CDO ECA Indices**, where we will specifically cover how to calculate Extreme Climate Indices like **TXx (Hottest day)**, **TNn (Coldest night)**, **Rx1day (Max 1-day rainfall)**, and **Rx5day**!

Are you ready to move on to CDO basics, or would you like to jump straight to the **ECA Extreme Indices** guide?
