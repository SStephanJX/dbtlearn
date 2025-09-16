# DBTLEARN: Airbnb Data Transformation Pipeline

This dbt (Data Build Tool) project transforms raw, operational Airbnb data into a clean, tested, and documented analytics-ready dataset. It was developed as the culmination of the **dbt Fundamentals** course, demonstrating core competencies in modern data transformation practices.

## 🎯 Overview

The primary goal of this project is to model raw data from an Airbnb-style source into a structured star schema, enabling powerful analysis on hosts, listings, and guest reviews. The pipeline implements slowly changing dimensions (SCDs) for host data and efficiently handles high-volume review data with incremental modeling.

## 👤 Author

**Stephen Stephan** — Data Engineer & Analytics Practitioner  
Jacksonville, FL | Open to remote, hybrid, or local roles  
Focused on building reliable, well-documented data pipelines that serve clear business needs.

## 🌟 Why This Project Matters

DBTLEARN is more than a technical exercise—it’s a demonstration of how modern data transformation tools like dbt can bring software engineering best practices to analytics. Every model is built for clarity, every transformation is documented, and every decision is made with the future maintainer in mind.

## 📊 Data Modeling

This project follows a medallion architecture, structuring the transformations into distinct layers for clarity and reliability.

**Model Layers:**
1.  **`src` (Ephemeral Models):** Lightly transforms raw source data from the `AIRBNB.RAW` schema. This includes column renaming, basic type casting, and filtering.
    *   `src_hosts`: Creates a cleansed view of host data.
    *   `src_listings`: Creates a cleansed view of listing data, preparing the `price_str` for conversion to a numeric type.
    *   `src_reviews`: Creates a cleansed view of review data.

2.  **`dim` (Dimension Models):** Builds reusable dimension tables using views and tables.
    *   `dim_hosts_cleansed` (View): A cleansed dimension of hosts, implementing business logic to handle null `host_names`.
    *   `dim_listings_cleansed` (View): A cleansed dimension of listings, enforcing business rules for `minimum_nights` and parsing the `price` field.
    *   `dim_listings_w_hosts` (Table): An enriched dimension table that joins listings with host information (like `is_superhost`), creating a one-stop-shop for listing attributes.

3.  **`fct` (Fact Model):** Builds an incremental fact table for event-based data.
    *   `fct_reviews` (**Incremental Table**): A fact table for reviews. Implements complex incremental logic to efficiently process only new data, using a surrogate key generated from natural keys for robust unique identification.

4.  **`mart` (Data Mart):** A business-specific dataset answering a particular analytical question.
    *   `mart_fullmoon_reviews` (Table): An example analytical mart that enriches review data with information from a seed file (`seed_full_moon_dates`) to analyze if review sentiment correlates with full moons.

**Data Lineage:**
The transformation flow follows this path:
raw_hosts → src_hosts → dim_hosts_cleansed -─┐
├─> dim_listings_w_hosts
raw_listings → src_listings → dim_listings_cleansed ─┘
│
raw_reviews → src_reviews → fct_reviews → mart_fullmoon_reviews
│
seed_full_moon_dates ────────┘


## 🧠 Design Decisions

Key architectural choices made for performance, clarity, and maintainability:
-   **`src` models are ephemeral** to reduce warehouse clutter and avoid building unnecessary tables for simple transformations.
-   **`dim_listings_w_hosts` is materialized as a table** to support frequent downstream joins and benefit from caching for better performance.
-   **Incremental logic in `fct_reviews`** uses conditional logic with `start_date` and `end_date` variables to allow for targeted, scoped refreshes of data, optimizing run times and cost.
-   **Full moon enrichment** is offset by one day (`DATEADD(DAY, 1, fm.full_moon_date)`) to reflect the hypothesis that sentiment spikes may occur *after* the event itself.

## 🛠️ Technologies Used

- **Data Transformation:** dbt Core 1.9.0
- **Data Warehouse:** Snowflake
- **Source Data:** `AIRBNB.RAW` schema (RAW_HOSTS, RAW_LISTINGS, RAW_REVIEWS)
- **Development:** SQL, Jinja, YAML

## 📦 Package Highlights

This project leverages the power of dbt's package ecosystem:
-   **`dbt_utils`**: Used for reliable surrogate key generation (`generate_surrogate_key`) to ensure unique identifiers for fact records.
-   **`dbt_expectations`** (Suggested): This package provides declarative data tests, enhancing maintainability and clarity for data quality checks.
-   **`dbt_date`** (Suggested): Useful for date spine generation and advanced date logic, complementing time-based analysis.

## 📁 Project Structure
├── models/
│ ├── staging/
│ │ ├── src_hosts.sql
│ │ ├── src_listings.sql
│ │ ├── src_reviews.sql
│ │ └── sources.yml
│ ├── dim/
│ │ ├── dim_hosts_cleansed.sql
│ │ ├── dim_listings_cleansed.sql
│ │ └── dim_listings_w_hosts.sql
│ ├── fct/
│ │ └── fct_reviews.sql
│ └── mart/
│ └── mart_fullmoon_reviews.sql
├── seeds/
│ └── full_moon_dates.csv
├── snapshots/
│ └── scd_raw_hosts.sql
├── macros/
├── tests/
├── analysis/
├── dbt_project.yml
└── packages.yml


## 🚀 Getting Started

### Prerequisites
- Access to a Snowflake data warehouse with the `AIRBNB.RAW` schema populated.
- dbt Core installed and configured on your machine.
- A configured `~/.dbt/profiles.yml` file pointing to your Snowflake dev environment.

### Installation & Setup
1.  **Clone this repository:**
    ```bash
    git clone <your-repo-url>
    cd dbtlearn
    ```
2.  **Install dbt dependencies** (includes `dbt_utils`):
    ```bash
    dbt deps
    ```
3.  **Validate your connection:**
    ```bash
    dbt debug
    ```

## 💻 Usage

- **Run all models:**
  ```bash
  dbt run
  dbt run --select dim/*
  dbt test
  dbt docs generate
  dbt docs serve
  Navigate to http://localhost:8080 to explore complete documentation and data lineage.

📜 Certification
This project was completed as part of the requirements for the official dbt Fundamentals Certification.

Note: This project uses a simulated Airbnb dataset for educational purposes.
