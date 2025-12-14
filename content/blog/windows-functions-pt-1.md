---
title: "Window Functions in SQL, Part 1: The Philosophy"
date: 2025-12-12
draft: false
tags: ["sql", "data", "window functions", "tutorial"]
---

If you've ever written a SQL query that needed to compare each row to other rows in the same table, you've probably felt the pain. Maybe you needed to rank salespeople within each department, or calculate a running total of daily revenue, or compare today's sales to yesterday's.

The traditional tools - subqueries, self-joins, and multiple passes over the data - often leave you with queries that are hard to read, hard to maintain, and slow to run.

Window functions change that.

But before we write a single line of code, we need to understand what they are and why they work the way they do. This isn't about memorizing syntax. It's about changing how you think about rows and calculations in SQL.

## What a Window Function Actually Does

Let's start with the most important idea: a window function lets you perform a calculation *across* rows while keeping each individual row visible.

Think about the regular `SUM()` function with `GROUP BY`:

```sql
SELECT department, SUM(sales) 
FROM orders 
GROUP BY department;
```

**Result:**
```
department  | sum
------------|-----
Electronics | 8500
Clothing    | 4200
Home        | 3100
```

This collapses all rows in each department into a single total. You lose the individual orders.

A window function does something different:

```sql
SELECT 
    order_id,
    department,
    sales,
    SUM(sales) OVER (PARTITION BY department) as dept_total
FROM orders
ORDER BY department, order_id;
```

**Result:**
```
order_id | department  | sales | dept_total
---------|-------------|-------|-----------
101      | Electronics | 500   | 8500
102      | Electronics | 800   | 8500  
103      | Electronics | 1200  | 8500
104      | Electronics | 6000  | 8500
201      | Clothing    | 300   | 4200
202      | Clothing    | 900   | 4200
203      | Clothing    | 1500  | 4200
204      | Clothing    | 1500  | 4200
```

Here, every original row stays in the result. The `SUM(sales)` is still calculated per department, but instead of collapsing the rows, it adds the total as a new column on every row.

That `OVER (PARTITION BY department)` clause is the "window." It defines which rows are included in the calculation for each row.

## The Three Pieces of the Puzzle

Every window function has three components:

1. **The function itself** - What to calculate (`SUM`, `AVG`, `ROW_NUMBER`, etc.)
2. **The `OVER()` clause** - Where to calculate it  
3. **The window definition** - Which rows are in the window (`PARTITION BY`, `ORDER BY`, frame)

## The "Window" Metaphor

Picture your result set as a long list of rows. Now imagine a sliding glass window that you can place over different sections of that list.

When you write `SUM(sales) OVER (PARTITION BY department)`, you're saying:  
"For each row, create a window that includes only rows from the same department, then calculate the sum of sales within that window."

The window moves with each row. For a row in the Electronics department, the window includes all Electronics rows. For a row in Clothing, the window includes all Clothing rows.

## Why This Matters: Two Mindset Shifts

### Shift 1: From Collapsing to Enriching

Regular aggregates reduce many rows to few. Window functions enrich each row with context from related rows.

**Without window functions:**
```sql
-- This gives you totals, but loses detail
SELECT department, AVG(salary) 
FROM employees 
GROUP BY department;
```

**With window functions:**
```sql
-- This keeps all employees, adds context
SELECT 
    name,
    department, 
    salary,
    AVG(salary) OVER (PARTITION BY department) as dept_avg_salary,
    salary - AVG(salary) OVER (PARTITION BY department) as diff_from_avg
FROM employees;
```

**Result:**
```
name    | department | salary | dept_avg_salary | diff_from_avg
--------|------------|--------|-----------------|--------------
Mariam  | Engineering| 95000  | 87500           | 7500
Zeid    | Engineering| 80000  | 87500           | -7500
Dania   | Sales      | 65000  | 60000           | 5000
Semyon  | Sales      | 55000  | 60000           | -5000
```

### Shift 2: From Multiple Passes to One Pass

Consider the problem: "Show each employee's sales and what percentage of their department's total that represents."

Without window functions, you might write:
```sql
-- Step 1: Get department totals
WITH dept_totals AS (
    SELECT department, SUM(sales) as total
    FROM sales
    GROUP BY department
)
-- Step 2: Join back and calculate
SELECT 
    s.employee,
    s.department,
    s.sales,
    s.sales / dt.total * 100 as pct_of_dept
FROM sales s
JOIN dept_totals dt ON s.department = dt.department;
```

With window functions, it becomes:
```sql
SELECT 
    employee,
    department,
    sales,
    sales / SUM(sales) OVER (PARTITION BY department) * 100 as pct_of_dept
FROM sales;
```

The window version does it in one pass. The database calculates the department total once per department, then uses it for every row in that department.

## The Simplest Window: `PARTITION BY`

`PARTITION BY` splits your data into groups, like `GROUP BY` does. But instead of collapsing each group, it keeps all rows and calculates within each partition.

```sql
-- Compare these two side by side

-- GROUP BY (collapses)
SELECT department, COUNT(*) 
FROM employees 
GROUP BY department;

-- Window with PARTITION BY (keeps rows)
SELECT 
    name,
    department,
    COUNT(*) OVER (PARTITION BY department) as dept_count
FROM employees
ORDER BY department, name;
```

**Window function result:**
```
name    | department  | dept_count
--------|-------------|-----------
Mariam  | Engineering | 4
Zeid    | Engineering | 4
Sewa    | Engineering | 4
Dmitry  | Engineering | 4
Daria   | Sales       | 3
Vladimir| Sales       | 3
Musa    | Sales       | 3
```

The window version gives you a useful fact on every row: "You're one of X people in your department."

## When You're Ready to Order

Sometimes the order of rows matters. `ORDER BY` inside the window lets you create running totals, rankings, and comparisons:

```sql
-- Running total of sales by date
SELECT 
    sale_date,
    amount,
    SUM(amount) OVER (ORDER BY sale_date) as running_total
FROM daily_sales
ORDER BY sale_date;
```

**Result:**
```
sale_date  | amount | running_total
-----------|--------|--------------
2024-01-01 | 100    | 100
2024-01-02 | 150    | 250
2024-01-03 | 200    | 450
2024-01-04 | 50     | 500
2024-01-05 | 300    | 800
```

Here the window changes: instead of "all rows in the same partition," it becomes "all rows from the start up to this row, in this order."

## The Mental Model That Works

When I finally understood window functions, I stopped thinking about SQL as a tool that either filters or aggregates. I started thinking in terms of:

**"For this row, what can I see from nearby rows?"**

That question - and the ability to answer it in a single query - is what makes window functions feel like a different kind of SQL.

## What's Next

In the next part, we'll look at specific window functions and real problems they solve. We'll see how to rank rows without sorting the whole table, how to compare each row to the previous one, and how to calculate moving averages that update automatically.

But first, spend a moment with this idea: a window function doesn't change your data. It doesn't filter it or group it. It just gives each row a little more information about its neighbors.

That simple shift in thinking - from reducing data to enriching it - is where the real power begins.


*This is part one of a series on window functions. In part two, we'll write actual queries that solve real problems.*