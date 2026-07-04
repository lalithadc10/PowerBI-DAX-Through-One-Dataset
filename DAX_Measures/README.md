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
% of Total = 
DIVIDE(
    [Net Revenue],
    CALCULATE(
        [Net Revenue],
        ALL(customers[city])
    ),0
)
## Learning
- ALL removes city filters.
- CALCULATE changes filter context
# Day 3 — ALLEXCEPT
## Business Problem
Calculate the contribution of products within their own category while ignoring product-level filters.
## Measure
% of category = DIVIDE([Net revenue],[category_revenue],0)
category_revenue = CALCULATE([Net revenue],
 ALLEXCEPT(products,products[category]))

## Learning

- ALLEXCEPT removes all filters except the specified one.
- Useful for comparing products within the same category.
- Ideal for category-level contribution analysis.

# Day 4 — RANKX
## Business Problem
Rank products based on revenue to identify the best-performing products.
## Measure
Products by Rank = RANKX(ALL(products[product]),[Net revenue], ,DESC)
## Learning
- RANKX assigns rankings based on a measure.
- ALL removes product filters before ranking.
- DENSE ranking avoids skipped ranking numbers.
# Day 5 — FILTER
## Business Problem
Find how many customers spend above the average customer spend.
## Measure
Customers Above Average = 
COUNTROWS(
    FILTER(
        VALUES(customers[customer_id]),
        [Net revenue] > [Avg Customer Spend (With ALL)]
    )
)
Above Average Flag = 
IF(
    [Net revenue] > [Avg Customer Spend (With ALL)],
    1,
    0
)

## Learning

- FILTER creates a custom filter table.
- AVERAGEX calculates averages over customers.
- Demonstrates combining iterators with logical conditions.

---

# Day 6 — AVERAGEX & Context Transition
**## Business Problem

Calculate the average revenue generated per customer.

## Measure
Avg Customer Spend (With ALL) = 
CALCULATE(
    AVERAGEX(
        VALUES(customers[customer_id]),
        [Net Revenue]
    ),
    ALL(customers[customer_id])
)

## Learning

- AVERAGEX iterates over each customer.
- Measures automatically perform context transition.
- Useful for customer-level KPIs.

---

# Day 7 — DATEADD

## Business Problem

Compare current month revenue with the previous month.

## Measure
Revenue LM = 
CALCULATE([Net revenue],
DATEADD(calender[date],-1,MONTH))

## Learning

- DATEADD shifts the current filter context.
- Enables Month-over-Month analysis.
- Requires a proper Calendar table.
# Day 8 — SAMEPERIODLASTYEAR

## Business Problem

Compare current sales with the same period last year.

## Measure
Revenue LY = 
CALCULATE(
    [Net revenue],
    SAMEPERIODLASTYEAR(calender[date])
)

## Learning

- SAMEPERIODLASTYEAR shifts dates exactly one year back.
- Essential for Year-over-Year analysis.
- Works correctly only with a marked Date table.

---

# Day 9 — TOTALYTD

## Business Problem

Calculate cumulative revenue from the beginning of the fiscal year.

## Measure
Revenue YTD = 
TOTALYTD(
    [Net revenue],
    calender[date],
    "03/31"
)

## Learning
- TOTALYTD creates running totals.
- Fiscal year end can be customized.
- Commonly used in financial reporting.

# Day 10 — Calendar Filter

## Business Problem

Calculate revenue generated only on weekdays.

## Measure
Weekday Revenue = 
CALCULATE(
    [Net revenue],
    calender[is_weekend] = 0
)
Weekend Revenue = 
CALCULATE(
    [Net revenue],
    calender[is_weekend] = 1
)
## Learning

- Filters from the Calendar table propagate to the Sales table.
- Demonstrates the power of relationships in Power BI.
- Useful for operational analysis.

---

# Day 11 — ALLSELECTED

## Business Problem

Calculate each city's contribution only within the selected cities.

## Measure
% of Selected Cities = 
DIVIDE(
    [Net Revenue],
    CALCULATE(
        [Net Revenue],
        ALLSELECTED(customers[city])
    ),0
)

## Learning

- ALLSELECTED respects slicers.
- Provides interactive percentage calculations.
- Ideal for dashboards with multiple filters.

---

# Day 12 — SWITCH(TRUE())

## Business Problem

Categorize customers based on spending.

## Measure
customer segment = SWITCH(
    TRUE(),
    [Net revenue]>= 6,"High",
    [Net revenue]>= 4,"Mid",
    "Low"
)

## Learning

- SWITCH(TRUE()) simplifies multiple IF statements.
- Creates dynamic business classifications.
- Improves readability of complex logic.

---

# Day 13 — Disconnected Table

## Business Problem

Allow users to dynamically switch KPI values from a slicer.

## Measure
Selected KPI = 
SWITCH(
    SELECTEDVALUE('Metric selector'[Metric]),
    "Revenue", [Net Revenue],
    "Profit", [Total Profit]
)

## Learning

- Disconnected tables do not filter the data model.
- SWITCH returns different measures dynamically.
- Creates flexible and interactive dashboards.

---

# Day 14 — DATESINPERIOD

## Business Problem

Display a rolling 3-month average revenue.

## Measure
3 Month Rolling Avg = 
DIVIDE(
    CALCULATE(
        [Net Revenue],
        DATESINPERIOD(
            'calender'[date],
            MAX('calender'[date]),
            -3,
            MONTH
        )
    ),
    3
)


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

Profit = [Net revenue]-[Total Cost]

Profit Margin % = 
DIVIDE(
    [Total profit],
    [Net revenue],0
)
avg_ordervalue = DIVIDE([Net revenue],DISTINCTCOUNT(sales[order_id]),0)*100000

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
