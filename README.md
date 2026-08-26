# Optimizing NYC Yellow Taxi Services: Demand Analysis and Revenue Prediction

A large-scale data analytics project investigating **New York City Yellow Taxi demand patterns, weather effects, geospatial demand distribution, and trip revenue prediction** using approximately **19.8 million taxi trip records** from Autumn 2022 and 2023.

The project combines NYC Taxi & Limousine Commission trip data with hourly weather observations and applies **PySpark-based data preprocessing, exploratory data analysis, geospatial visualisation, and regression modelling** to derive practical insights for taxi operations.

📄 **[View the full project report](./report_pdf_version.pdf)**

---

## Project Motivation

Yellow taxis are a major component of New York City's transportation network, particularly in Manhattan.

As taxi demand varies substantially by **time of day, day of week, location, trip characteristics, and weather conditions**, understanding historical demand patterns can help drivers and taxi companies make better operational decisions.

This project uses Autumn 2022 and 2023 trip data to investigate:

* **When is taxi demand highest?**
* **Where is peak-hour demand concentrated?**
* **How do weather and trip characteristics relate to taxi demand?**
* **Can trip-level characteristics be used to predict total trip revenue?**
* **How can historical patterns support better vehicle deployment and working-hour decisions?**

The overall goal is to transform large-scale trip records into practical insights that can support **demand planning, operational efficiency, and revenue optimisation**.

---

## Dataset

### NYC Yellow Taxi Trip Records

The primary dataset comes from the **New York City Taxi & Limousine Commission (TLC)**.

The analysis covers:

* **1 September 2022 – 30 November 2022**
* **1 September 2023 – 30 November 2023**

The combined taxi dataset contains approximately:

**19,820,617 trip records**

Source:

https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page

---

### NYC Weather Data

Hourly weather observations are obtained from **Visual Crossing Weather Data** for the same periods.

The external dataset contains approximately:

**4,370 hourly weather records**

Weather variables include:

* precipitation;
* feels-like temperature;
* wind speed; and
* visibility.

Source:

https://www.visualcrossing.com/weather-data

Because detailed weather observations for every taxi zone were not available, Manhattan weather conditions are used as an approximation for broader NYC conditions within each hour.

---

## Project Workflow

```text
NYC Yellow Taxi Data
        │
        │
        ├─────────────── NYC Weather Data
        │                        │
        ▼                        ▼
     Data Cleaning        Weather Processing
        │                        │
        └──────────┬─────────────┘
                   ▼
          Hour-Level Data Merge
                   │
                   ▼
          Feature Engineering
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
       EDA     Geospatial   Modelling
                Analysis
        │          │          │
        └──────────┴──────────┘
                   ▼
       Operational Recommendations
```

---

## Data Preprocessing

Large-scale preprocessing is primarily implemented using **PySpark**.

### Data Cleaning

The preprocessing pipeline includes:

* removing variables not required for demand and revenue analysis;
* handling missing `passenger_count` and `RatecodeID` values;
* removing duplicate records;
* converting variables into appropriate data types;
* filtering records outside the target Autumn 2022/2023 periods; and
* removing unrealistic or anomalous trip records.

---

### External Data Integration

Taxi and weather datasets are aligned at the **hourly level**.

Datetime variables are standardised and used to associate each taxi trip with the corresponding hourly weather conditions.

This creates a combined dataset containing both:

**Trip characteristics + Weather conditions**

for downstream analysis.

---

## Feature Engineering

Several features are constructed for demand and revenue analysis.

### Trip Duration

Trip duration is calculated from pickup and drop-off timestamps.

```text
Trip Duration = Drop-off Time - Pickup Time
```

### Total Revenue

Total trip revenue combines recorded trip revenue with tips.

```text
Total Revenue = Order Revenue + Tip Amount
```

### Temporal Features

Pickup timestamps are used to extract:

* hour of day;
* day of week;
* peak-hour indicator; and
* Autumn year.

These features support hourly and weekly demand analysis.

### Peak-Hour Aggregation

For peak-hour analysis, trip records are aggregated with variables including:

* trip count;
* precipitation;
* feels-like temperature;
* wind speed;
* visibility;
* trip duration; and
* trip distance.

---

## Outlier Handling

Domain-based and statistical rules are applied to identify unrealistic trips.

### Passenger Count

Trips with invalid passenger counts are removed.

### Trip Distance

Very short trips likely associated with cancelled or incorrectly recorded journeys are excluded.

Extreme trip-distance observations are additionally identified using **Z-score-based outlier detection**.

### Trip Duration

Trips lasting more than **6 hours** are treated as anomalous records.

### Revenue

Trips with recorded revenue below the valid minimum fare threshold are removed, followed by additional statistical outlier filtering.

---

## Exploratory Data Analysis

The EDA examines demand patterns across:

* year;
* day of week;
* hour of day;
* peak commuting periods; and
* weather/trip characteristics.

---

## Key Demand Patterns

### Weekly Demand

Taxi demand is generally stronger during weekdays.

In both analysed years, **Wednesday and Thursday** are among the highest-demand days.

---

### Hourly Demand

The 2022 and 2023 datasets show similar daily demand profiles.

Two major high-demand periods are identified:

**Morning Peak**

```text
7:00 AM – 9:00 AM
```

**Evening Peak**

```text
5:00 PM – 7:00 PM
```

Demand falls substantially during the early morning and rises rapidly after approximately 6:00 AM.

---

## Peak-Hour Correlation Analysis

Several relationships were observed between peak-hour demand and trip/weather characteristics.

| Feature                | Correlation with Peak-Hour Trip Count |
| ---------------------- | ------------------------------------: |
| Average Trip Distance  |                             **-0.91** |
| Average Trip Duration  |                             **+0.72** |
| Precipitation          |                             **+0.32** |
| Visibility             |                             **-0.20** |
| Wind Speed             |                             **-0.18** |
| Feels-Like Temperature |                            **-0.027** |

The strongest observed relationships suggest that peak-demand periods are associated with **shorter-distance but longer-duration trips**, which is consistent with increased congestion during busy commuting periods.

Precipitation also shows a positive association with peak-hour taxi demand.

---

## Geospatial Analysis

Taxi-zone geographic data are used to visualise the **spatial distribution of peak-hour demand across New York City**.

The geospatial workflow uses:

* GeoPandas;
* NYC Taxi Zone spatial data;
* Folium; and
* interactive geographic visualisation.

This provides a spatial perspective that complements the temporal demand analysis and helps identify areas with greater peak-hour activity.

---

## Revenue Prediction

Two regression approaches are evaluated for predicting **total trip revenue**.

Predictor variables include:

* trip distance;
* wind speed;
* precipitation;
* trip duration; and
* passenger count.

---

### Linear Regression

The Linear Regression model uses an **80/20 train-test split**.

| Metric |    Result |
| ------ | --------: |
| MAE    |  **5.87** |
| MSE    | **97.62** |
| RMSE   |  **9.88** |
| R²     |  **0.82** |

The model explains approximately **82% of the observed variance in trip revenue**.

---

### Random Forest Regression

A Random Forest model is also evaluated to capture potential non-linear relationships.

| Metric |     Result |
| ------ | ---------: |
| MAE    |   **5.80** |
| MSE    | **101.38** |
| RMSE   |  **10.07** |
| R²     |   **0.81** |

Both models demonstrate substantial predictive capability, with the Linear Regression model producing a slightly higher R² and lower RMSE in the reported experiments.

---

## Key Findings

### 1. Taxi demand follows clear temporal patterns

Morning and evening commuting periods consistently show elevated demand across both years.

### 2. Weekdays generally have stronger demand

Wednesday and Thursday are particularly important operating days in the analysed Autumn periods.

### 3. Peak-hour trips tend to be shorter but slower

Peak-hour trip count is strongly negatively associated with average trip distance and positively associated with trip duration.

### 4. Weather contributes to demand variation

Precipitation shows a positive relationship with taxi demand, while wind speed and visibility show negative associations in the analysed data.

### 5. Trip revenue is reasonably predictable

Trip distance, trip duration, weather conditions, and passenger count provide substantial predictive information for trip-level revenue.

---

## Operational Recommendations

Based on the analysis:

* **Increase taxi availability during 7–9 AM and 5–7 PM.**
* Prioritise vehicle deployment during higher-demand weekdays, particularly **Wednesday and Thursday**.
* Reduce unnecessary vehicle supply during very low-demand early-morning periods.
* Consider weather conditions when planning short-term vehicle deployment.
* Use historical demand and revenue patterns to support driver scheduling and fleet planning.

---

## Repository Structure

```text
nyc-yellow-taxi-analysis/
│
├── README.md
├── report_pdf_version.pdf
│
├── Playground.ipynb
├── pre-process.ipynb
├── EDA.ipynb
├── geography graph.ipynb
└── Model.ipynb
```

### `Playground.ipynb`

Handles **data acquisition**, including:

* NYC Yellow Taxi trip data;
* Visual Crossing weather data; and
* NYC Taxi Zone geographic data.

### `pre-process.ipynb`

Contains the main **PySpark preprocessing and feature-engineering pipeline**, including data cleaning, external-data integration, feature construction, and outlier handling.

### `EDA.ipynb`

Contains exploratory analysis of:

* weekly demand;
* hourly demand;
* peak-hour identification; and
* correlations between demand, trip characteristics, and weather.

### `geography graph.ipynb`

Contains the **geospatial analysis and visualisation** of peak-hour taxi demand using NYC taxi-zone geographic data.

### `Model.ipynb`

Contains the revenue prediction experiments using:

* Linear Regression; and
* Random Forest Regression.

---

## Technologies

* Python
* PySpark
* Spark SQL
* Spark MLlib
* Pandas
* NumPy
* Matplotlib
* Seaborn
* GeoPandas
* Folium
* Scikit-learn ecosystem
* Jupyter Notebook

---

## Installation

The project primarily requires Python, PySpark, and geospatial/data-analysis libraries.

Example installation:

```bash
pip install pyspark pandas numpy matplotlib seaborn geopandas folium selenium pillow
```

A dedicated `requirements.txt` can also be used to reproduce the environment.

---

## Data Availability

The original NYC taxi dataset is not included in this repository because of its size.

Data can be downloaded from:

**NYC Taxi & Limousine Commission**
https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page

Weather data can be obtained from:

**Visual Crossing Weather**
https://www.visualcrossing.com/weather-data

NYC taxi-zone geographic data are available through the TLC Taxi Zone Maps and Lookup Tables.

---

## Limitations

Several limitations should be considered when interpreting the results:

* Manhattan weather observations are used as an approximation for broader NYC weather conditions.
* Weather conditions are assumed to remain consistent within each hourly interval.
* Some potentially useful explanatory variables are not included.
* Linear Regression may not capture all non-linear relationships in revenue generation.
* Historical 2022–2023 demand patterns may not fully represent future taxi behaviour.

---

## Full Report

For the complete preprocessing methodology, exploratory analysis, geospatial visualisations, modelling results, recommendations, limitations, and references:

📄 **[View the full project report](./report_pdf_version.pdf)**
