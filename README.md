# MSCS 634 Lab 1: Data Visualization, Preprocessing, and Statistical Analysis

**Todd Simonson**
MSCS-634: Advanced Big Data and Data Mining
Dr. Satish Penmatsa

## Purpose

This lab applies the first stage of the data mining workflow to a real dataset: exploring it visually, cleaning and preparing it, and summarizing it statistically. The dataset is ten years (January 1, 2015 through December 31, 2024) of daily weather observations from the NOAA Global Historical Climatology Network Daily (GHCN-Daily) station at Colorado Springs Municipal Airport (station USW00093037), 3,653 days in total.

Weather data was chosen because it contains the kinds of problems the lab is meant to practice on: real sensor gaps rather than artificially removed values, real extreme events, a time dimension, and variables measured on very different scales.

## Repository Contents

| Path | Description |
|---|---|
| `MSCS_634_Lab_1.ipynb` | Jupyter Notebook containing all four lab steps |
| `data/USW00093037.csv` | Raw NOAA GHCN-Daily file for the station (full history, 108 columns) |
| `data/cos_weather_cleaned.csv` | 2015-2024 data after unit conversion and missing-value handling, written by the notebook |
| `screenshots/` | Screenshots of the required outputs |
| `requirements.txt` | Python packages needed to run the notebook |

Data source: https://www.ncei.noaa.gov/data/global-historical-climatology-network-daily/access/USW00093037.csv

## How to Run

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook MSCS_634_Lab_1.ipynb
```

Run all cells from the top. The notebook reads `data/USW00093037.csv` and writes `data/cos_weather_cleaned.csv`.

## Screenshots

| Lab step | Screenshot |
|---|---|
| 1. First five rows | [`01_head.png`](screenshots/01_head.png) |
| 2. Line plot | [`02_line_plot.png`](screenshots/02_line_plot.png) |
| 2. Histogram | [`03_histogram.png`](screenshots/03_histogram.png) |
| 2. Box plot by month | [`04_boxplot_month.png`](screenshots/04_boxplot_month.png) |
| 2. Scatter plot | [`05_scatter.png`](screenshots/05_scatter.png) |
| 2. Bar chart | [`06_bar_precip.png`](screenshots/06_bar_precip.png) |
| 2. Pie chart | [`07_pie.png`](screenshots/07_pie.png) |
| 3.1 Missing values: report, before, after | [`08_missing_report.png`](screenshots/08_missing_report.png), [`09_missing_before.png`](screenshots/09_missing_before.png), [`10_missing_after.png`](screenshots/10_missing_after.png) |
| 3.2 Outliers: IQR calculation, outliers, after removal | [`11_iqr_calc.png`](screenshots/11_iqr_calc.png), [`12_outliers.png`](screenshots/12_outliers.png), [`13_after_outliers.png`](screenshots/13_after_outliers.png) |
| 3.3 Data reduction: dimension elimination, sampling | [`14_dimension_reduction.png`](screenshots/14_dimension_reduction.png), [`15_sampling.png`](screenshots/15_sampling.png) |
| 3.4 Scaling (before, after) and discretization | [`16_scaling_before.png`](screenshots/16_scaling_before.png), [`17_scaling_after.png`](screenshots/17_scaling_after.png), [`18_discretization.png`](screenshots/18_discretization.png) |
| 4.1 `.info()` and `.describe()` | [`19_info.png`](screenshots/19_info.png), [`20_describe.png`](screenshots/20_describe.png) |
| 4.2 Central tendency | [`21_central_tendency.png`](screenshots/21_central_tendency.png) |
| 4.3 Dispersion | [`22_dispersion.png`](screenshots/22_dispersion.png) |
| 4.4 Correlation matrix | [`23_correlation.png`](screenshots/23_correlation.png) |

## Key Insights

### Visualizations

- **Line plot:** The data shows ten clear annual temperature cycles. The average gap between the daily high and low is 27.64 °F, typical of a dry, high-elevation station (1,884 m). The coldest day in the window was February 14, 2021, with a high of −0.76 °F.
- **Histogram:** Daily highs do not follow a single bell curve. A full year mixes a cool-season hump (about 45 to 60 °F) with a warm-season hump (about 80 to 90 °F), and a long left tail of cold days gives a skewness of −0.39, pulling the mean (65.47 °F) below the median (66.92 °F).
- **Box plot by month:** July highs have a median of 88.0 °F and a standard deviation of only 6.3 °F, while February's standard deviation is 15.1 °F. Summer outliers are all cool, stormy days, not errors.
- **Scatter plot:** Daily high and low temperature are strongly linearly related (r = 0.90), with the seasons separating cleanly by color.
- **Bar chart:** July (3.44 in) and May (3.10 in) are the wettest months, and December (0.32 in) is the driest. May through August supply about 67% of the 16.49 in mean annual total.
- **Pie chart:** Measurable precipitation fell on 906 of 3,653 days (about 1 day in 4), and summer accounts for 38.2% of those days.

### Statistical Measures

- **Central tendency:** The mean daily high is 65.47 °F and the median is 66.92 °F, while the mode is 87.08 °F. A mode that far from the mean is another sign of the two-season shape seen in the histogram. Precipitation and snowfall both have a median and mode of 0.
- **Dispersion:** Daily high temperature has a range of 101.70 °F, an IQR of 28.98 °F, and a standard deviation of 18.80 °F. Precipitation has an IQR of 0, because at least 75% of days are completely dry.
- **Correlation:** The strongest relationships are daily high vs. daily low (r = 0.90), daily low vs. dew point (r = 0.86), and average wind vs. peak gust (r = 0.72). Sea-level pressure is negatively correlated with peak gust (r = −0.53), consistent with low-pressure storms bringing wind.

## Challenges and Decisions

1. **Units.** GHCN-Daily stores most values in tenths of metric units. For example, the January 1, 2015 high of 24.26 °F is stored as `-43` (−4.3 °C). All analysis columns were converted to °F, inches, mph, and hPa before plotting; otherwise every chart and summary statistic would be in units that are hard to read and, for precipitation and wind, ten times too large.

2. **Missing values needed three different treatments.** The raw 2015-2024 window has 122,175 missing cells, but most of them belong to columns the station never reported.
   - Columns less than 50% populated (13 columns, including `TAVG` and `TSUN`) were dropped with `dropna(axis=1, thresh=...)`.
   - Weather-type flags (`WT01` through `WT22`) were filled with the constant 0, because NOAA leaves the cell blank when an event did not occur. Treating those blanks as unknown, or dropping the columns, would have discarded real information, such as the 827 days with fog.
   - Humidity, dew point, and pressure had real sensor gaps. These were forward filled with a limit of 3 days. The limit matters: the station logged no humidity for all of February 2015, and an unlimited forward fill would have copied the January 31 reading across 28 days. The 25 days still missing after the limited fill were filled with the mean for that calendar month across all ten years.

3. **Choosing a column for IQR outlier detection.** Precipitation was not a usable choice. Its Q1 and Q3 are both 0, so the IQR is 0 and every rainy day would be flagged. Average daily wind speed was used instead (Q1 = 6.93 mph, Q3 = 10.96 mph, upper bound 17.00 mph), which flagged 118 high-wind days. Of these, 68 fall in spring, matching Colorado's spring wind season.

4. **Outliers are real weather, not errors.** The 118 windy days and the 7 extreme cold days flagged on daily high temperature (the IQR rule and the 3-sigma rule flagged the same 7 days) are verifiable events, including the February 2021 cold wave. They were removed only from a separate DataFrame (`df_iqr`) to show the effect of removal. The statistical analysis in Step 4 keeps them, because removing real extremes would understate how variable this climate is.

5. **Library versions.** The class examples use `fillna(method='ffill')`, which has been removed from current pandas. The notebook uses `.ffill()` and `.bfill()` instead.
