
# 📁 Data Directory

This folder contains both **raw** and **processed** datasets used in the Crash & Fatalities in Australia Data Warehouse Project.

---

## 📄 Contents

### 🔹 `bitre_fatal_crashes_dec2024.csv`
- **Source:** [BITRE Fatal Crash Database](https://www.bitre.gov.au/statistics/safety/fatal_road_crash_database)
- **Description:** Contains crash-level information including crash type, location, time, and number of fatalities.
- **Status:** Raw data

### 🔹 `bitre_fatalities_dec2024.csv`
- **Source:** [BITRE Fatal Crash Database](https://www.bitre.gov.au/statistics/safety/fatal_road_crash_database)
- **Description:** Contains individual-level data about people involved in crashes (drivers, passengers, pedestrians).
- **Status:** Raw data

### 🔹 Processed CSV Files
- **Source:** Generated through ETL (Jupyter Notebook)
- **Description:** Cleaned and transformed dimension and fact tables used for PostgreSQL loading
- **Status:** Processed data

---

## 📜 License & Usage

All datasets originate from the [BITRE Fatal Road Crash Database](https://www.bitre.gov.au/statistics/safety/fatal_road_crash_database), provided by the Australian Government under the following terms:

- Licensed under the [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/)
- Use must comply with the **Copyright Act 1968**

---

## 📌 Notes

- Missing values were marked as `-9`, `Unknown`, or `Undetermined` in the raw data and replaced with `NaN` during processing.
- Boolean fields (e.g., Bus Involvement) were standardized to `True`/`False`.
- All transformed files were exported as `.csv` and loaded into PostgreSQL during the ETL process.

