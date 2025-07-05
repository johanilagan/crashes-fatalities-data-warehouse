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
- Python / Jupyter Notebook — for data cleaning and ETL
- `pandas`, `numpy`, `mlxtend` — for transformations and preparation
- PostgreSQL — for data warehousing
- Tableau — for visualizations and dashboards

**Modelling Approach:**
Follows Kimball’s 4-step dimensional modelling methodology:

1. Select the business process
2. Declare the grain of the fact table
3. Identify the dimensions
4. Define the numeric measures

---

## 🧾 Fact Table and Dimensions

The fact table contains crash-level records, with:
- `crashkey` as primary key
- `crash_count = 1` for aggregation
- `fatalities_count` as a numeric measure

Dimension tables (10 total):
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
*Which state in Australia recorded the most multiple crashes in the last 10 years?*
```text
Relevant Dimensions:
- Location: State
- Crash Type: Filter = Multiple
- Date: Last 10 years
- Others: All
```

### 🚌 Query 2: Fatalities on Monday Bus Crashes
*How many fatalities occurred on a Monday crash that involved a bus?*
```text
Relevant Dimensions:
- Date: Day of Week = Monday
- Vehicle: Bus Involvement = True
- Fact: Fatalities count
- Others: All
```

### 🌃 Query 3: Nighttime Crash Involvement by Age Group (2021, NSW/VIC/QLD)
*What age group is most involved in crashes in states NSW, Queensland, and Victoria at night time in 2021?*
```text
Relevant Dimensions:
- Age Group
- Location: State in ["NSW", "QLD", "VIC"]
- Time: Time of Day = Night
- Date: Year = 2021
- Others: All
```

### 🎄 Query 4: Crashes by Speed Limit During Christmas (Past 3 Years)
*How does the speed limit affect the number of crashes in Australia the past 3 Christmas seasons?*
```text
Relevant Dimensions:
- Road: Speed Limit
- Holiday: Christmas
- Date: Last 3 Years
- Others: All
```

### ⚥ Query 5: Gender and Month Combo with Most Fatalities in Major Cities
*Which gender and month combination had the most fatalities in major cities of Australia?*
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

## 🛠️ ETL Process

With the data cleaned and transformed into CSV format, the next step in the ETL (Extract, Transform, Load) pipeline is the **Load phase**, where the data is inserted into the previously designed data warehouse.

This phase was carried out using PostgreSQL, where both the dimension tables and the fact table were created and populated using SQL.

### 🧱 SQL Code: Table Creation
```sql
CREATE TABLE dim_age (
    age_id INT PRIMARY KEY,
    age_value INT NULL,
    age_grp VARCHAR(20) NULL
);

CREATE TABLE FactCrash (
    CrashKey INT PRIMARY KEY,
    DateKey INT NOT NULL,
    TimeKey INT NOT NULL,
    HolidayKey INT NOT NULL,
    LocationKey INT NOT NULL,
    RoadKey INT NOT NULL,
    OutcomeKey INT NOT NULL,
    VehicleKey INT NOT NULL,
    GenderKey INT NULL,
    PersonKey INT NOT NULL,
    AgeKey INT NULL,
    crash_count INTEGER,
    fatality_count INTEGER,
    FOREIGN KEY (DateKey) REFERENCES dim_date(date_id),
    FOREIGN KEY (TimeKey) REFERENCES dim_time(time_id),
    FOREIGN KEY (HolidayKey) REFERENCES dim_holiday(holiday_id),
    FOREIGN KEY (LocationKey) REFERENCES dim_location(location_id),
    FOREIGN KEY (RoadKey) REFERENCES dim_road(road_id),
    FOREIGN KEY (OutcomeKey) REFERENCES dim_outcome(outcome_id),
    FOREIGN KEY (VehicleKey) REFERENCES dim_vehicle(vehicle_id),
    FOREIGN KEY (GenderKey) REFERENCES dim_gender(gender_id),
    FOREIGN KEY (PersonKey) REFERENCES dim_person(person_id),
    FOREIGN KEY (AgeKey) REFERENCES dim_age(age_id)
);
```

### 📥 Data Loading
Once the schema was defined, the cleaned CSV files were loaded into PostgreSQL using the `COPY` command:

```sql
COPY dim_vehicle
FROM '/tmp/Dim_vehicle.csv'
WITH (FORMAT csv, HEADER true);

COPY factcrash
FROM '/tmp/Fact_crashes.csv'
WITH (FORMAT csv, HEADER true);
```

Similar commands were executed for the other dimension tables.

---

After the ETL process, the fact and dimension tables were fully populated. A schema Entity Relationship Diagram (ERD) was created in PostgreSQL based on the foreign key relationships.

This ERD represents a star schema, with:
- A central fact table (`FactCrash`)
- Connected to 10 surrounding dimension tables

Having a low number of tables results in having fewer joins, which makes its performance faster. Star schemas are also easier to use since the numeric measures involve counting and aggregating the crash and fatalities data. Since it is more straightforward than a snowflake schema, it is also beneficial to use this when answering business queries.

## 📊 Data Visualization

This section presents the visualizations used to answer the business queries formulated earlier.

### Query 1: Most Multiple Crashes by State (Last 10 Years)

*Which state in Australia recorded the most multiple crashes in the last 10 years?*

A pie chart was used to compare multiple crash counts by state.  
**Insight:** NSW recorded the most multiple crashes, followed by QLD and VIC. There's a large drop-off after the top 3, suggesting a higher concentration of crashes in these states.

---

### Query 2: Fatalities on Monday Bus Crashes

*How many fatalities occurred on a Monday crash that involved a bus?*

A circle chart displays fatalities per day for bus-involved crashes.  
**Insight:** 23 fatalities occurred on Mondays. Saturdays recorded the highest number overall.

---

### Query 3: Age Group in Nighttime Crashes (NSW/QLD/VIC, 2021)

*What age group is most involved in crashes in states NSW, Queensland, and Victoria at night time in 2021?*

A circle chart shows the distribution of crash involvement by age group.  
**Insight:** The 40–64 age group had the highest involvement, possibly due to its wider age range.

---

### Query 4: Speed Limit vs Crashes (Christmas Seasons)

*How does the speed limit affect the number of crashes in Australia the past 3 Christmas seasons?*

A line chart was used to evaluate crash count across speed limits.  
**Insight:** Roads with a speed limit of 100 km/h saw the most crashes. However, this may reflect the number of roads with this speed limit rather than crash propensity.

---

### Query 5: Gender + Month with Most Fatalities (Major Cities)

*Which gender and month combination had the most fatalities in major cities of Australia?*

A grouped bar chart illustrates fatality counts by gender and month.  
**Insight:** Males in September had the highest fatality count. August and September consistently ranked high for both genders.

---
## 🤝 Association Rules Mining

Association Rules describe relationships between variables in the form of:

```
Antecedent → Consequent
```

Key metrics:
- **Support**: Frequency of occurrence
- **Confidence**: Accuracy of the rule
- **Lift**: Strength of the rule over randomness

This project used the **Apriori algorithm** to identify frequent itemsets, focusing on rules where the consequent = 'Road User'. Rules were sorted by lift and confidence.

### Top 3 Discovered Rules:
1. **{100, Weekday} → {Driver}**
2. **{Male, 100} → {Driver}**
3. **{Single, 100} → {Driver}**

---

## ✅ Recommendations for Crash Prevention

Based on the discovered patterns and visual trends:

### 1. 📍 Monitor Areas with 100 km/h Speed Limits
- These speed zones appeared in all top rules.
- Recommend stricter monitoring, review of speed limits, and enforcement.

### 2. 🔍 Investigate Male, Weekday, and Single-Vehicle Crashes
- Consider time-of-day trends and driving behavior of male drivers.
- Analyze urban vs rural differences and weekday peak times.

### 3. 🧠 Improve Driver Training & Evaluation
- All rules pointed to drivers as the critical factor.
- Suggest updates to curriculum, tougher testing, and road safety education.

---
## References

[1] Z. Kean. “What is it like not driving as an adult?” ABC News.https://www.abc.net.au/news/2023-01-18/adults-who-do-not-drive/101767786 (accessed April 14, 2025).

[2] J. Beazley. “2023 the deadliest year on Australia’s roads in more than half a decade, data shows.” The Guardian. https://www.theguardian.com/australia-news/2023/dec/18/2023-the-deadliest-year-on-australias -roads-in-more-than-half-a-decade-data-shows CMP=soc_568 (accessed April 14, 2025).

[3] National Road Safety Strategy. “Fact sheet: Vision Zero and the Safe System.” National Road Safety Strategy. https://www.roadsafety.gov.au/nrss/fact-sheets/vision-zero-safe-system (accessed April 14, 2025).

[4] Kimball Group. “Four-Step Dimensional Design Process.” Kimball Group. https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniqu es/dimensional-modeling-techniques/four-4-step-design-process/ (accessed April 14, 2025).

[5] ChatGPT (2025), OpenAI. Accessed April 14, 2025. [Online]. Available: https://chat.openai.com/

[6] C. Hashemi-Pour. “Association rules.” TechTarget. https://www.techtarget.com/searchbusinessanalytics/definition/association-rules-in-data-mining (accessed April 14, 2025).

[7] J. Noble. “What is the Apriori algorithm?” IBM. https://www.ibm.com/think/topics/apriori-algorithm (accessed April 14, 2025)
