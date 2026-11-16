# 🚲 Philadelphia Bike-Share (Indego) Strategic Expansion & CapEx Analysis

![Python](https://img.shields.io/badge/Python-3.12%2B-blue?logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Pipeline-150458?logo=pandas&logoColor=white)
![SciPy](https://img.shields.io/badge/SciPy-Hypothesis%20Testing-8CAAE6?logo=scipy&logoColor=white)
![Statsmodels](https://img.shields.io/badge/Statsmodels-Confidence%20Intervals-orange)
![Seaborn](https://img.shields.io/badge/Seaborn-Data%20Viz-4C72B0)

## 📌 Executive Summary & Business Problem
For duration-billed micro-mobility networks, **total trip volume is a vanity metric**—a 5-minute commuter ride and a 45-minute leisure ride look identical on a trip counter but yield vastly different unit economics. 

This project architects an end-to-end telemetry pipeline and inferential statistics framework across **10 quarters of historical trip data (Q1 2024 – Q2 2026)** from Philadelphia's **Indego Bike-Share program** (~3.5M+ raw trip records). The analysis answers three core executive questions to guide Capital Expenditure (CapEx) and fleet deployment:

1. **Growth Velocity:** How fast is revenue-generating ride duration growing when controlling for Philadelphia's extreme winter/summer seasonality?
2. **Operational Health & Capacity:** Is the program expanding too fast (causing station stockouts and excessive wear-and-tear) or too slow (leaving capital assets idle)?
3. **Fleet & Demographic Strategy:** Should CapEx prioritize Electric Bikes (E-bikes) or Standard Pedal Bikes, and which customer tier maximizes billable ride duration?

---

## 📂 Dataset & Data Engineering Pipeline

* **Data Source:** [Indego Philadelphia Official Trip Data Portal](https://www.rideindego.com/about/data/)
* **Scope:** 10 consecutive quarterly CSV files (`indego-trips-2024-q1.csv` through `indego-trips-2026-q2.csv`).
* **Key Telemetry Features Used:**
  * `trip_id` & `bike_id`: Unique identifiers used to track trip volume and active physical fleet size (`nunique`).
  * `start_time` & `duration`: Ride timestamps (parsed into quarterly periods) and billed trip duration in minutes.
  * `bike_type`: Fleet classification (`standard` pedal bikes vs. `electric` assist bikes).
  * `passholder_type`: Customer subscription tier (`Walk-up` casual riders, `Indego30` monthly subscribers, `Indego365` annual subscribers).

### Data Hygiene & Preprocessing Rules
1. **Published Boundary Enforcement:** Filtered out dock-test anomalies, false starts, and lost/stolen bike outliers by enforcing Indego's official data dictionary bounds (`1 min <= duration <= 1440 min` / 24-hour cap).
2. **Memory & Type Optimization:** Handled mixed-type station ID columns across legacy and modern schemas (`low_memory=False`) and cast duration arrays to `float64` after scrubbing `NaN` and infinite (`±inf`) records to prevent numerical overflow during variance calculations.
3. **Seasonality Control:** Standardized growth tracking using **4-Quarter Lagged Year-over-Year (YoY) Percentage Change** (`pct_change(4)`) rather than Quarter-over-Quarter (QoQ) metrics to eliminate weather-induced distortion.

---

## 🔬 Statistical Methodology

Because bike-share trip durations are heavily **right-skewed** (a high concentration of short 8–12 minute commutes with a long tail of multi-hour recreational rides), relying on raw sample averages alone can mislead leadership.

1. **Central Limit Theorem (CLT):** With sample sizes exceeding N > 100,000 per subgroup, the sampling distribution of the sample mean converges to a normal distribution, mathematically justifying parametric inference.
2. **Welch’s Two-Sample T-Test (`scipy.stats.ttest_ind`):** Deployed a one-tailed Welch's T-test (`equal_var=False`) to test the initial hypothesis that E-bikes generate strictly longer trip durations than Standard bikes (`H0: μ_electric <= μ_standard` vs. `Ha: μ_electric > μ_standard`) without assuming equal population variances.
3. **95% Confidence Intervals (`statsmodels.stats.api`):** Constructed an unequal-variance 95% confidence interval for the difference in population means (`μ_standard - μ_electric`) to quantify the exact per-trip revenue margin for executive stakeholders.
4. **One-Way ANOVA (`scipy.stats.f_oneway`):** Executed an Analysis of Variance (F-test) across the three primary passholder tiers (`Walk-up`, `Indego30`, `Indego365`) to test for statistically significant behavioral segmentation (`H0: μ_1 = μ_2 = μ_3`).

---

## 📊 Quantitative Results & Key Insights

### 1. Quarterly Growth Velocity & Asset Utilization Summary
By tracking **Minutes per Active Bike** (`Total Billed Duration / Unique Active Bikes`) as a proxy for fleet strain and dock capacity, the pipeline produced the following YoY diagnostics:

| Quarter | Total Billed Duration (Min) | Total Trips | Active Fleet (Bikes) | Duration YoY Growth (%) | Trips YoY Growth (%) | Utilization (Mins / Bike) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **2025 Q1** | 3,102,536 | 201,588 | 2,278 | +10.19% | +4.03% | 1,361.96 |
| **2025 Q2** | 6,522,150 | 365,760 | 2,330 | +5.65% | -0.63% | 2,799.21 |
| **2025 Q3** | 8,054,846 | 465,464 | 2,460 | +21.25% | +13.97% | 3,274.33 |
| **2025 Q4** | 5,156,273 | 325,361 | 2,450 | +8.74% | +8.77% | 2,104.60 |
| **2026 Q1** | 3,229,645 | 205,908 | 2,757 | +4.10% | +2.14% | 1,171.43 |
| **2026 Q2** | **8,592,561** | **483,327** | **2,809** | **+31.74%** | **+32.14%** | **3,058.94** |

* **Hyper-Growth Diagnosis:** In Q2 2026, total billed ride minutes surged by **31.74% YoY** (from 6.52M to 8.59M minutes), pacing alongside a **32.14% YoY** increase in trip volume.
* **Sustainable Fleet Scaling:** Despite the ~32% demand shock, asset utilization scaled safely from **2,799 mins/bike** in Q2 2025 to **3,058 mins/bike** in Q2 2026 (staying well within historical Q3 peak capacity of 3,274 mins/bike). Indego proactively expanded its active physical fleet by **20.5%** (from 2,330 to 2,809 bikes), absorbing demand without triggering systemic station stockouts.

---

### 2. The Duration Paradox: Standard vs. Electric Bikes
Executive intuition often assumes E-bikes drive longer rides. Statistical testing proved the exact opposite:

| Vehicle Type | Mean Trip Duration | Welch's t-Statistic | p-Value (Ha: μ_ebike > μ_std) | 95% Confidence Interval (μ_std - μ_ebike) |
| :--- | :---: | :---: | :---: | :---: |
| **Standard Pedal Bike** | **17.55 min** | -26.86 | 1.0000 | **+1.38 to +1.59 min / trip** |
| **Electric Assist Bike** | **16.06 min** | — | — | — |

* **Business Insight:** E-bikes enable riders to complete point-to-point commutes faster, *reducing* total billed minutes per trip. With **95% confidence**, standard pedal bikes generate between **1.38 and 1.59 additional billable minutes per trip**—making standard bikes the higher-margin duration asset.

---

### 3. Customer Segmentation (One-Way ANOVA)
Evaluating ride duration across subscription tiers revealed extreme behavioral divergence:

| Passholder Segment | User Profile | Mean Trip Duration | ANOVA F-Statistic | ANOVA p-Value |
| :--- | :--- | :---: | :---: | :---: |
| **Walk-up** | Casual / Leisure / Tourist | **35.98 min** | **19,639.48** | **< 0.0001** |
| **Indego30** | Monthly Commuter Pass | **15.65 min** | — | — |
| **Indego365** | Annual Commuter Pass | **12.15 min** | — | — |

* **Business Insight:** Casual `Walk-up` riders average **35.98 minutes per trip**—more than **2.3x longer** than monthly subscribers (15.65 min) and nearly **3x longer** than annual commuters (12.15 min).

---

## 🚀 Strategic Blueprint & CapEx Recommendations

1. **Pivot CapEx Toward the Leisure Segment:** Halt any fleet strategy that phases out standard pedal bikes in favor of an all-electric fleet. Standard bikes carry lower upfront acquisition/maintenance costs while yielding **+1.38 to +1.59 more billed minutes per ride**.
2. **Geographic Dock Segmentation:**
   * **Recreational & Tourist Corridors (Parks, Waterfronts, Museums):** Heavily weight dock expansion and rebalancing toward **Standard Pedal Bikes** to capture `Walk-up` riders averaging **35.98+ minutes** per trip.
   * **High-Density Transit Hubs (SEPTA Stations, Downtown Commercial Districts):** Deploy **E-bikes** where `Indego30` and `Indego365` commuters prioritize speed and high dock turnover.

---

## 📁 Repository Structure

    ├── Shaury_Bike_Share_Strategy.ipynb   # End-to-end Python pipeline, statistical tests, and Seaborn charts
    ├── Shaury_Bike_Share_Strategy.pdf     # 2-slide executive presentation deck with visual diagnostics
    └── README.md                          # Project documentation and methodology overview

---

## ⚙️ How to Run Locally

1. **Clone the repository:**

       git clone [https://github.com/YOUR-USERNAME/Philadelphia-Bike-Share-Strategy.git](https://github.com/YOUR-USERNAME/Philadelphia-Bike-Share-Strategy.git)
       cd Philadelphia-Bike-Share-Strategy

2. **Install dependencies:**

       pip install pandas numpy scipy statsmodels matplotlib seaborn jupyter

3. **Download the raw telemetry CSVs:**
   Download the Q1 2024 – Q2 2026 quarterly trip datasets from the [Indego Data Portal](https://www.rideindego.com/about/data/) and place the `indego-trips-*.csv` files in the project root directory.

4. **Launch the notebook:**

       jupyter notebook Shaury_Bike_Share_Strategy.ipynb
