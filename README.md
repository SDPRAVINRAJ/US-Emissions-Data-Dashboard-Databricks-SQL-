# 🌍 US GHG Emissions Dashboard — Databricks SQL

> An interactive greenhouse gas emissions analytics dashboard analyzing **3,000+ US counties** built on **Databricks SQL Free Edition**.

![Databricks](https://img.shields.io/badge/Databricks-SQL-FF3621?style=flat-square&logo=databricks&logoColor=white)
![SQL](https://img.shields.io/badge/Language-SQL-336791?style=flat-square&logo=postgresql&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-00C851?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-black?style=flat-square)

---

## 📌 Overview

This project builds a full-stack environmental analytics dashboard using Databricks SQL to explore US greenhouse gas (GHG) emissions at the county level. From raw string-formatted data to interactive geospatial maps and state rankings, the pipeline demonstrates end-to-end data wrangling and visualization entirely within Databricks SQL Free Edition.

**Total Emissions Measured: `562.46M` metric tons CO₂ equivalent**

---

## 📊 Dashboard Pages

| Page | Title | Visualizations |
|------|-------|----------------|
| 1 | **US Emissions Breakdown** | KPI counter · Mapbox bubble map |
| 2 | **Emissions Dashboard** | Scatter plot · Donut chart |
| 3 | **County Rankings** | Horizontal bar chart |

---

## 🔍 Key Findings

- **Harris County (TX)** and **Maricopa County (AZ)** are the top-emitting counties at ~9.8M and ~9.7M Mt CO₂e respectively
- **Texas** accounts for **20.63%** of top-10 state emissions — the single largest state share
- **Los Angeles County** (pop. 10M+) has significantly lower per-capita emissions than smaller industrial counties — consistent with urban density efficiency
- Emission hotspots cluster along the **Texas Gulf Coast**, **Midwest industrial belt**, and **Southeast corridor**

---

## 🗃️ Dataset

**Source table:** `emissions.default.emissions_data`

| Column | Type | Notes |
|--------|------|-------|
| `county_state_name` | STRING | Format: `"County, ST"` |
| `GHG emissions mtons CO2e` | STRING | Comma-formatted — requires cleaning |
| `population` | STRING | Comma-formatted — requires cleaning |
| `latitude` | DOUBLE | County centroid |
| `longitude` | DOUBLE | County centroid |

> ⚠️ All numeric fields are stored as comma-formatted strings and require `REPLACE` + `CAST` before arithmetic.

---

## 🧠 SQL Queries

### 1. Geospatial Map — Emissions by Location
```sql
SELECT latitude, longitude,
       `GHG emissions mtons CO2e` AS Emissions
FROM `emissions`.`default`.`emissions_data`
```

### 2. Per-Capita Emissions (Population-Normalized)
```sql
SELECT
    county_state_name,
    population,
    CAST(REPLACE(`GHG emissions mtons CO2e`, ',', '') AS DOUBLE)
        / NULLIF(CAST(REPLACE(population, ',', '') AS DOUBLE), 0)
        AS Emissions_per_person
FROM `emissions`.`default`.`emissions_data`
ORDER BY Emissions_per_person DESC;
```

### 3. Top 10 States by Total Emissions
```sql
SELECT
    TRIM(SPLIT(county_state_name, ',')[1]) AS state,
    SUM(TRY_CAST(REPLACE(`GHG emissions mtons CO2e`, ',', '') AS DOUBLE))
        AS total_emissions_mtons
FROM `emissions`.`default`.`emissions_data`
WHERE TRY_CAST(REPLACE(`GHG emissions mtons CO2e`, ',', '') AS DOUBLE) IS NOT NULL
GROUP BY state
ORDER BY total_emissions_mtons DESC
LIMIT 10;
```

### 4. National Total (KPI Card)
```sql
SELECT SUM(CAST(REPLACE(`GHG emissions mtons CO2e`, ',', '') AS DOUBLE))
    AS Total_Emissions
FROM `emissions`.`default`.`emissions_data`
```

### 5. Top 10 Counties by Total Emissions
```sql
SELECT
    SPLIT(county_state_name, ',')[0] AS county_name,
    population,
    CAST(REPLACE(`GHG emissions mtons CO2e`, ',', '') AS DOUBLE) AS Total_Emissions
FROM `emissions`.`default`.`emissions_data`
ORDER BY Total_Emissions DESC
LIMIT 10;
```

---

## 🏗️ Tech Stack

| Component | Tool |
|-----------|------|
| Data Platform | Databricks SQL Free Edition |
| Query Engine | Databricks Serverless SQL Warehouse |
| Data Catalog | Unity Catalog (`emissions.default`) |
| Geospatial Map | Mapbox (via Databricks) |
| Visualizations | Databricks SQL Dashboards |

---



## 🔮 Future Enhancements

- [ ] Multi-year trend analysis with Delta Live Tables
- [ ] Sector-level breakdowns (transport, industrial, residential)
- [ ] Parameterized dashboard filters by state / emission threshold
- [ ] Census API integration for real-time population estimates
- [ ] Choropleth polygon map (county boundary shapefiles)

---

## 👤 Author

Built as part of a data engineering & analytics project using Databricks SQL Free Edition.

---

## 📄 License

This project is licensed under the MIT License.
