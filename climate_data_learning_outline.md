# Climate Modeling & Climate Data Science — Full Learning Outline
### From Fundamentals to Advanced Practice

---

## MODULE 1 — Foundations (Concepts Before Tools)

### 1.1 Core Climate Science Vocabulary
- Weather vs. climate vs. climate variability
- Radiative forcing (W/m²) and the greenhouse effect
- Energy balance of Earth (albedo, longwave/shortwave radiation)
- Climate sensitivity (Equilibrium Climate Sensitivity, Transient Climate Response)
- Feedback loops (ice-albedo, water vapor, cloud feedback)
- Anthropogenic vs. natural forcing (volcanic, solar, GHG, aerosols)

### 1.2 Types of Climate Models
- **GCM** — General Circulation Model (global, atmosphere+ocean physics)
- **ESM** — Earth System Model (GCM + carbon cycle, vegetation, ice sheets, chemistry)
- **RCM** — Regional Climate Model (dynamical downscaling, e.g. WRF, RegCM)
- **EMIC** — Earth Models of Intermediate Complexity
- Statistical vs. dynamical downscaling (conceptual difference)
- Model resolution, grid types (lat-lon, curvilinear, unstructured mesh)
- Model spin-up and initialization

---

## MODULE 2 — Integrated Assessment Models (IAM)

### 2.1 What IAMs Do
- Coupling economy + energy systems + land use + emissions
- Why IAMs exist: bridge between human decisions and physical climate

### 2.2 Key IAM Frameworks
- REMIND, MESSAGE, GCAM, IMAGE, AIM — who runs what
- Division of labor: each IAM team assigned specific SSP-RCP combos for CMIP6

### 2.3 SSP Framework (Shared Socioeconomic Pathways)
- SSP1 (Sustainability) → SSP5 (Fossil-fueled Development) — the five narratives
- Population, GDP, urbanization, technology assumptions per SSP
- SSP Database (IIASA) — variables: energy, land use, emissions by sector

### 2.4 Forcing Targets (RCP-style)
- What "4.5", "2.6", "7.0", "8.5" mean (W/m² by 2100)
- SSPx-y.z naming convention (e.g. SSP2-4.5, SSP5-8.5)
- Why not every SSP × RCP combination exists/is run
- Harmonization: how raw IAM output becomes standardized emissions input

### 2.5 From IAM Output → Climate Model Input
- Emissions grids (spatial, sectoral, by gas)
- Land-use change files
- How ScenarioMIP consumes IAM output

---

## MODULE 3 — CMIP6 and the MIP Ecosystem

### 3.1 CMIP Basics
- What CMIP is (Coupled Model Intercomparison Project) and why it exists
- CMIP5 vs CMIP6 — what changed
- DECK experiments (baseline diagnostic runs every model must do)

### 3.2 Core CMIP6 Experiments
- `historical` (1850–2014) — observed forcing, model validation baseline
- `piControl` — pre-industrial control run (no forcing changes, for internal variability)
- `abrupt-4xCO2`, `1pctCO2` — idealized sensitivity experiments

### 3.3 ScenarioMIP (Future Runs)
- SSP1-2.6, SSP2-4.5, SSP3-7.0, SSP5-8.5 and others
- 2015–2100 (some extended to 2300)
- How historical + ssp runs get concatenated into one continuous series

### 3.4 Other MIPs You'll Encounter
- **GeoMIP** — Geoengineering Model Intercomparison Project (solar radiation management experiments)
- **ISIMIP** — Inter-Sectoral Impact Model Intercomparison Project (climate → impacts on agriculture, water, health, ecosystems; uses bias-corrected CMIP data)
- **CORDEX** — Coordinated Regional Downscaling Experiment (regional, higher-resolution)
- **DAMIP** — Detection and Attribution MIP
- **LUMIP** — Land-Use MIP
- **AerChemMIP** — Aerosols and Chemistry MIP
- (You don't need to memorize all — know they exist and what question each answers)

### 3.5 Where to Get the Data
- ESGF (Earth System Grid Federation) — main CMIP6 NetCDF portal
- ISIMIP Repository — bias-corrected, impact-ready data
- Copernicus Climate Data Store (CDS)

---

## MODULE 4 — NetCDF Fundamentals

### 4.1 What NetCDF Is and Why Climate Science Uses It
- Self-describing, multi-dimensional array format
- Comparison to CSV/Excel (why gridded time-series data needs something else)

### 4.2 Structure of a NetCDF File
- Dimensions (time, lat, lon, level/plev)
- Variables (the actual data arrays, e.g. `tas`, `pr`, `psl`)
- Attributes (metadata: units, long_name, standard_name)
- Global attributes (model name, experiment ID, institution)

### 4.3 CF Conventions
- Climate and Forecast metadata standard
- Standard variable names (`tas` = near-surface air temperature, `pr` = precipitation)
- Calendars: `standard`, `noleap`, `360_day` — why this matters for date handling

### 4.4 Reading/Inspecting NetCDF
- `ncdump -h file.nc` (header/metadata inspection)
- Python: `xarray`, `netCDF4`, `cftime` for non-standard calendars
- Basic plotting a variable slice

---

## MODULE 5 — CDO (Climate Data Operators)

### 5.1 CDO Basics
- Installing CDO, general syntax pattern: `cdo <operator> infile.nc outfile.nc`
- Chaining operators

### 5.2 File Info & Inspection
- `cdo sinfo`, `cdo info`, `cdo showdate`, `cdo showvar`

### 5.3 Selection Operators
- `cdo selvar`, `cdo sellonlatbox`, `cdo seldate`, `cdo selyear`, `cdo sellevel`

### 5.4 Arithmetic & Statistics
- `cdo add`, `cdo sub`, `cdo mul`, `cdo div`
- `cdo timmean`, `cdo yearmean`, `cdo monmean`, `cdo seasmean`
- `cdo timstd`, `cdo trend`

### 5.5 Regridding / Remapping
- `cdo remapbil`, `cdo remapcon`, `cdo remapnn` — bilinear, conservative, nearest-neighbor
- Why regridding is needed (different models = different grids)

### 5.6 Merging & Concatenation
- `cdo mergetime` — stitching historical + ssp files into one continuous series
- `cdo cat`, `cdo mergegrid`

### 5.7 Ensemble Operations
- `cdo ensmean`, `cdo ensstd` — multi-model ensemble mean/spread

---

## MODULE 6 — Data Gap-Filling (Missing Data Handling)

### 6.1 Why Gaps Occur
- Satellite outages, station dropouts, quality-control masking

### 6.2 Gap-Filling Methods (Conceptual)
- Linear interpolation (temporal)
- Spatial interpolation (nearest-neighbor, kriging, IDW)
- Climatological substitution (fill with long-term mean for that date)
- Regression-based / analog methods
- Machine-learning-based infilling (advanced)

### 6.3 Tools
- `cdo fillmiss`, `cdo setmisstonn`
- Python: `xarray.interpolate_na`, `scipy.interpolate`

---

## MODULE 7 — Bias Correction

### 7.1 Why Bias Correction Is Needed
- Raw model output has systematic biases vs. observations
- Impact models (ISIMIP-style) require bias-corrected input to be usable

### 7.2 Core Methods
- **Delta / Change-factor method** — apply model's projected *change* to observed baseline
- **Quantile Mapping (QM)** — match model's statistical distribution to observed distribution
- **Empirical Quantile Mapping (EQM)** vs **Parametric QM**
- **Trend-preserving bias correction** (used in ISIMIP3BASD) — preserves long-term trend while correcting distribution

### 7.3 Practical Considerations
- Reference/observation dataset choice (e.g. W5E5, ERA5)
- Correcting individual variables (temperature, precipitation) vs. multivariate consistency
- Validation: comparing corrected vs. observed distributions

### 7.4 Tools
- ISIMIP3BASD (the standard bias-adjustment tool used in ISIMIP)
- `cdo` can do simple delta-method corrections; full QM usually needs Python (`xclim`, `bias-correction` packages)

---

## MODULE 8 — Putting It Together: End-to-End Workflow

1. Define scientific question → pick relevant SSP scenario(s)
2. Pull `historical` (1850–2014) + `ssp*` (2015–2100) NetCDF from ESGF
3. Inspect with `ncdump`/`cdo sinfo`
4. Regrid to common resolution across models (`cdo remapbil`)
5. Merge historical + future into continuous series (`cdo mergetime`)
6. Gap-fill if needed (`cdo fillmiss`)
7. Bias-correct against observations (ISIMIP3BASD or quantile mapping)
8. Compute derived stats (`cdo timmean`, ensemble means)
9. Analyze/visualize in Python (`xarray`, `matplotlib`, `cartopy`)

---

## MODULE 9 — Advanced Topics (Once Comfortable)

- Multi-model ensemble uncertainty quantification
- Model weighting schemes (performance-based, independence-based)
- Extreme event indices (ETCCDI indices: heatwaves, dry spells)
- Regional downscaling (CORDEX) vs. global bias-corrected data (ISIMIP)
- Attribution studies (DAMIP) — separating human vs. natural influence
- Geoengineering scenario analysis (GeoMIP)
- Coupling climate output into sector-specific impact models (crop models, hydrology models)

---

### Suggested Learning Order
Module 1 → 2 → 3 (concepts) → Module 4 (NetCDF) → Module 5 (CDO, hands-on) → Module 6 & 7 (data prep) → Module 8 (full workflow) → Module 9 (advanced)

