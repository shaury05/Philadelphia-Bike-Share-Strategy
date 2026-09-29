# Philadelphia Bike-Share Strategic Expansion Analysis

This repository contains a strategic data analysis of the Indego bike-share program in Philadelphia. The goal of this project was to analyze 10 quarters of historical trip data (Q1 2024 – Q2 2026) to diagnose system health and develop a data-driven, revenue-maximizing expansion strategy.

##  Tech Stack & Methodology
* **Python (Pandas, Numpy):** Data ingestion, time-series parsing, and quarterly KPI aggregation across millions of rows.
* **SciPy & Statsmodels:** Inferential statistics and hypothesis testing.
* **Matplotlib & Seaborn:** Data visualization.
* **Statistical Concepts Applied:** Welch’s Two-Sample T-Test, 95% Confidence Intervals, and One-Way ANOVA.

##  Key Insights
1. **System Health & Utilization:** Despite a massive 31.7% Year-over-Year surge in billed ride minutes, asset utilization remained stable (scaling safely from ~2,799 to ~3,058 minutes per bike). The physical fleet expanded efficiently to absorb the demand shock without causing stockouts.
2. **The Duration Paradox (T-Test):** While e-bikes are highly popular, hypothesis testing proved that standard pedal bikes generate significantly longer trips. With 95% confidence, standard bikes generate between 1.38 and 1.59 more revenue-generating minutes per trip than e-bikes.
3. **Customer Segmentation (ANOVA):** A One-Way ANOVA test (p < 0.001) revealed extreme behavioral variances. Casual "Walk-up" users average 35.98 minutes per trip, dwarfing the 12-15 minute durations of monthly and annual commuters.

##  Strategic Recommendation
Based on the statistical findings, capital expenditure should pivot toward the "Walk-up" leisure segment. Expanding standard-bike docks along parks, waterfronts, and weekend tourist corridors is mathematically proven to capture higher-duration, higher-margin riders compared to expanding e-bikes in dense commuter hubs.

##  Files in this Repository
* `Shaury_Bike_Share_Strategy.ipynb`: The complete data pipeline, cleaning, and statistical testing code.
* `Shaury_Bike_Share_Strategy.pdf`: The executive presentation summarizing the findings and final business recommendations.
