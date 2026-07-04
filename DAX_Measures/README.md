# 📘 DAX Measures Reference
This document contains the DAX measures created throughout my **15-Day Power BI DAX Learning Journey** using a retail sales dataset.
The focus was not only on writing DAX formulas but also on solving real business problems through data analysis.
# DAX Topics Covered

- SUMX
- CALCULATE
- ALL
- ALLEXCEPT
- FILTER
- AVERAGEX
- RANKX
- DATEADD
- SAMEPERIODLASTYEAR
- TOTALYTD
- ALLSELECTED
- SWITCH(TRUE())
- Disconnected Tables
- DATESINPERIOD
- DIVIDE

# Day 1 — Revenue Calculation
## Business Problem
Calculate revenue after applying discounts.
## Measure
Net Revenue =
SUMX(
    Sales,
    Sales[Qty] *
    Sales[Unit Price] *
    (1 - Sales[Discount])
)
## Learning
- SUMX iterates through every sales row.
- Performs row-by-row calculation.
- Returns total net revenue

# Day 2 — Contribution %
## Business Problem
Find each city's contribution to total revenue.

## Measure
Revenue % =
DIVIDE(
    [Net Revenue],
    CALCULATE(
        [Net Revenue],
        ALL(Customers[City])
    )
)
## Learning
- ALL removes city filters.
- CALCULATE changes filter context
# Day 3 — ALLEXCEPT
## Business Problem
Calculate the contribution of products within their own category while ignoring product-level filters.
## Measure
% of Category =
DIVIDE(
    [Net Revenue],
    CALCULATE(
        [Net Revenue],
        ALLEXCEPT(
            Products,
            Products[Category]
        )
    )
)
## Learning

- ALLEXCEPT removes all filters except the specified one.
- Useful for comparing products within the same category.
- Ideal for category-level contribution analysis.

# Day 4 — RANKX
## Business Problem
Rank products based on revenue to identify the best-performing products.
## Measure
Products by Rank =
RANKX(
    ALL(Products[Product]),
    [Net Revenue],
    ,
    DESC,
    DENSE
)
## Learning
- RANKX assigns rankings based on a measure.
- ALL removes product filters before ranking.
- DENSE ranking avoids skipped ranking numbers.
# Day 5 — FILTER
## Business Problem

Find how many customers spend above the average customer spend.

## Measure
Customers Above Average =
VAR AvgCustomerSpend =
AVERAGEX(
    VALUES(Customers[Customer_ID]),
    [Net Revenue]
)

RETURN
COUNTROWS(
    FILTER(
        VALUES(Customers[Customer_ID]),
        [Net Revenue] > AvgCustomerSpend
    )
)
```

## Learning

- FILTER creates a custom filter table.
- AVERAGEX calculates averages over customers.
- Demonstrates combining iterators with logical conditions.

---

# Day 6 — AVERAGEX & Context Transition
**## Business Problem

Calculate the average revenue generated per customer.

## Measure

```DAX
Average Customer Spend =
AVERAGEX(
    VALUES(Customers[Customer_ID]),
    [Net Revenue]
)
```

## Learning

- AVERAGEX iterates over each customer.
- Measures automatically perform context transition.
- Useful for customer-level KPIs.

---

# Day 7 — DATEADD

## Business Problem

Compare current month revenue with the previous month.

## Measure

```DAX
Revenue LM =
CALCULATE(
    [Net Revenue],
    DATEADD(
        Calendar[Date],
        -1,
        MONTH
    )
)
```

## Learning

- DATEADD shifts the current filter context.
- Enables Month-over-Month analysis.
- Requires a proper Calendar table.

---

# Day 8 — SAMEPERIODLASTYEAR

## Business Problem

Compare current sales with the same period last year.

## Measure

```DAX
Revenue LY =
CALCULATE(
    [Net Revenue],
    SAMEPERIODLASTYEAR(
        Calendar[Date]
    )
)
```

## Learning

- SAMEPERIODLASTYEAR shifts dates exactly one year back.
- Essential for Year-over-Year analysis.
- Works correctly only with a marked Date table.

---

# Day 9 — TOTALYTD

## Business Problem

Calculate cumulative revenue from the beginning of the fiscal year.

## Measure

```DAX
Revenue YTD =
TOTALYTD(
    [Net Revenue],
    Calendar[Date],
    ,
    "03/31"
)
```

## Learning

- TOTALYTD creates running totals.
- Fiscal year end can be customized.
- Commonly used in financial reporting.

---

# Day 10 — Calendar Filter

## Business Problem

Calculate revenue generated only on weekdays.

## Measure

```DAX
Weekday Revenue =
CALCULATE(
    [Net Revenue],
    Calendar[Is_Weekend] = 0
)
```

## Learning

- Filters from the Calendar table propagate to the Sales table.
- Demonstrates the power of relationships in Power BI.
- Useful for operational analysis.

---

# Day 11 — ALLSELECTED

## Business Problem

Calculate each city's contribution only within the selected cities.

## Measure

```DAX
% of Selected Cities =
DIVIDE(
    [Net Revenue],
    CALCULATE(
        [Net Revenue],
        ALLSELECTED(Customers[City])
    )
)
```

## Learning

- ALLSELECTED respects slicers.
- Provides interactive percentage calculations.
- Ideal for dashboards with multiple filters.

---

# Day 12 — SWITCH(TRUE())

## Business Problem

Categorize customers based on spending.

## Measure

```DAX
Customer Segment =
SWITCH(
    TRUE(),
    [Net Revenue] >= 500000, "High Value",
    [Net Revenue] >= 250000, "Mid Value",
    "Low Value"
)
```

## Learning

- SWITCH(TRUE()) simplifies multiple IF statements.
- Creates dynamic business classifications.
- Improves readability of complex logic.

---

# Day 13 — Disconnected Table

## Business Problem

Allow users to dynamically switch KPI values from a slicer.

## Measure

```DAX
Selected KPI =
SWITCH(
    SELECTEDVALUE(Metric[Metric]),
    "Revenue", [Net Revenue],
    "Profit", [Total Profit],
    "Margin", [Profit Margin %],
    "AOV", [Average Order Value]
)
```

## Learning

- Disconnected tables do not filter the data model.
- SWITCH returns different measures dynamically.
- Creates flexible and interactive dashboards.

---

# Day 14 — DATESINPERIOD

## Business Problem

Display a rolling 3-month average revenue.

## Measure

```DAX
Rolling 3 Month Average =
DIVIDE(
    CALCULATE(
        [Net Revenue],
        DATESINPERIOD(
            Calendar[Date],
            MAX(Calendar[Date]),
            -3,
            MONTH
        )
    ),
    3
)
```

## Learning

- DATESINPERIOD builds rolling time windows.
- Smooths trends by reducing monthly fluctuations.
- Frequently used in executive dashboards.

---

# Day 15 — Executive Summary Card

## Business Problem

Build executive KPI cards showing the most important business metrics.

## Measures

```DAX
Total Revenue =
SUMX(
    Sales,
    Sales[Qty] *
    Sales[Unit Price] *
    (1 - Sales[Discount])
)
```

```DAX
Total Profit =
[Total Revenue] -
SUMX(
    Sales,
    Sales[Qty] * Sales[Unit Cost]
)
```

```DAX
Profit Margin % =
DIVIDE(
    [Total Profit],
    [Total Revenue]
)
```

```DAX
Average Order Value =
DIVIDE(
    [Total Revenue],
    DISTINCTCOUNT(Sales[Order_ID])
)
```

## Learning

- Executive dashboards begin with reliable KPI measures.
- DIVIDE prevents errors when the denominator is zero.
- Reusable measures make dashboards easier to maintain.
- The entire learning journey comes together in one executive summary.

---

# Overall Learning Summary

Over 15 days, I built a complete DAX foundation using a single retail dataset. The journey covered data modeling, filter context, ranking, iterators, time intelligence, dynamic measures, and executive dashboard design.

The most valuable lesson was understanding **filter context**. Every DAX function—whether `ALL`, `FILTER`, `CALCULATE`, or `DATESINPERIOD`—helps reshape filter context to answer a specific business question.

This project strengthened my ability to write reusable, business-focused DAX measures and build interactive Power BI dashboards that support data-driven decision-making.

...
