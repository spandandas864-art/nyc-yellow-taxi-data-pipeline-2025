# NYC Yellow Taxi Data Processing, Feature Engineering & Analytics Pipeline (2025)

[![Tableau](https://img.shields.io/badge/Tableau-Dashboard-E97627?logo=tableau&logoColor=white)](https://public.tableau.com/views/NYCTaxi2025Jan-June/CBDanalysis?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)
[![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Pandas](https://img.shields.io/badge/Pandas-Data%20Wrangling-150458?logo=pandas&logoColor=white)](https://pandas.pydata.org/)
[![SciPy](https://img.shields.io/badge/SciPy-Hypothesis%20Testing-8CAAE6?logo=scipy&logoColor=white)](https://scipy.org/)

An end-to-end data engineering, cleaning, feature engineering, and statistical analysis pipeline for the 2025 New York City Yellow Taxi dataset provided by the NYC Taxi and Limousine Commission (TLC). This project handles multi-month parquet aggregation, data sanitization, feature derivation, business assumption hypothesis testing, and interactive dashboarding.

---

## Tableau Dashboard Overview
Access the interactive dashboard showing key ride trends, fare distributions, peak travel hours, and tip analytics:
**[View Interactive Tableau Dashboard](https://public.tableau.com/views/NYCTaxi2025Jan-June/CBDanalysis?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)**

---

## Dataset Overview
The dataset comprises 6 months of 2025 NYC Yellow Taxi trip records published by the NYC TLC.

* **Initial Record Count:** 24,083,384 rows across 20 columns
* **Initial File Memory Footprint:** ~3.3+ GB
* **Optimized Dataset Memory:** ~1.4+ GB (via datatype casting & memory tuning)

| Metric / Dimension | Column Field | Operational Context |
| :--- | :--- | :--- |
| **Primary Identifier** | `VendorID`, `tpep_pickup_datetime` | TPEP provider records & timestamp tracking |
| **Geographic Coverage** | 265 TLC Taxi Zones | Pickup (`PULocationID`) & Dropoff (`DOLocationID`) |
| **Core Financials** | `fare_amount`, `tip_amount`, `tolls_amount` | Itemized fares, surcharges, and payment types |
| **Fulfillment Metrics** | `trip_distance`, `passenger_count` | Metered mileage and passenger capacity tracking |
| **Surcharges & Fees** | `mta_tax`, `extra`, `cbd_congestion_fee` | Rush hour, night, and CBD congestion fees |

---

## Core Business Questions & Hypotheses

### Pillar A: Spatial Demand & Congestion Dynamics
* **Q1. Peak Hour Bottlenecks**: Which pickup taxi zones (`PULocationID`) experience the highest trip density during weekday morning (`7 AM – 10 AM`) vs. evening rush hours (`4 PM – 7 PM`)?
* **Q2. Airport Corridor Efficiency**: What is the average trip duration and speed (`mph`) between Midtown Manhattan and JFK/LaGuardia airports across rate codes (`RatecodeID`)?
* **Q3. Congestion Fee Friction**: How has the introduction of CBD congestion surcharges impacted total trip volume in lower Manhattan?

### Pillar B: Revenue & Tipping Drivers
* **Q4. Payment Method & Tip Rates**: What is the average tip percentage for credit card transactions vs. cash payments across short (`<2 miles`) vs. long-haul trips?

---

## 3. Analytical Methodologies & Tools
* **Data Processing**: `pandas` for multi-gigabyte Parquet transactional processing.
* **Geospatial Analytics**: Mapped TLC Taxi Zone shapefiles for spatial joins & zone-to-zone routing density.
* **Exploratory Analytics**: `matplotlib`, `seaborn` for visual temporal fare & distance distributions.

## Pipeline Architecture & Workflow

### Phase 1: Data Merging & Storage Optimization (`01_data_merging`)
1. **Automated File Merging:** Used Python's `glob` module to dynamically iterate through 6 individual monthly parquet files (`yellow_tripdata_2025-*.parquet`) and concatenated them into a single consolidated DataFrame.
2. **Export:** Saved the unified 24.08M row dataset to a single compressed Parquet file (`yellow_taxi_6_months.parquet`).

### Phase 2: Data Quality & Cleaning (`02_data_cleaning`)
* **Timestamp Anomaly Correction:** Identified 793 records where pickup timestamps occurred after dropoff timestamps. Automatically resolved negative trip durations by swapping pickup and dropoff values.
* **Missing Value Imputation:**
  * `passenger_count`: Imputed 5,418,601 missing entries and 0-passenger trips with the mode (`1.0`). Capped extreme passenger values (>6) to the median.
  * `RatecodeID`: Replaced invalid/unknown codes (`99.0`) and missing values with the most frequent rate code (`1.0` - Standard rate).
* **Logical Filtering & Error Removal:**
  * Removed 657,758 records with a trip distance of `0` miles but a non-zero `fare_amount`.
  * Filtered out 1,323,048 rows containing negative fare amounts.
  * Removed 9,342 records with zero fare amounts.
* **Outlier Detection:**
  * Applied Interquartile Range (IQR) filtering on `trip_distance` to remove physically impossible single-trip distance outliers.

### Phase 3: Feature Engineering (`03_feature_engineering`)
* **Temporal Feature Extraction:** Extracted `pickup_hour`, `day_of_week`, `month`, `is_weekend`, and categorized day periods (`Morning Rush`, `Midday`, `Evening Rush`, `Night`).
* **Derived Operational Metrics:**
  * `trip_duration_minutes`: Calculated exact duration derived from pickup and dropoff timestamps.
  * `average_speed_mph`: Calculated speed using trip distance and duration to flag improbable speed anomalies.
  * `tip_percentage`: Computed relative tip percentage (`tip_amount` / `fare_amount` * 100) for tipping behavior analysis.
* **Categorical Binning:** Grouped trip distances into buckets (`Short <2mi`, `Medium 2-5mi`, `Long 5-10mi`, `Extreme >10mi`) for segmented passenger behavior analysis.

### Phase 4: Exploratory Data Analysis & Hypothesis Testing (`04_EDA_and_hypothesis_testing`)
* **Exploratory Data Analysis (EDA):**
  * Profiled trip demand density across hours of the day and days of the week.
  * Analyzed fare revenue distributions across pickup locations and passenger count groups.
* **Statistical Hypothesis Testing:**
  * **Hypothesis 1 (Tip Rate vs. Payment Method):** Conducted a Two-Sample T-Test / Mann-Whitney U Test to verify if credit card users tip significantly higher rates than cash payers ($p < 0.05$).
  * **Hypothesis 2 (Rush Hour Speed Impact):** Evaluated whether average trip speeds during peak commute hours differ significantly from off-peak hours using Welch's t-test.
  * **Hypothesis 3 (Distance vs. Tip Willingness):** Tested correlation between trip distance bins and tip percentages to validate long-distance tipping behavior assumptions.

---

## Repository Structure

├── 01_data_merging_and_preprocessing.ipynb     # Batch parquet merging & memory optimization
├── 02_data_cleaning_and_outlier_treatment.ipynb  # Data cleaning, missing value imputation, IQR outlier removal
├── 03_feature_engineering.ipynb                 # Temporal feature extraction, duration, speed, and tipping ratios
├── 04_EDA_and_hypothesis_testing.ipynb          # Exploratory analysis and statistical hypothesis testing
├── .gitignore                                  # Ignores heavy parquet files (>100MB)
└── README.md                                   # Project documentation and pipeline breakdown

## Tech Stack & Libraries

* **Language:** Python 3.12
* **Data Processing & Feature Engineering:** `pandas`, `numpy`, `glob`, `datetime`, `pyarrow`
* **Statistical Testing & Analytics:** `scipy`, `statsmodels`
* **Visualization:** Tableau Public, `matplotlib`, `seaborn`
* **Development Environment:** Jupyter Notebook / Google Colab

## 🚀 How to Run Locally

### Prerequisites
* Python 3.10+
* Git
* Jupyter Notebook or JupyterLab

---

### Step 1: Clone the Repository
```bash
git clone [https://github.com/YOUR_USERNAME/nyc-yellow-taxi-data-pipeline-2025.git](https://github.com/YOUR_USERNAME/nyc-yellow-taxi-data-pipeline-2025.git)
cd nyc-yellow-taxi-data-pipeline-2025
```

### Step 2: Set Up a Virtual Environment & Install Dependencies
```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
# On macOS/Linux:
source venv/bin/activate
# On Windows:
venv\Scripts\activate

# Install required packages
pip install -r requirements.txt
```

### Step 3: Download the Datasets
```
1.Go to the official NYC TLC Trip Record Data page(https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page).
2.Download the 2025 Yellow Taxi trip record Parquet files (e.g., yellow_tripdata_2025-01.parquet through yellow_tripdata_2025-06.parquet).
3.Place all raw .parquet files in your project root directory (or inside a designated data/ folder).
```

### Step 4: Execute the Pipeline
```
Run the notebooks in this exact sequence:
1.01_data_merging_and_preprocessing.ipynb — Concatenates monthly parquet files and optimizes memory.
2.02_data_cleaning_and_outlier_treatment.ipynb — Handles missing values, fixes timestamp errors, and filters outliers.
3.03_feature_engg.ipynb — Derives trip duration, speed, temporal features, and tip ratios.
4.04_EDA_and_hypothesis_testing.ipynb — Performs exploratory analysis and statistical tests.
```
