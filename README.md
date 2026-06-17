# E-Commerce Sales Dashboard — Power BI Project

**Author:** Naren Karthikeyan  
**Tool:** Microsoft Power BI Desktop  
**Dataset:** Indian E-Commerce Sales (2022–2023) — 4 tables, 10,760 total rows  
**Domain:** E-Commerce / Retail  
**GitHub:** github.com/A-NarenKarthikeyan
**LinkedIn:** linkedin.com/in/narenkarthikeyana/  

---

## Project Overview

An end-to-end Power BI project built on a multi-table Indian e-commerce dataset — covering data loading, Power Query cleaning, star schema data modelling, DAX measure writing, and interactive dashboard creation. The project follows a business-problem-first approach where every visualisation is driven by a real stakeholder question rather than arbitrary chart selection.

---

## Dataset Description

Four relational tables designed to mirror how real e-commerce data lives in a database:

| Table | Rows | Type | Description |
|---|---|---|---|
| Orders | ~7,900 | Fact | Transactional data — sales, profit, quantity, shipping, payment |
| Customers | 2,000 | Dimension | Customer demographics — city, state, segment |
| Products | 28 | Dimension | Product catalogue — category, price, cost, margin |
| Dates | 730 | Dimension | Full date table for time intelligence |

**Product categories:** Electronics, Fashion, Home, Sports, Books

**Customer segments:** Retail, Business, Premium

**Indian cities covered:** Mumbai, Delhi, Bangalore, Chennai, Hyderabad, Pune, Kolkata, Ahmedabad, Jaipur, Surat, Lucknow, Kochi

---

## Phase 1 — Power Query (Data Cleaning)

Each table was validated and cleaned inside Power Query Editor before loading:

| Table | Action Taken |
|---|---|
| Orders | Replaced null Payment_Method with "Unknown", removed duplicates, removed negative profit rows |
| Customers | Validated — clean, Join_Date confirmed as Date type |
| Products | Added custom column `Profit_Margin_Pct = (Unit_Price - Unit_Cost) / Unit_Price * 100` |
| Dates | Validated — 730 consecutive dates, no gaps, marked as official Date Table |

---

## Phase 2 — Data Modelling (Star Schema)

Built a Star Schema with Orders as the central Fact table and 3 Dimension tables:

```
           [Dates]
              |
           (1:*)
              |
(1:*)——— [Orders] ———(1:*)
|                        |
[Customers]          [Products]
```

**Relationships:**
| From | To | Cardinality | Cross Filter |
|---|---|---|---|
| Customers[Customer_ID] | Orders[Customer_ID] | One to Many | Single |
| Products[Product_ID] | Orders[Product_ID] | One to Many | Single |
| Dates[Date] | Orders[Order_Date] | One to Many | Single |

**Why Single cross-filter direction:**
Dimension tables filter the Fact table — not the other way around. This prevents ambiguous filter paths and incorrect DAX calculations.

---

## Phase 3 — DAX Measures

All measures written using best practices — DIVIDE for safe division, VAR/RETURN for readability, ISINSCOPE for correct total row handling.

### Base Measures
```dax
Total Revenue = SUM(Orders[Sales])

Total Profit = SUM(Orders[Profit])

Total Orders = DISTINCTCOUNT(Orders[Order_ID])

AOV = DIVIDE([Total Revenue], [Total Orders], 0)

Profit Margin % = DIVIDE([Total Profit], [Total Revenue], 0) * 100
```

### CALCULATE Measures
```dax
Electronics Revenue = 
CALCULATE([Total Revenue], Products[Category] = "Electronics")

Delivered Revenue = 
CALCULATE([Total Revenue], Orders[Order_Status] = "Delivered")

Revenue 2023 = 
CALCULATE([Total Revenue], Dates[Year] = 2023)

Revenue % of Total = 
DIVIDE([Total Revenue], CALCULATE([Total Revenue], ALL(Products)), 0) * 100
```

### Time Intelligence Measures
```dax
Revenue LY = 
CALCULATE([Total Revenue], SAMEPERIODLASTYEAR(Dates[Date]))

YoY Growth % = 
VAR CurrentY = [Total Revenue]
VAR LastYear = [Revenue LY]
VAR Growth   = DIVIDE(CurrentY - LastYear, LastYear, 0) * 100
RETURN IF(ISINSCOPE(Dates[Year]), Growth, BLANK())
```

### Operations Measures
```dax
Return Rate % = 
DIVIDE(
    CALCULATE([Total Orders], Orders[Order_Status] = "Returned"),
    [Total Orders], 0) * 100

Delivery Rate % = 
DIVIDE(
    CALCULATE([Total Orders], Orders[Order_Status] = "Delivered"),
    [Total Orders], 0) * 100
```

---

## Phase 4 — Dashboard

Seven business problems solved, each driving a specific visualisation:

### Business Problem 1 — Revenue vs Profitability
**Question:** Which categories drive revenue vs actually make profit?
**Visual:** Line and Stacked Column Chart (combo)
**Finding:** Electronics = 82% of revenue but lowest margin (15.7%). Books = highest margin (51.6%) but negligible volume.

### Business Problem 2 — Growth Trend
**Question:** Are we growing compared to last year month by month?
**Visual:** Dual Line Chart (2022 vs 2023)
**Finding:** 2023 started stronger but H2 weakness dragged full year to -2.33% YoY decline.

### Business Problem 3 — Customer Segments
**Question:** Which segment is most valuable?
**Visual:** Donut Chart + AOV Card (cross-filtered)
**Finding:** Retail = 54% of revenue. Premium = highest AOV at ₹26.98K. Marketing should prioritise Premium acquisition.

### Business Problem 4 — Order Fulfilment
**Question:** How healthy is our delivery vs returns vs cancellations?
**Visual:** 100% Stacked Bar Chart by Category
**Finding:** ~82% delivery rate consistent across all categories — systemic issue, not category-specific.

### Business Problem 5 — Payment Methods
**Question:** Which payment method is most popular and which drives highest spend?
**Visual:** Side-by-side Clustered Bar Charts
**Finding:** UPI = highest volume. COD = highest AOV. Improving COD experience could lift revenue.

### Business Problem 6 — Shipping Analysis
**Question:** Does faster shipping lead to better outcomes?
**Visual:** Donut + Bar + 100% Stacked Bar
**Finding:** Same Day = highest AOV + highest delivery rate (92%). Standard = most popular but worst delivery rate.

---

## Key Business Insights

1. **Electronics dependency risk** — 82% revenue concentration in one category. Fashion and Sports offer 2.5x better margins at lower volume — scaling these reduces business risk.

2. **2023 H2 weakness** — Revenue declined -2.33% YoY driven by a sharp June-July dip. H1 2023 was actually stronger than H1 2022 — the problem is seasonal, not structural.

3. **Premium segment underutilised** — Only 14% of revenue but highest AOV. Targeted Premium acquisition campaigns could significantly improve revenue quality.

4. **Fulfilment consistency** — ~82% delivery rate across all categories suggests a systemic logistics issue rather than product-specific problem. Single operational fix could improve all categories simultaneously.

5. **COD = high value customers** — Counterintuitive finding: COD users have the highest AOV. Improving COD experience (faster delivery, easier returns) could unlock significant revenue.

6. **Speed = quality** — Same Day shipping has 92% delivery success vs Standard's lower rate. Investment in faster fulfilment infrastructure pays dividends in both customer satisfaction and revenue.

---

## Dashboard Features

- 5 KPI cards — Revenue, Profit, Orders, AOV, Profit Margin %
- Revenue vs Profit combo chart with margin % line
- Monthly trend dual line (2022 vs 2023 comparison)
- Customer segment donut with interactive AOV card
- Order status 100% stacked bar by category
- Payment method side-by-side bars
- Shipping mode analysis (3 visuals)
- Year slicer — filters entire dashboard
- Cross-filtering enabled across all visuals

---

## DAX Concepts Demonstrated

- Basic aggregations (SUM, DISTINCTCOUNT)
- Safe division (DIVIDE)
- Conditional filtering (CALCULATE)
- Context modification (ALL, ALLEXCEPT)
- Time intelligence (SAMEPERIODLASTYEAR, TOTALYTD, DATEADD)
- Variable pattern (VAR / RETURN)
- Scope detection (ISINSCOPE)
- Percentage of total pattern

---

## Data Modelling Concepts Demonstrated

- Star schema design
- Fact vs Dimension table distinction
- One-to-Many relationships
- Single cross-filter direction
- Date table configuration for time intelligence
- Power Query transformations (type fixing, null handling, custom columns)

---

## Tools Used

- Microsoft Power BI Desktop
- Power Query (M language)
- DAX (Data Analysis Expressions)

---

## Workbook Structure

| Page | Purpose |
|---|---|
| Dashboard | Final interactive dashboard — all visuals |
| Analysis | Individual visual exploration and testing |

---

*Part of an end-to-end Data Analyst preparation path covering SQL → Excel → Python → Statistics → Power BI.*
