# Data Model Reference
**Project:** AdventureWorks Sales Analytics — Power BI  
**Author:** Disha Jain  
**Schema type:** Star Schema  

---

## Overview

The report uses a classic star schema — two fact tables at the center, five dimension/lookup tables surrounding them, and one dedicated measure table. All relationships are one-to-many, with the dimension side always on the "one" end.

```
                    Calendar Lookup
                         │ (1)
                         │
Territory Lookup ── (*)  │  (*) ── Customer Lookup
       │ (1)        Sales (fact)        (1) │
       │                │                   │
       └────────────────┤                   │
                        │ (*)               │
                  Returns (fact)            │
                        │ (*)               │
                        │                   │
              Product Lookup (1) ───────────┘
                        │ (1)
                        │
          Product Subcategories Lookup (1)
                        │ (1)
                        │
           Product Categories Lookup (1)
```

---

## Tables

### 📊 Sales (Fact Table)
The primary fact table — one row per line item per order.

| Column | Type | Description |
|--------|------|-------------|
| OrderDate | Date | Date the order was placed |
| StockDate | Date | Date stock was allocated |
| OrderNumber | Text | Unique order identifier |
| CustomerKey | Integer | Foreign key → Customer Lookup |
| TerritoryKey | Integer | Foreign key → Territory Lookup |
| ProductKey | Integer | Foreign key → Product Lookup |
| OrderLineItem | Integer | Line item number within the order |
| OrderQuantity | Integer | Number of units ordered |

---

### 📊 Returns (Fact Table)
One row per return transaction.

| Column | Type | Description |
|--------|------|-------------|
| ReturnDate | Date | Date the return was processed |
| TerritoryKey | Integer | Foreign key → Territory Lookup |
| ProductKey | Integer | Foreign key → Product Lookup |
| ReturnQuantity | Integer | Number of units returned |

---

### 👥 Customer Lookup (Dimension)
One row per customer.

| Column | Type | Description |
|--------|------|-------------|
| CustomerKey | Integer | Primary key |
| Prefix | Text | Title (Mr., Mrs., etc.) |
| FirstName | Text | Customer first name |
| LastName | Text | Customer last name |
| BirthDate | Date | Date of birth |
| MaritalStatus | Text | Single / Married |
| Gender | Text | M / F |
| EmailAddress | Text | Customer email |
| AnnualIncome | Currency | Annual household income |
| TotalChildren | Integer | Number of children |
| EducationLevel | Text | Highest education level |
| Occupation | Text | Job category |
| HomeOwner | Text | Y / N |
| FullName | Calculated | FirstName & " " & LastName |

---

### 📦 Product Lookup (Dimension)
One row per product SKU.

| Column | Type | Description |
|--------|------|-------------|
| ProductKey | Integer | Primary key |
| ProductSubcategoryKey | Integer | Foreign key → Product Subcategories Lookup |
| ProductSKU | Text | Stock keeping unit code |
| ProductName | Text | Full product name |
| ModelName | Text | Product model |
| ProductDescription | Text | Full description |
| ProductColor | Text | Color variant |
| ProductSize | Text | Size variant |
| ProductStyle | Text | Style classification |
| ProductCost | Currency | Unit cost |
| ProductPrice | Currency | Retail price |

---

### 🗂️ Product Subcategories Lookup (Dimension)
One row per subcategory — bridges Products to Categories.

| Column | Type | Description |
|--------|------|-------------|
| ProductSubcategoryKey | Integer | Primary key |
| SubcategoryName | Text | Subcategory name (e.g. Helmets, Tires) |
| ProductCategoryKey | Integer | Foreign key → Product Categories Lookup |

---

### 🗂️ Product Categories Lookup (Dimension)
One row per top-level category.

| Column | Type | Description |
|--------|------|-------------|
| ProductCategoryKey | Integer | Primary key |
| CategoryName | Text | Category name (Bikes, Accessories, Clothing) |

---

### 📅 Calendar Lookup (Dimension)
One row per date — powers all time intelligence DAX measures.

| Column | Type | Description |
|--------|------|-------------|
| Date | Date | Primary key — full date |
| Day of Week | Integer | 1 (Sunday) to 7 (Saturday) |
| Day Name | Text | Monday, Tuesday, etc. |
| Day of Month | Integer | 1–31 |
| Day of Year | Integer | 1–366 |
| Month Number | Integer | 1–12 |
| Month Name | Text | January, February, etc. |
| Month Short | Text | Jan, Feb, etc. |
| Quarter | Integer | 1–4 |
| Year | Integer | e.g. 2020, 2021, 2022 |
| Start of Week | Date | Monday of that week |
| Start of Month | Date | First day of that month |
| Start of Quarter | Date | First day of that quarter |
| Start of Year | Date | First day of that year |

---

### 🌍 Territory Lookup (Dimension)
One row per sales territory.

| Column | Type | Description |
|--------|------|-------------|
| SalesTerritoryKey | Integer | Primary key |
| Region | Text | Region name (e.g. Northwest, France) |
| Country | Text | Country name |
| Continent | Text | North America, Europe, Pacific |

---

### 🧮 Measure Table (Calculated)
An empty table created solely to house all DAX measures — no data columns, no relationships. This is a deliberate design decision to keep all business logic in one place.

See [`dax_measures.md`](./dax_measures.md) for the full list of measures with code.

---

## Relationships

| From (many side) | To (one side) | Join key |
|-----------------|---------------|----------|
| Sales | Calendar Lookup | OrderDate → Date |
| Sales | Customer Lookup | CustomerKey |
| Sales | Territory Lookup | TerritoryKey |
| Sales | Product Lookup | ProductKey |
| Returns | Calendar Lookup | ReturnDate → Date |
| Returns | Territory Lookup | TerritoryKey |
| Returns | Product Lookup | ProductKey |
| Product Lookup | Product Subcategories Lookup | ProductSubcategoryKey |
| Product Subcategories Lookup | Product Categories Lookup | ProductCategoryKey |

All relationships are **single direction** (dimension → fact). No bidirectional filters — this keeps DAX measure calculations predictable and avoids ambiguous filter paths.

---

## Design Decisions

**Why two separate Sales fact tables by year rather than one?**
The source data came as three separate CSVs (2020, 2021, 2022). They were appended into a single Sales table in Power Query using **Append Queries** — so the final model has one Sales fact table, not three. This keeps the star schema clean.

**Why no direct relationship between Sales and Product Categories?**
Categories connect to Sales through the chain: Sales → Product Lookup → Product Subcategories → Product Categories. This is intentional — it avoids a many-to-many relationship and keeps the hierarchy intact. DAX handles cross-table filtering through this chain automatically.

**Why a dedicated Measure Table?**
Keeping all DAX measures isolated in one empty table prevents them from being mixed with data columns, makes them easy to find and audit, and keeps the field list clean for end users building their own visuals.
