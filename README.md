# **E-Commerce Sales and Operation Analysis**

***
## **Project Goal**
To Provide data-driven and recommendation that optimize sales, Delivery, Customer Experience, and Product Strategy for a fast growing e-commerce company.

***
### Total Used
_SQL_
***
## Data Source
_Kaggle_

## **DATA EXPLORATION**
- Number of customers, orders, products, cities covered.
``` sql
-- Using distinct is to get total unique customer we have
SELECT count(distinct Customer_id) AS total_customers
FROM customers; -- Total Customer is 1,000

-- Total order is 1,000
SELECT count(distinct order_id) AS total_Orders
FROM orders;

-- Total Product is 100
SELECT count(distinct name) AS total_Product
FROM products;

-- Total Cities is 977
SELECT count(distinct city) AS total_City
FROM customers
```
***
- Common Order Status
``` sql
/*
Delivered
Returned
Cancelled */ 
SELECT distinct Order_Status AS Status
FROM orders;
```
***
## Core Analysis

- Top-selling products by total quantity sold and revenue.
``` sql
-- This query is helping to identify the products that generated the most revenue and sales volume
-- Join is to combine columns coming from the product and orderitem tables
-- the round function is to make the revenue return in 2 decimal place
SELECT p.name AS Product_Name,
SUM(oi.quantity) AS total_quantity,
ROUND(SUM(oi.quantity * p.unit_price),2) AS Revenue
FROM order_items oi
JOIN products p ON p.product_id = oi.product_id
GROUP BY p.name 
ORDER BY Revenue DESC;
```

- Customer lifetime value (CLV): Total amount spent per customer.
``` sql
-- This query will help identify our top 10 customer on the platform and the last time the used our services
-- The order_date column is not a date datetype so convert was used to convert it to date
SELECT
Convert(max(order_date),DATE) AS last_order,
c.full_name,
count(o.order_id) AS total_orders,
round(sum(o.total_amount_y),2) AS amount_Spent
FROM customers c
Join orders o ON c.customer_id = o.customer_id
WHERE order_status = 'Delivered'
GROUP BY c.full_name 
ORDER BY SUM(o.total_amount_y) DESC
LIMIT 10;
```

- Monthly sales trend across all products and regions.
``` sql
-- This query shows us the total sales trend MoM
-- Date_formate exracting the year and month from the order_date
SELECT 
date_format(order_date,'%Y %M') AS Month,
Round(Sum(total_amount_y),2) AS total_sales
FROM orders
WHERE order_status = 'Delivered'
GROUP BY month
ORDER BY total_sales DESC;
```
- Delivery delays: Count and average by courier and region.
```sql
-- Avg Delay per Courier
-- Good thing thing is not up to a day delay, and DPD had the highest delay
SELECT 
d.courier,
count(d.delivery_id) as total_deliveries,
round(avg(delivery_delay_days),2) AS avg_delay_days
FROM deliveries d
GROUP BY d.courier;
```
- Percentage of orders with discount codes vs without.
``` sql
-- Compare orders that used discount code vs those that didnt
-- It shows that discount impacted average order valiue with less order made
SELECT
   CASE
   WHEN discount_code IS NULL OR discount_code = '' OR discount_code = '    ' THEN 'No Discount'
   ELSE 'Discount' 
   END AS Discount_Status,
   COUNT(order_id) AS total_orders,
   ROUND(AVG(total_amount_y), 2) AS avg_order_value
FROM orders
WHERE order_status = 'Delivered'
GROUP BY Discount_Status;
```

- Orders by city and postcode — regional demand analysis

```sql
-- is suming our total revenue by city based on delivered orders.
-- This is helping identify hight-value location for logistics
SELECT 
c.city,
round(sum(o.total_amount_y),2) AS Revenue,
Round(count(o.order_id),2) AS total_Orders
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE order_status = 'Delivered'
GROUP BY c.city
order by  total_Orders desc; 
```

- Cancellation and return rates: % of total orders.

``` sql
-- showing the percentage of return and canclled under
-- cancelled and return order are in 0.097 and 0.079 respectively which is relatively low and thats what we want on our platform.
SELECT 
count(order_id) AS totalOrder,
Sum(case when order_status = 'Returned' THEN 1 ELSE 0 END) AS returned_orders,
sum(Case when order_status = 'Cancelled' THEN 1 ELSE 0 END) AS cancelled_orders,
round(Sum(case when order_status = 'Returned' THEN 1 ELSE 0 END) / count(order_id),3) AS returned_orders_percentage,
round(sum(Case when order_status = 'Cancelled' THEN 1 ELSE 0 END)/ count(order_id),3) AS cancelled_orders_percentage
FROM orders;
```


``` sql
-- this is breaking down the product that is been returned the most and the total amount we had to refund
-- its gives insight on why our customers are returning our product.
SELECT 
p.name,
r.return_reason,
count(r.return_id) As Total_Return,
round(Sum(r.refund_amount),2) AS total_refund
FROM returns r
JOIN order_items oi ON oi.order_id = r.order_id
JOIN products p ON p.product_id = oi.product_id
GROUP BY p.name,r.return_reason
order by total_refund desc;
```
