# AdventureWorks Sales Analytics — Power BI Report
### End-to-End Business Intelligence Dashboard — by Disha Jain

---

## 👋 Why I Built This

Power BI is the tool I use most in my day job. I've built production dashboards for operations, billing, and inventory teams — but I wanted a portfolio piece that shows the *full range* of what Power BI can do, not just bar charts and slicers.

This report covers everything from executive KPIs to AI-powered visuals (Key Influencers, Decomposition Tree, Q&A). It's built on the AdventureWorks dataset — a fictional bike company's sales, returns, and customer data — and structured the way I'd build a real client-facing report: clean navigation, drillthrough, and a data model that any analyst can maintain.

---

## 📋 Report Pages (8 Pages)

### 1. 🏠 Exec Dashboard
The landing page — the one a CEO or VP sees first. Shows the full business health at a glance.

**Visuals included:**
- KPI cards: **Total Revenue**, **Total Profit**, **Total Orders**, **Return Rate**
- KPI tiles with MoM comparison (vs. previous month using `CALCULATE` + `DATEADD`)
- Line chart: Monthly revenue trend over time
- Clustered bar chart: Orders by product category
- Matrix table: Top 10 products by Orders, Revenue, and Return %
- Slicers: Year filter, Continent filter
- Navigation buttons to all other pages

**Key measures powering this page:**
```dax
Total Revenue = SUMX(Sales, Sales[OrderQuantity] * RELATED('Product Lookup'[ProductPrice]))

Total Profit = [Total Revenue] - [Total Cost]

Return Rate = DIVIDE([Total Returns], [Total Orders], 0)

Previous Month Revenue = CALCULATE([Total Revenue], DATEADD('Calendar'[Date], -1, MONTH))
```

---

### 2. 🗺️ Map
Geographic breakdown of order volume by customer location.

**Visuals included:**
- Filled map: Bubble size = Total Orders by Country
- Continent slicer for regional filtering

**Why this page matters:** Quickly surfaces which markets are growing and which are flat — useful for territory planning and regional sales targets.

---

### 3. 📦 Product Detail
Drillthrough page — click any product on the Exec Dashboard to land here.

**Visuals included:**
- Dynamic product name card (changes based on drillthrough selection)
- Area chart: Product orders over time (weekly/monthly/yearly toggle)
- Line chart: Product revenue trend
- Gauge charts: Actual vs. Target for Orders, Revenue, and Profit
- Price adjustment slicer: What-if parameter that simulates how a % price change affects revenue — **this is the most technically advanced part of the report**

**What-If Parameter DAX:**
```dax
Adjusted Revenue = 
[Total Revenue] * (1 + 'Price Adjustment (%)'[Price Adjustment (%) Value])

Adjusted Profit = 
[Adjusted Revenue] - [Total Cost]
```

---

### 4. 👤 Customer Detail
Deep-dive on individual customer performance.

**Visuals included:**
- KPI cards: Unique Customers, Revenue per Customer, Total Orders
- Top customer card: dynamically shows the #1 customer by revenue
- Line chart: Orders over time for selected customer
- Donut chart: Orders split by customer Occupation
- Donut chart: Orders split by customer Income Level
- Customer table: Ranked list of customers with Orders and Revenue
- Customer metric toggle (slicer between Orders / Revenue / Profit view)
- Year filter slicer

---

### 5. 📊 Category Tooltip *(hidden — tooltip page)*
A custom tooltip that appears when hovering over category visuals on the Exec Dashboard. Shows category-level metrics inline without navigating away.

---

### 6. 💬 Q&A Page
Natural language Q&A visual — type a question in plain English and Power BI answers it.

Example questions this answers:
- *"Total revenue by year"*
- *"Top 5 customers by orders"*
- *"Return rate by category"*

---

### 7. 🌳 Decomposition Tree
AI visual that lets users break down Total Orders by any dimension interactively.

Pre-configured dimensions: Category → Subcategory → Product

**Why it's useful:** Business users can self-serve their own root-cause analysis without needing a new report page built for every question.

---

### 8. 🔑 Key Influencers
AI visual answering two business questions:

- **What drives high Total Orders?** → Segments by AnnualIncome, EducationLevel, Is Parent
- **What drives high product cost?** → Segments by Subcategory, ProductCost, Retail Price

**Why I included this:** In stakeholder meetings, the hardest question is always "why did this metric move?" Key Influencers gives a statistically-grounded answer without requiring a data scientist.

---

## 🗂️ Data Model

The report uses a **star schema** with one central fact table and multiple dimension/lookup tables:

```
Calendar (dim)          Product Lookup (dim)
     │                        │
     └──────────────┬──────────┘
                    │
              Sales (fact) ─────── Returns (fact)
                    │
         Customer Lookup (dim)
```

**Tables:**
| Table | Type | Key fields |
|-------|------|-----------|
| Sales | Fact | OrderDate, CustomerKey, ProductKey, OrderQuantity |
| Returns | Fact | ReturnDate, ProductKey, ReturnQuantity |
| Customer Lookup | Dimension | CustomerKey, FullName, AnnualIncome, Occupation |
| Product Lookup | Dimension | ProductKey, ProductName, CategoryName, ProductPrice |
| Calendar | Dimension | Date, Month, Year, Quarter |
| Measure Table | Calculated | All DAX measures isolated here |

**Why a dedicated Measure Table?**
Keeping all DAX measures in one table (not scattered across fact/dim tables) makes the report easier to maintain, easier to audit, and keeps the field list clean for end users. This is a pattern I use in all my production reports.

---

## 💡 Technical Highlights

**1. What-If Parameter (Price Adjustment)**
Lets business users simulate the revenue impact of a price change (−20% to +20%) in real time. Built using Power BI's What-If parameter feature tied to a DAX measure.

**2. Dynamic KPI Titles**
Measure names on the Customer Detail page update dynamically based on the metric slicer selection, using `SELECTEDVALUE()` in DAX.

**3. Drillthrough Navigation**
Product Detail and Customer Detail pages are drillthrough targets — right-click any product/customer and drill directly to their detail page, preserving the filter context.

**4. Custom Tooltip Page**
The Category Tooltip page is a hidden page used as a visual tooltip on the Exec Dashboard, giving richer hover-over context without cluttering the main page.

**5. MoM KPI Comparisons**
Every KPI card on the Exec Dashboard shows the delta vs. the prior month, powered by `DATEADD` time intelligence — the same pattern I use in production dashboards.

---

## 📂 Repository Structure

```
powerbi-adventureworks/
│
├── AdventureWorks_Report_FINAL.pbix    # Full Power BI report file
│
├── datasets/                                          # Raw CSV files — connect directly in Power BI Desktop
│   ├── AdventureWorks_Sales_Data_2020.csv             # Fact: sales transactions 2020
│   ├── AdventureWorks_Sales_Data_2021.csv             # Fact: sales transactions 2021
│   ├── AdventureWorks_Sales_Data_2022.csv             # Fact: sales transactions 2022
│   ├── AdventureWorks_Returns_Data.csv                # Fact: product returns
│   ├── AdventureWorks_Customer_Lookup.csv             # Dim: customer demographics
│   ├── AdventureWorks_Product_Lookup.csv              # Dim: product details and pricing
│   ├── AdventureWorks_Product_Categories_Lookup.csv   # Dim: product categories
│   ├── AdventureWorks_Product_Subcategories_Lookup.csv# Dim: product subcategories
│   ├── AdventureWorks_Calendar_Lookup.csv             # Dim: date table for time intelligence
│   └── AdventureWorks_Territory_Lookup.csv            # Dim: sales territories and regions
├── screenshots/                         # Page-by-page screenshots
│   ├── 01_exec_dashboard.png
│   ├── 02_map.png
│   ├── 03_product_detail.png
│   ├── 04_customer_detail.png
│   ├── 07_decomposition_tree.png
│   └── 08_key_influencers.png
│
├── dax_measures.md                      # All DAX measures documented
├── data_model.md                        # Data model diagram and table descriptions
└── README.md
```

---

## 🚀 How to Open This Report

1. Download and install [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (free)
2. Clone or download this repository
3. Open `AdventureWorks_Report_FINAL.pbix` in Power BI Desktop
4. The report uses imported data — no live connection needed, it opens immediately

---


## 🔗 Related Projects

This report is part of a three-project series covering the full data pipeline — from raw data to business insights:

| Project | What it covers |
|---------|---------------|
| [SQL Data Warehouse](https://github.com/dishajain-dataanalyst/sql-data-warehouse-project) | ETL pipeline, Medallion Architecture (Bronze/Silver/Gold), star schema modeling, data quality checks |
| [SQL Data Analytics](https://github.com/dishajain-dataanalyst/sql-data-analytics-project) | 14 advanced SQL scripts — EDA, time-series, segmentation, cumulative analysis, report views |
| **This repo** | End-to-end Power BI report — 8 pages, DAX measures, drillthrough, What-If analysis, AI visuals |

---

## 🛡️ License

MIT License — free to use and adapt with attribution.
Dataset: AdventureWorks (Microsoft sample data, publicly available).

---

## 👤 About Me

I'm **Disha Jain**, an Analytics & BI Professional with 3+ years of experience building Power BI dashboards, ETL pipelines, and data models in production. I'm Microsoft Power BI Certified and have delivered dashboards used by 3+ business units for daily decision-making.

Currently seeking **Data Analyst** and **BI Developer** roles.

**Skills:** Power BI · DAX · SQL · Tableau · Python · Alteryx · Azure · ETL · Data Modeling

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/dishadineshjain)
[![SQL Warehouse](https://img.shields.io/badge/Related-SQL_Warehouse_Project-blue?style=for-the-badge)](https://github.com/dishajain-dataanalyst/sql-data-warehouse-project)
[![SQL Analytics](https://img.shields.io/badge/Related-SQL_Analytics_Project-blue?style=for-the-badge)](https://github.com/dishajain-dataanalyst/sql-data-analytics-project)
