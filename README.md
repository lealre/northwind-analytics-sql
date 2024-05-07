# Analytic Report with SQL - Northwind database

This project aims to generate an analytical business report from a sales database. 

The Northwind database contains sales data from a company called Northwind Traders, which imports and exports specialty foods from around the world. In this report, we will primarily focus on extracting insights from revenue, product, and customer data using SQL operations in a PostgreSQL database.

The analyses provided here can benefit companies of all sizes looking to enhance their analytical capabilities. Through these reports, organizations can strategically position themselves in the market, leveraging data-driven decisions to improve their future results.

## Table of Contents
- [Questions we want to answer](#questions-we-want-to-answer)
  - [1. What were our total revenues in 1997?](#1-what-were-our-total-revenues-in-1997)
  - [2. Perform a monthly growth analysis and calculate YTD.](#2-perform-a-monthly-growth-analysis-and-calculate-ytd)
  - [3. What is the total amount each customer has paid so far?](#3-what-is-the-total-amount-each-customer-has-paid-so-far)
  - [4. Separate the customers into 5 groups according to the payment value per customer.](#4-separate-the-customers-into-5-groups-according-to-the-payment-value-per-customer)
  - [5. Now only the customers who are in groups 3, 4, and 5 will be selected for a special marketing analysis with them.](#5-now-only-the-customers-who-are-in-groups-3-4-and-5-will-be-selected-for-a-special-marketing-analysis-with-them)
  - [6. Identify the top 10 best-selling products.](#6-identify-the-top-10-best-selling-products)
  - [7. Filter for only UK customers who paid more than 1000 dollars.](#7-filter-for-only-uk-customers-who-paid-more-than-1000-dollars)
- [Context](#context)
- [How to run this project](#how-to-run-this-project)

-------------------------------

## Questions we want to answer

#### 1. What were our total revenues in 1997?

```sql
SELECT 
    SUM((od.unit_price * od.quantity) * (1 - od.discount)) as total_price
FROM order_details AS od
LEFT JOIN orders AS o ON od.order_id = o.order_id
WHERE EXTRACT(YEAR FROM o.order_date) = 1997;
```

| Total Revenues in 1997 |
|------------------------|
|   617,085.2023927002  |

#### 2. Perform a monthly growth analysis and calculate YTD.

```sql
WITH monthly_revenue_table AS (
    SELECT
        EXTRACT(YEAR FROM o.order_date) AS year,
        EXTRACT(MONTH FROM o.order_date) AS month,
        SUM(od.unit_price * od.quantity * (1.0 - od.discount)) AS monthly_revenue
    FROM order_details AS od
    LEFT JOIN orders AS o ON od.order_id = o.order_id
    GROUP BY
        EXTRACT(YEAR FROM o.order_date),
        EXTRACT(MONTH FROM o.order_date)
),
cumulative_revenue_table AS (
    SELECT
        year,
        month,
        monthly_revenue,
        SUM(monthly_revenue) OVER (PARTITION BY year ORDER BY month) AS ytd_revenue
    FROM monthly_revenue_table
)
SELECT
    year,
    month,
    monthly_revenue,
    monthly_revenue - LAG(monthly_revenue) OVER (PARTITION BY year ORDER BY month) AS monthly_difference,
    ytd_revenue,
    (monthly_revenue - LAG(monthly_revenue) OVER (PARTITION BY year ORDER BY month)) / LAG(monthly_revenue) OVER (PARTITION BY year ORDER BY month) * 100 AS percentage_monthly_difference
FROM cumulative_revenue_table
ORDER BY year, month;
```

| Year | Month | Cumulative Revenue | Monthly Revenue Change | Cumulative Revenue Change (%) | Monthly Revenue Change (%) |
|------|-------|--------------------|------------------------|-------------------------------|----------------------------|
| 1996 | 7     | 27861.89512966156 |                        |                               |                            |
| 1996 | 8     | 25485.275070743264 | -2376.6200589182954 | 53347.17020040483 | -8.530001451294545 |
| 1996 | 9     | 26381.400132587554 | 896.12506184429 | 79728.57033299239 | 3.51624637896504 |
| 1996 | 10    | 37515.72494547888 | 11134.32481289133 | 117244.29527847127 | 42.20520805162909 |
| ...| ...| ... |...|...|...|
| 1998 | 3     | 104854.15500015698 | 5438.86761714879 | 298491.552590101 | 5.47085640480519 |
| 1998 | 4     | 123798.6822555472 | 18944.527255390218 | 422290.2348456482 | 18.06750267107856 |
| 1998 | 5     | 18333.630432192596 | -105465.0518233546 | 440623.8652778408 | -85.1907709370056 |

YTD Analysis
```sql

WITH monthly_revenue_table AS (
    SELECT
        EXTRACT(YEAR FROM o.order_date) AS year,
        EXTRACT(MONTH FROM o.order_date) AS month,
        SUM(od.unit_price * od.quantity * (1.0 - od.discount)) AS monthly_revenue
    FROM order_details AS od
    LEFT JOIN orders AS o ON od.order_id = o.order_id
    GROUP BY
        EXTRACT(YEAR FROM o.order_date),
        EXTRACT(MONTH FROM o.order_date)
)
SELECT
    year,
    month,
    monthly_revenue,
    SUM(monthly_revenue) OVER (PARTITION BY year ORDER BY month) AS ytd_revenue
FROM monthly_revenue_table
```



#### 3. What is the total amount each customer has paid so far?

```sql
SELECT
    c.company_name,
    SUM((od.unit_price * od.quantity) * (1 - od.discount)) AS total_revenue
FROM order_details AS od
LEFT JOIN orders AS o ON od.order_id = o.order_id
LEFT JOIN customers AS c ON c.customer_id = o.customer_id
GROUP BY 1
ORDER BY total_revenue DESC;
```

| Customer                             | Total Payment Value    |
|--------------------------------------|-------------------------|
| QUICK-Stop                           | 110277.30503039382      |
| Ernst Handel                         | 104874.97814367746      |
| Save-a-lot Markets                   | 104361.94954039395      |
| Rattlesnake Canyon Grocery           | 51097.80082826822       |
| ... | ... |

#### 4. Separate the customers into 5 groups according to the payment value per customer.

```sql
SELECT
    c.company_name,
    SUM((od.unit_price * od.quantity) * (1 - od.discount)) AS total_revenue,
    NTILE(5) OVER (ORDER BY SUM((od.unit_price * od.quantity) * (1 - od.discount)) DESC) AS revenue_group
FROM order_details AS od
LEFT JOIN orders AS o ON od.order_id = o.order_id
LEFT JOIN customers AS c ON c.customer_id = o.customer_id
GROUP BY 1
ORDER BY total_revenue DESC;
```

| Customer                           | Total Payment Value | Group |
|------------------------------------|---------------------|-------|
| QUICK-Stop                         | 110277.30503039382 | 1     |
| Ernst Handel                       | 104874.97814367746 | 1     |
| ...       | ...  | ...  |
| Lazy K Kountry Store               | 356.99999809265137 | 5     |
| Centro comercial Moctezuma         | 100.79999923706055 | 5     |

#### 5. Now only the customers who are in groups 3, 4, and 5 will be selected for a special marketing analysis with them.

```sql
WITH companies_revenue_groups AS (
	SELECT
            c.company_name,
            SUM((od.unit_price * od.quantity) * (1 - od.discount)) AS total_revenue,
            NTILE(5) OVER (ORDER BY SUM((od.unit_price * od.quantity) * (1 - od.discount)) DESC) AS revenue_group
	FROM order_details AS od
	LEFT JOIN orders AS o ON od.order_id = o.order_id
	LEFT JOIN customers AS c ON c.customer_id = o.customer_id
	GROUP BY 1
	ORDER BY total_revenue DESC
)

SELECT 
    *   
FROM companies_revenue_groups
WHERE revenue_group IN (3,4,5);
```

| Cliente                              | Total Payment Value   | Group |
|--------------------------------------|-----------------------|-------|
| Split Rail Beer & Ale                | 11441.630072270782    | 3     |
| Tortuga Restaurante                  | 10812.150033950806    | 3     |
| ...                     | ...     | ...     |
| Lazy K Kountry Store                 | 356.99999809265137    | 5     |
| Centro comercial Moctezuma           | 100.79999923706055    | 5     |

#### 6. Identify the top 10 best-selling products.

```sql
SELECT 
    products.product_name, 
    SUM(order_details.unit_price * order_details.quantity * (1.0 - order_details.discount)) AS sales
FROM products
INNER JOIN order_details ON order_details.product_id = products.product_id
GROUP BY products.product_name
ORDER BY sales DESC;
```

| Product                           | Total Sales         |
|----------------------------------|---------------------|
| Côte de Blaye                    | 141396.7356273254  |
| Thüringer Rostbratwurst          | 80368.6724385033   |
| Raclette Courdavault             | 71155.69990943     |
| Tarte au sucre                   | 47234.969978504174 |
| Camembert Pierrot                | 46825.48029542655  |
| Gnocchi di nonna Alice           | 42593.0598222503   |
| Manjimup Dried Apples            | 41819.65024582073  |
| Alice Mutton                     | 32698.380216373203 |
| Carnarvon Tigers                 | 29171.874963399023 |
| Rössle Sauerkraut                | 25696.63978933155  |

#### 7. Filter for only UK customers who paid more than 1000 dollars.

```sql

```

## Context

## How to run this project
