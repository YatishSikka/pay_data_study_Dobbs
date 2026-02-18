# Healthcare Job Market Impact Analysis (Post-Dobbs)
This repository contains a data analysis pipeline and econometric study investigating the impact of the **Dobbs v. Jackson (2022)** decision on the US healthcare labor market. The project specifically tracks job posting trends for specialized reproductive healthcare roles across states with varying legal landscapes.

## Project Overview
The core of this project is a longitudinal study using job posting data from Google BigQuery. It categorizes US states based on their legal response to the 2022 Supreme Court ruling and employs an event study model to quantify shifts in demand for medical professionals.

### Key Roles Analyzed
The pipeline filters millions of records to identify specific healthcare categories:
- **Obstetrics & Gynecology (OBGYN)**
- **Certified Nurse Midwives (CNM)**
- **Labor & Delivery Nurses (L&D)**
- **Maternal-Fetal Medicine (MFM)**

## Data Pipeline
1. **Extraction:** Connects to the `linkup` dataset on Google BigQuery to fetch massive volumes of longitudinal job posting data.
2. **Processing:** Iterates through 428 partitioned tables, utilizing regular expressions to filter and classify job titles and descriptions while handling missing records.
3. **Categorization:** Classifies states into four legal tiers:
  - **Illegal:** Immediate or trigger bans (e.g., TX, AL, MS, AR).
  - **Contested:** Active legal challenges or blocked bans (e.g., AZ, GA, WI, NC).
  - **Restricted but Legal:** Significant gestational limits (e.g., FL, NE, UT).
  - **Protected Legal:** Access is codified or protected (e.g., CA, NY, IL, WA).

### Data Retrieval & Filtering Strategies
To handle millions of records across hundreds of table partitions, the following strategies were employed:
- **Partitioned Batch Processing:** The dataset was distributed across 428 individual table partitions. Instead of a single massive join, the pipeline iterates through these partitions using a programmatic loop. This strategy prevents query timeouts and memory exhaustion while allowing for granular error tracking.
- **Source-Side Filtering (SQL Optimization):** Filtering is performed directly within the BigQuery SQL environment rather than in local memory. By using `CASE` statements and `WHERE` clauses to discard "Other" categories before the data reaches the Python environment, the transfer volume is significantly reduced.
- **Precision Filtering via Regular Expressions:** To distinguish between specialized medical roles and administrative or support staff, the queries use complex `REGEXP_CONTAINS` logic. This ensures "valid records" by:
  - **Inclusion:** Matching specific professional keywords (e.g., `obgyn`, `midwife`, `mfm`).
  - **Exclusion:** Explicitly filtering out noise terms like `assistant`, `technician`, `scheduler`, `admin`, and `secretary`.
- **Fault-Tolerant Execution:** The retrieval script implements a `try-except` architecture to handle missing partitions (e.g., 404 Not Found errors) or intermittent connection issues. This allows the process to log failures for specific parts (like parts 212 and 230) and continue processing the remaining data without crashing the entire pipeline.
- **Data Integrity Mapping:** The pipeline utilizes `LEFT JOIN` operations between metadata and description tables, followed by a post-processing validation step to identify and export records with missing descriptions for further troubleshooting.

## Econometric Methodology
The analysis utilizes a Log-Linear Event Study Model to measure market shifts:
- **Dependent Variable:** Log-transformed job posting counts.
- **Fixed Effects:** The model controls for both `state` and `month` to isolate the specific impact of the decision from general market trends.
- **Time Window:** Analyzes the 12 months preceding and following the June 2022 decision to visualize lead and lag effects.

## Tech Stack
- **Data Warehouse:** Google BigQuery
- **Language:** Python
- **Key Libraries:**

  - `pandas` & `numpy`: Data manipulation and processing.
  - `statsmodels`: OLS Regression and Event Study modeling.
  - `matplotlib`: Data visualization and coefficient plots.
  - `google-cloud-bigquery`: Cloud data integration.

## Getting Started
1. **Authentication:** Ensure you have a Google Cloud account with access to the BigQuery project. Authenticate via:
```
gcloud auth application-default login
```
2. **Configuration:** Update the `PROJECT_ID` variable in the notebook to your active GCP project.
3. **Data Generation:** Run the fetching cells to process the partitions and generate local datasets (`full_data.csv` and `nurse_postings.csv`).
4. **Analysis:** Execute the econometric section to produce the coefficient plots for the event study.
