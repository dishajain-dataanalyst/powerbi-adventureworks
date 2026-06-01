# DAX Measures Reference
**Project:** AdventureWorks Sales Analytics  
**Author:** Disha Jain

All measures live in a dedicated **Measure Table** to keep the data model clean and auditable.

---

## 💰 Revenue & Profit

```dax
Total Revenue = 
SUMX(
    Sales,
    Sales[OrderQuantity] * RELATED('Product Lookup'[ProductPrice])
)

Total Cost = 
SUMX(
    Sales,
    Sales[OrderQuantity] * RELATED('Product Lookup'[ProductCost])
)

Total Profit = [Total Revenue] - [Total Cost]

Profit Margin = DIVIDE([Total Profit], [Total Revenue], 0)

Revenue per Customer = DIVIDE([Total Revenue], [Total Customers], 0)
```

---

## 📦 Orders & Returns

```dax
Total Orders = DISTINCTCOUNT(Sales[OrderNumber])

Total Returns = COUNT(Returns[ReturnQuantity])

Return Rate = DIVIDE([Total Returns], [Total Orders], 0)

Total Customers = DISTINCTCOUNT(Sales[CustomerKey])
```

---

## 📅 Time Intelligence (MoM Comparisons)

```dax
Previous Month Revenue = 
CALCULATE(
    [Total Revenue],
    DATEADD('Calendar'[Date], -1, MONTH)
)

Previous Month Orders = 
CALCULATE(
    [Total Orders],
    DATEADD('Calendar'[Date], -1, MONTH)
)

Previous Month Returns = 
CALCULATE(
    [Total Returns],
    DATEADD('Calendar'[Date], -1, MONTH)
)

Revenue Target = [Previous Month Revenue] * 1.1
Order Target   = [Previous Month Orders]  * 1.1
Profit Target  = [Previous Month Revenue] * 0.9
```

---

## 🔢 What-If: Price Adjustment

```dax
-- Tied to the Price Adjustment (%) What-If parameter slicer

Adjusted Revenue = 
[Total Revenue] * (1 + 'Price Adjustment (%)'[Price Adjustment (%) Value])

Adjusted Profit = 
[Adjusted Revenue] - [Total Cost]
```

---

## 🏅 Dynamic Customer Metrics

```dax
-- Powers the metric toggle slicer on Customer Detail page

Customer Metric = 
SWITCH(
    SELECTEDVALUE('Customer Metric Selection'[Customer Metric]),
    "Orders",  [Total Orders],
    "Revenue", [Total Revenue],
    "Profit",  [Total Profit],
    [Total Orders]
)
```

---

## Design Note

All measures are stored in a single **Measure Table** (an empty table created just to hold measures). This is a best practice I follow in all production reports because:
- Keeps fact and dimension tables clean — no measures mixed with data columns
- Makes it easy to find and audit all business logic in one place
- Prevents accidental use of implicit measures by report consumers
