# Analytics Report with SQL - Northwind database

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
