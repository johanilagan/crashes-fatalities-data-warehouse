# 🚗 Crash & Fatalities in Australia — Data Warehouse Project

A comprehensive data warehousing and analytics project analyzing road crash and fatality data in Australia using PostgreSQL, Python, and Tableau. This project covers everything from ETL to dimensional modeling, business queries, and data visualization — supporting public safety initiatives like Vision Zero.

---

## 📂 Project Structure

```
crash-fatalities-data-warehouse/
│
├── etl-process/                      
│   └── Crash-Data-Cleaning-ETL-Mining.ipynb
│
├── data/                    
│   ├── raw
│        └── raw-data-compressed.zip
│   └── processed
          ├── Dim_age.csv
│         ├── Dim_date.csv
│         ├── Dim_gender.csv
│         ├── Dim_holiday.csv
│         ├── Dim_location.csv
│         ├── Dim_outcome.csv
│         ├── Dim_person.csv
│         ├── Dim_road.csv
│         ├── Dim_time.csv
│         ├── Dim_vehicle.csv
│         └── Fact_Crashes.csv
│
├── starnet/                  
│   ├── query-1-starnet.png
│   ├── query-2-starnet.png
│   ├── query-3-starnet.png
│   ├── query-4-starnet.png
│   ├── query-5-starnet.png
│   ├── schema.png
│   └── starnet.png
│
├── charts/          
│   ├── query-1-chart.png
│   ├── query-2-chart.png
│   ├── query-3-chart.png
│   ├── query-4-chart.png
│   └── query-5-chart.png
│
├── Crash_Fatalities_Complete_Report.md
└── README.md
```


## 🧠 Project Goals

- Clean and transform crash & fatality datasets for relational warehousing
- Design a star schema with fact/dimension tables
- Implement the ETL pipeline using Python and PostgreSQL
- Answer real-world business questions using analytical queries
- Use association rule mining for pattern discovery
- Create visualizations to support road safety recommendations

---

## 🔧 Tools & Technologies

- Data Cleaning: `pandas`, `numpy`, `mlxtend` in Jupyter Notebook
- Data Warehousing: `PostgreSQL`, star schema, ERD
- Visualization: `Tableau Desktop`
- Association Rules: Apriori algorithm

---

## 🗃️ Data Sources

- [BITRE Road Fatalities Database](https://www.bitre.gov.au/statistics/safety/fatal_road_crash_database)  
  Licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)

---

## 📊 Sample Business Queries

1. Which state recorded the most multiple crashes in the last 10 years?
2. How many fatalities occurred on Monday bus crashes?
3. What age group is most involved in nighttime crashes in NSW, QLD, VIC in 2021?
4. How does speed limit affect crashes during Christmas?
5. Which gender and month combination had the most fatalities in major cities?

🔗 All queries are visualized using Tableau. See `charts/` folder.

---

## 🔍 Key Insights

- NSW has the highest multiple crash rate
- Speed limits of 100 km/h are highly associated with crash fatalities
- Male drivers in September dominate fatality records in urban areas

---

## 🤖 Association Rule Mining

Using the Apriori algorithm, the following top rules were extracted:

- `{100, Weekday} → {Driver}`
- `{Male, 100} → {Driver}`
- `{Single, 100} → {Driver}`

---

## ✅ Recommendations

1. Monitor 100 km/h roads more strictly
2. Investigate male, weekday, and single-vehicle crash patterns
3. Upgrade driver training programs across Australia

---

## 🙋🏻‍♂️ Author

**Johan Carlo Ilagan** 
- [GitHub](https://github.com/johanilagan)
- [LinkedIn](www.linkedin.com/in/johan-ilagan)
  
Created as part of a data warehousing portfolio project.  

---

## 📥 How to Use

1. Clone the repo  
2. Explore the Jupyter Notebook in `etl-process/`  
3. Review schema design and SQL in `starnet/`  
4. View dashboards in `charts/`
5. View full analysis and report in `Crash_Fatalities_Complete_Report.md`

---

## 📎 License

This project uses public datasets and complies with the **Copyright Act 1968** and the [CC BY 4.0 License](https://creativecommons.org/licenses/by/4.0/).
