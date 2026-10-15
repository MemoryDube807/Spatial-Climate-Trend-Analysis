# 🌍 Zambia Climate Trends: Reading 30 Years of Warming and Drying from CMIP6

> **Spatial trend analysis of temperature and rainfall over Zambia using MIROC6 (CMIP6) and district boundaries, 1981-2010.**

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![Jupyter](https://img.shields.io/badge/Notebook-Jupyter-orange)
![Data](https://img.shields.io/badge/Data-CMIP6%20MIROC6-green)
![Stats](https://img.shields.io/badge/Stats-Sen's%20Slope%20%2B%20Mann--Kendall-purple)

---

## 📖 The Story in 60 Seconds

Global averages are comfortable numbers, but nobody farms the globe. A district agricultural officer in Zambia needs to know what is changing **in their area**, **how fast**, and **whether the change is real or just noise**.

This project takes 30 years of daily output from a global climate model, cuts out Zambia using district boundaries, and asks three questions of every grid cell:

1. **How big is the trend?** (Sen's slope, per decade)
2. **Is it statistically real?** (Mann-Kendall test, α = 0.05)
3. **Does it differ across the country?** (spatial maps)

**What I found:** Zambia warmed everywhere in the simulation, nights more reliably than days. Rainfall shows a drier north and a wetter central-west, but almost none of it is statistically confirmed.

---

## 🧭 Table of Contents

- [Key Results](#-key-results)
- [The Maps, Explained](#-the-maps-explained)
- [Method Pipeline](#-method-pipeline)
- [Data](#-data)
- [Getting Started](#-getting-started)
- [Repository Structure](#-repository-structure)
- [Limitations](#-limitations)
- [Skills Demonstrated](#-skills-demonstrated)
- [Author](#-author)

---

## 📊 Key Results

| Variable | Min slope | Max slope | Mean | Median | Significant cells (p < 0.05) |
|---|---|---|---|---|---|
| **Max temperature** (°C/decade) | 0.103 | 0.858 | 0.453 | 0.484 | 53.1% |
| **Min temperature** (°C/decade) | 0.187 | 0.442 | 0.300 | 0.292 | 96.9% |
| **Precipitation** (mm/year per decade) | −89.4 | +67.7 | −16.8 | −28.1 | 3.1% |

All statistics are computed over the **32 grid cells** that fall inside Zambia. Precipitation slopes are in **mm per year, per decade**, because annual totals were trended.

**Takeaways**

- 🔥 **Every cell warmed.** Maximum temperature rose about 0.45 °C per decade, roughly 1.4 °C over 30 years.
- 🌙 **Nights are the most reliable signal.** 97% of cells show a significant minimum-temperature trend.
- 🌧️ **Rainfall is a pattern without a verdict.** Only 1 of 32 cells is significant, which is about what chance alone would produce.

---

## 🗺️ The Maps, Explained

Each map shows the **decadal trend** per grid cell. **Black dots mark cells where the trend is statistically significant (p < 0.05).**

### Study area: Zambia's districts

<p align="center">
  <img src="images/zambia_districts.png" alt="Zambia level 2 administrative districts" width="55%">
</p>

The GADM level-2 district boundaries used to mask the climate grid. Any grid cell outside this border is set to NaN, which leaves 32 valid cells for analysis.

---

### 🔥 Maximum temperature: a hotter country with an east-west divide

<p align="center">
  <img src="images/tasmax_spatial_trend_map.png" alt="Spatial trend of annual maximum temperature in Zambia" width="75%">
</p>

**How to read it:** Darker red means faster warming. The scale is symmetrical around zero, so white would mean no trend.

**What it shows:**

- Warming is strongest in the **east and northeast**, up to **0.86 °C per decade**, around 13-15°S and 30-32°E.
- It is weakest in the **west**, near 0.10 °C per decade.
- Only **53% of cells are significant**, and they cluster in the central-east and north. Daytime highs vary a lot from year to year, which hides the trend in noise in the west.

---

### 🌙 Minimum temperature: warmer nights, far more reliably

<p align="center">
  <img src="images/tasmin_spatial_trend_map.png" alt="Spatial trend of annual minimum temperature in Zambia" width="75%">
</p>

**What it shows:**

- Warming is slower than for maximum temperature (mean **0.30 °C per decade**) but nearly universal. **97% of cells are significant** and even the weakest cell warms by 0.19 °C per decade.
- Night-time lows fluctuate less from year to year, so the signal stands out clearly.
- Because maximum temperature warms faster than minimum, the **gap between daily highs and lows is widening**. Plausible drivers include drying soils or changes in cloud cover, but this analysis cannot confirm either.

---

### 🌧️ Precipitation: a spatial contrast, not a confirmed trend

<p align="center">
  <img src="images/pr_spatial_trend_map.png" alt="Spatial trend of annual precipitation in Zambia" width="75%">
</p>

**How to read it:** Brown means drying and green means wetting.

**What it shows:**

- The **north and northeast dry**, by up to about 89 mm per decade.
- Parts of the **central-west get wetter**, by up to about 68 mm per decade.
- Only **one cell** in the far north passes the significance test. With 32 tests at the 5% level, one or two false positives are expected by chance, so this is **not** convincing evidence of a real trend.
- Relative to the model's own mean rainfall (about 1,600 mm/year), the median trend is roughly **−1.8% per decade**, a weak signal.

---

### 🖱️ Interactive version

`zambia_climate_interactive.html` is a hover-enabled map of maximum temperature trends. Hovering over a cell shows its warming rate and whether the trend is reliable. Download the file and open it in any browser.

---

## 🔬 Method Pipeline

```
Daily NetCDF (tasmax, tasmin, pr)
        │
        ▼
1. Unit standardisation ── Kelvin → °C ; kg m⁻² s⁻¹ → mm/day (× 86,400)
        │
        ▼
2. Spatial masking ─────── Clip to district shapefile (EPSG:4326), outside = NaN
        │
        ▼
3. Temporal aggregation ── Annual mean (temperature) / annual total (rainfall)
        │                  + completeness filter (≥ 300 valid days per year)
        ▼
4. Trend detection ─────── Sen's slope × 10 (per decade) + Mann-Kendall p-value
        │
        ▼
5. Visualisation ───────── Diverging maps, district outlines, significance stippling
```

**Why these choices?**

| Choice | Reason |
|---|---|
| **Sen's slope** | Median of all pairwise slopes, so a single extreme year cannot distort the trend |
| **Mann-Kendall test** | Non-parametric, so no assumption of normally distributed data |
| **Diverging colour maps** (`RdBu_r`, `BrBG`) | Neutral at zero, so warming/cooling and wetting/drying read instantly |
| **Symmetric colour limits** | Keeps white at exactly zero on every map |

**A note on the completeness filter:** the notebook flags 720 year-cells under the 300-day rule. These are the 24 masked cells outside Zambia counted over 30 years (24 × 30 = 720). Every Zambian cell had full coverage in every year, so the filter removed no real data.

---

## 📦 Data

| Dataset | Details |
|---|---|
| **Climate model** | MIROC6, CMIP6 historical run (`r1i1p1f1`), daily, 1981-2010 |
| **Variables** | `tasmax`, `tasmin`, `pr` |
| **Boundaries** | GADM v4.1, Zambia level 2 (`gadm41_ZMB_2.shp`) |
| **Grid** | About 140 km cells, 10 × 10 over the study window, 7 × 8 after clipping |

Expected input filenames:

```
tasmax_day_MIROC6_historical_r1i1p1f1_gn_19810101-20101231.nc
tasmin_day_MIROC6_historical_r1i1p1f1_gn_19810101-20101231.nc
pr_day_MIROC6_historical_r1i1p1f1_gn_19810101-20101231.nc
gadm41_ZMB_2.shp  (plus its .dbf, .shx, .prj companion files)
```

> The data files are not included in this repository. CMIP6 files can be downloaded from the [ESGF portal](https://esgf-node.llnl.gov/projects/cmip6/) and the boundary file from [GADM](https://gadm.org/download_country.html).

---

## 🚀 Getting Started

**1. Clone the repository**

```bash
git clone https://github.com/<your-username>/<your-repo-name>.git
cd <your-repo-name>
```

**2. Install dependencies**

```bash
pip install numpy pandas xarray netCDF4 geopandas rioxarray shapely \
            pymannkendall matplotlib cartopy plotly jupyter
```

**3. Add the data files** to the project folder, using the filenames above.

**4. Run the notebook**

```bash
jupyter notebook cimp6_analysis.ipynb
```

Run the cells from top to bottom. Later cells reuse variables from earlier ones, so skipping around can cause errors.

---

## 📁 Repository Structure

```
.
├── cimp6_analysis.ipynb              # Full analysis, Tasks 1-5
├── images/
│   ├── zambia_districts.png
│   ├── tasmax_spatial_trend_map.png
│   ├── tasmin_spatial_trend_map.png
│   └── pr_spatial_trend_map.png
├── zambia_climate_interactive.html   # Interactive tasmax map (generated by the notebook)
└── README.md
```

---

## ⚠️ Limitations

Honest caveats matter in climate work:

- **One model, one realisation.** MIROC6 `r1i1p1f1` includes its own internal variability. A multi-model ensemble would show how much the regional pattern depends on this single run.
- **Coarse resolution.** Cells are about 140 km wide, so valleys, plateau edges and other local features are smoothed away.
- **Short record.** Thirty years is the minimum for trend work, and rainfall is noisy enough to bury a weak signal.
- **Model rainfall bias.** Simulated mean annual rainfall over Zambia averages about 1,600 mm (range 1,150-2,240 mm), above commonly reported observed values. Because Sen's slope measures change and not absolute amounts, the trend *patterns* remain informative, but absolute rainfall values should not be used for planning without bias correction (for example against CHIRPS).
- **Statistical assumptions.** The standard Mann-Kendall test assumes no autocorrelation, and testing many cells inflates false positives.
- **Simulated, not observed.** Warming rates here are higher than those usually reported from observations for the region, so results should be validated against station or reanalysis data before informing decisions.

---

## 🛠️ Skills Demonstrated

- Multidimensional **NetCDF** handling with `xarray`, including unit conversion and metadata checks
- **Vector-raster integration**: clipping and masking gridded data with district shapefiles (`geopandas`, `rioxarray`)
- **Non-parametric trend analysis**: pixel-wise Sen's slope and Mann-Kendall testing (`pymannkendall`)
- **Cartographic design**: diverging palettes, symmetric scales, significance stippling (`matplotlib`, `cartopy`)
- **Interactive visualisation** with `plotly`
- **Honest interpretation**: separating visible patterns from statistically supported findings

---

## 👤 Author

**Your Name**
📧 your.email@example.com · 🔗 [LinkedIn](https://linkedin.com/in/your-profile) · 🌐 [Portfolio](https://your-portfolio-link)

*This project was completed as part of a spatial climate analysis practical exercise.*

---

## 📚 Data Credits

- MIROC6 model output: MIROC (JAMSTEC, AORI, NIES, R-CCS), distributed via ESGF under CMIP6 terms of use
- Administrative boundaries: [GADM](https://gadm.org) v4.1
