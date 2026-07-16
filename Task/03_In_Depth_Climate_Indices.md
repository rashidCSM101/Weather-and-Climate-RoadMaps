# 📖 In-Depth Guide to Climate Extreme Indices
## Understanding the ETCCDI Temperature & Precipitation Extremes

When analyzing climate data, looking at the "average" temperature or rainfall is often not enough. A region's average temperature might only increase by 1°C over 30 years, but the *extremes* (the hottest days or heaviest rainfalls) might become exponentially worse. 

To standardize how scientists measure these extremes, the World Meteorological Organization created the **ETCCDI** (Expert Team on Climate Change Detection and Indices). 

Here is an in-depth look at the most critical indices for temperature and precipitation.

---

## 🔥 1. Absolute Maximums (The Hottest Peaks)

### **TXx (Hottest Day of the Year)**
- **What is it?** The absolute maximum value of daily maximum temperature (`tasmax`). 
- **What is the work?** It throws away 364 days and keeps the single highest daytime temperature of the year. *(CDO operator: `yearmax` on `tasmax`)*.
- **Why & Where we use it:** Measures peak heatwaves. Used for designing power grids (predicting peak AC usage) and public health (heatstroke warnings).

### **TNx (Warmest Night of the Year)**
- **What is it?** The absolute maximum value of daily minimum temperature (`tasmin`).
- **What is the work?** It keeps the single highest nighttime temperature of the year. *(CDO operator: `yearmax` on `tasmin`)*.
- **Why & Where we use it:** Extremely important for human health. When TNx is too high, the human body cannot recover from daytime heat while sleeping. High TNx values are the leading cause of severe heat-related mortality in cities without widespread air conditioning.

---

## ❄️ 2. Absolute Minimums (The Coldest Troughs)

### **TNn (Coldest Night of the Year)**
- **What is it?** The absolute minimum value of daily minimum temperature (`tasmin`).
- **What is the work?** It extracts the absolute lowest nightly temperature of the year. *(CDO operator: `yearmin` on `tasmin`)*.
- **Why & Where we use it:** Used to track the loss of severe cold spells. Agronomists map TNn to ensure crops (like apples and cherries) can survive winter freezes. If TNn becomes too warm, agricultural pests survive the winter and cause plagues the next summer.

### **TXn (Coldest Day of the Year)**
- **What is it?** The absolute minimum value of daily maximum temperature (`tasmax`).
- **What is the work?** It keeps the lowest daytime temperature recorded all year. *(CDO operator: `yearmin` on `tasmax`)*.
- **Why & Where we use it:** Used to measure the severity of winter daytime cold waves. Important for predicting energy demand for winter daytime heating.

---

## 📈 3. Percentile Indices (Relative Extremes)

Absolute indices (like TXx) just give you a temperature (e.g., 48°C). Percentile indices measure **how often** temperatures exceed what is "normal" for that specific location.

### **TX90p (Warm Days) & TN90p (Warm Nights)**
- **What is it?** The percentage of days (TX90p) or nights (TN90p) where the temperature is greater than the 90th percentile of the historical baseline.
- **What is the work?** CDO first calculates the "normal" top 10% hottest temperatures for a historical baseline period (e.g., 1961-1990). Then, it counts how many days in the current year exceeded that old boundary. *(CDO operators: `eca_tx90p` and `eca_tn90p`)*.
- **Why & Where we use it:** It tells us if extreme heat is becoming *more frequent*. Even if the TXx (absolute peak) hasn't changed much, a high TX90p means a city is suffering through 40 extreme heat days a year instead of the historical 10.

### **TX10p (Cold Days) & TN10p (Cold Nights)**
- **What is it?** The percentage of days (TX10p) or nights (TN10p) where the temperature drops below the 10th percentile of the historical baseline.
- **What is the work?** CDO calculates the "normal" coldest 10% of days from history, and counts how many days in the current year fell below that line. *(CDO operators: `eca_tx10p` and `eca_tn10p`)*.
- **Why & Where we use it:** In a rapidly warming world, we expect these numbers to drop significantly. Tracking TN10p is crucial for understanding the shrinking duration and severity of winter.

---

## 🌧️ 4. Precipitation Extremes

### **Rx1day (Maximum 1-Day Precipitation)**
- **What is it?** The highest amount of rainfall that fell in a single 24-hour period.
- **What is the work?** It ignores total annual rain, and finds the single most intense downpour of the year. *(CDO operator: `eca_rx1day`)*.
- **Why & Where we use it:** Climate change causes rain to fall in violent bursts. Rx1day is used by urban planners to design storm drains and sewers, and by disaster management agencies to map flash flood and landslide risks.
