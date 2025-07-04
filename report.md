# 🚗 Crashes and Fatalities in Australia — Data Warehouse Project

## 📌 Introduction

One of the most common modes of transportation for the average person is driving a car. With hundreds of new drivers obtaining their licenses each day, Australia’s roads are becoming increasingly populated. As of late 2023, there was an estimated 19.4 million licensed drivers in Australia alone – a number that has grown since and will continue to grow.  
Unfortunately, this rise in road usage is accompanied by a troubling trend: an increase in road crashes and fatalities. A 2023 report revealed that Australia experienced its highest number of road fatalities in five years.  

In response, the country has adopted the Vision Zero initiative – a bold commitment to achieve a total of zero deaths and serious injuries in car crashes. This strategy emphasizes that no loss of life is acceptable, regardless of the circumstances.

This project supports that goal by analyzing several datasets to develop a comprehensive recommendation strategy for both the government and the public. The process includes:

- Building a data warehouse (fact table + dimensions)
- Cleaning and transforming datasets using Python
- Performing ETL into PostgreSQL
- Answering business queries using a Starnet model
- Visualizing findings using Tableau

---

## 🛠️ Tools and Methodology

**Datasets:**
- `bitre_fatal_crashes_dec2024.csv`
- `bitre_fatalities_dec2024.csv`

**Tools & Technologies:**
- **Python / Jupyter Notebook** — for data cleaning and ETL
- **pandas**, **numpy**, **mlxtend** — for transformations and preparation
- **PostgreSQL** — for data warehousing
- **Tableau** — for visualizations and dashboards

**Modelling Approach:**
Follows Kimball’s 4-step dimensional modelling methodology:

1. Select the business process
2. Declare the grain of the fact table
3. Identify the dimensions
4. Define the numeric measures

---

## 🧾 Fact Table and Dimensions

The **fact table** contains crash-level records, with:
- `crashkey` as primary key
- `crash_count = 1` for aggregation
- `fatalities_count` as a numeric measure

**Dimension tables** (10 total):
- `dim_date`: date, day of week, month, year
- `dim_time`: time, time of day
- `dim_holiday`: holiday type (Christmas, Easter)
- `dim_outcome`: crash type, fatalities
- `dim_vehicle`: booleans like bus, truck involvement
- `dim_location`: state, LGA, SA4, remoteness
- `dim_road`: road type, speed limit
- `dim_gender`: gender of road user
- `dim_person`: type of road user (driver, passenger)
- `dim_age`: age and age group

---

## 🕸️ Starnet Model & Business Queries

The Starnet model allows for slicing and dicing facts by different combinations of dimensions, useful for business queries.

### 🧠 Query 1: Most Multiple Crashes by State (Last 10 Years)
```text
Relevant Dimensions:
- Location: State
- Crash Type: Filter = Multiple
- Date: Last 10 years
- Others: All
```

### 🚌 Query 2: Fatalities on Monday Bus Crashes
```text
Relevant Dimensions:
- Date: Day of Week = Monday
- Vehicle: Bus Involvement = True
- Fact: Fatalities count
- Others: All
```

### 🌃 Query 3: Nighttime Crash Involvement by Age Group (2021, NSW/VIC/QLD)
```text
Relevant Dimensions:
- Age Group
- Location: State in ["NSW", "QLD", "VIC"]
- Time: Time of Day = Night
- Date: Year = 2021
- Others: All
```

### 🎄 Query 4: Crashes by Speed Limit During Christmas (Past 3 Years)
```text
Relevant Dimensions:
- Road: Speed Limit
- Holiday: Christmas
- Date: Last 3 Years
- Others: All
```

### ⚥ Query 5: Gender and Month Combo with Most Fatalities in Major Cities
```text
Relevant Dimensions:
- Gender
- Date: Month + Year (last 10 years)
- Remoteness: Major Cities
- Fact: Fatalities count
- Others: All
```

---

## 🧼 Data Cleaning

To prepare the datasets for the ETL (Extract, Transform, Load) process, several cleaning steps were applied to ensure the data is reliable and compatible with the schema. The two primary datasets, `fatal_crashes` and `fatalities`, were read into Jupyter Notebook using `pandas` and merged on the shared `Crash ID` for unified cleaning.

In the data cleaning process, Generative AI was consulted to identify common data quality issues and best practices. Techniques used included handling missing values, standardizing formats, deduplication, range validation, and creating derived fields.

### 🔎 Replacing Missing and Invalid Values
Values such as blanks, 'unknown', and 'undetermined' were standardized to `NaN` to later be interpreted as `NULL` in SQL. The `-9` value was used in the dataset to indicate missing data and was treated accordingly.
```python
crashes_fatalities = crashes_fatalities.replace([-9, '-9'], pd.NA)
crashes_fatalities = crashes_fatalities.replace('Unknown', pd.NA)
crashes_fatalities = crashes_fatalities.replace('Undetermined', pd.NA)
crashes_fatalities = crashes_fatalities.replace('', pd.NA)
```

### 🔄 Converting Yes/No Values to Boolean
Categorical fields with "Yes"/"No" values were converted to Python boolean types for compatibility with SQL.
```python
crashes_fatalities['Bus Involvement'] = crashes_fatalities['Bus Involvement'].replace({'Yes': True, 'No': False})
crashes_fatalities['Heavy Rigid Truck Involvement'] = crashes_fatalities['Heavy Rigid Truck Involvement'].replace({'Yes': True, 'No': False})
crashes_fatalities['Articulated Truck Involvement'] = crashes_fatalities['Articulated Truck Involvement'].replace({'Yes': True, 'No': False})
crashes_fatalities['Christmas Period'] = crashes_fatalities['Christmas Period'].replace({'Yes': True, 'No': False})
crashes_fatalities['Easter Period'] = crashes_fatalities['Easter Period'].replace({'Yes': True, 'No': False})
```

### 📁 Renaming Columns to Match Schema
Renaming columns helped align the data with the target schema.
```python
crashes_fatalities = crashes_fatalities.rename(columns={
    'Crash ID': 'crashkey', 
    'State': 'state_name', 
    'Month': 'month_name', 
    'Year': 'year',
    'Dayweek': 'day_of_week',
    'Time': 'time_value',
    'Crash Type': 'crash_type',
    'Number Fatalities': 'number_fatalities',
    ...
})
```

### ❌ Removing Duplicate Records
Duplicates were removed to prevent inflated metrics.
```python
crashes_fatalities = crashes_fatalities.drop_duplicates()
```

### 🔢 Validating Speed Limits
Speed limits were validated to fall between 5 and 130 km/h. None were found to be invalid.
```python
invalid_speeds = crashes_fatalities[(crashes_fatalities['speed_limit'] < 5) | (crashes_fatalities['speed_limit'] > 130)]
```

### ➕ Creating Derived Columns
A derived column `holiday_type` was created to consolidate holiday flags.
```python
crashes_fatalities['holiday_type'] = 'None'
crashes_fatalities.loc[crashes_fatalities['Christmas Period'] == True, 'holiday_type'] = 'Christmas'
crashes_fatalities.loc[(crashes_fatalities['Easter Period'] == True) & (crashes_fatalities['Christmas Period'] != True), 'holiday_type'] = 'Easter'
```

### 🔍 Checking for Inconsistent Categorical Values
Categorical fields were validated for typos or inconsistent values.
```python
print('States included:', crashes_fatalities['state_name'].unique())
print('Months included:', crashes_fatalities['month_name'].unique())
```

### ⚠️ Dropping Incomplete Records
Highly incomplete records (with 6 or more missing values) were excluded to preserve data quality.
```python
crashes_fatalities = crashes_fatalities[crashes_fatalities.isna().sum(axis=1) < 6]
```

### 📆 Creating the Date Dimension Table
Once cleaned, the data was split into multiple DataFrames for each dimension and the fact table. Below is an example for the date dimension:
```python
date_df = crashes_fatalities[["year","month_name","day_of_week"]].drop_duplicates()
d_index = range(101,101+len(date_df))
date_df["date_id"] = ["D" + str(i) for i in d_index]
date_df = date_df[["date_id","day_of_week","month_name","year"]]
```

---
