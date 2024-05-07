# Analytic Report with SQL - Northwind database

This project has the objective ....

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

#### 7. UK Customers Who Paid More Than $1000

```sql

```

## Context

## How to run this project
