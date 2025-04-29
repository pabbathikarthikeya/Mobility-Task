# Mobility Task

To effectively address the problem statement for the Data Analyst Intern assignment at Alt Mobility, we can break down the SQL-based analysis task into four sections, using the provided datasets:

payments.csv (Payment Data)

customer_orders.csv (Order Data)

You import the CSV files into a relational database like PostgreSQL, MySQL, or SQLite. Then, you write SQL queries directly on those tables.

Import the CSVs:

Use COPY (PostgreSQL) or LOAD DATA (MySQL) or a GUI (like DBeaver or pgAdmin).

#### Advantages:

Fast and scalable for large data.

Powerful for multi-table JOINs and aggregations.

Easy to visualize results using tools like Power BI or Tableau.

## Create Tables in PostgreSQL

```bash
-- Create table for customer orders
CREATE TABLE customer_orders (
    order_id VARCHAR PRIMARY KEY,
    customer_id VARCHAR,
    order_date DATE,
    order_amount NUMERIC,
    shipping_address TEXT,
    order_status VARCHAR
);

-- Create table for payments
CREATE TABLE payments (
    payment_id VARCHAR PRIMARY KEY,
    order_id VARCHAR,
    payment_date DATE,
    payment_amount NUMERIC,
    payment_method VARCHAR,
    payment_status VARCHAR,
    FOREIGN KEY (order_id) REFERENCES customer_orders(order_id)
);

```

## Import CSV Data into PostgreSQL

```
-- Import customer orders
COPY customer_orders(order_id, customer_id, order_date, order_amount, shipping_address, order_status)
FROM '/absolute/path/to/customer_orders.csv'
DELIMITER ','
CSV HEADER;

-- Import payments
COPY payments(payment_id, order_id, payment_date, payment_amount, payment_method, payment_status)
FROM '/absolute/path/to/payments.csv'
DELIMITER ','
CSV HEADER;
```

## 1) a. Order and Sales Analysis

```
SELECT 
  order_status, 
  COUNT(*) AS total_orders
FROM customer_orders
GROUP BY order_status;
```
This SQL query is used to analyze how customer orders are distributed across different order statuses in the customer_orders table. By selecting the order_status column and applying the COUNT(*) function, the query calculates the total number of orders for each unique status, such as 'Delivered', 'Pending', or 'Cancelled'. The GROUP BY clause ensures that the count is grouped according to each order status.

## b.Monthly revenue trend (only delivered orders)

```
SELECT 
  DATE_TRUNC('month', order_date) AS month, 
  SUM(order_amount) AS monthly_revenue
FROM customer_orders
WHERE order_status = 'Delivered'
GROUP BY month
ORDER BY month;
```

### c. Average order value

```
SELECT 
  AVG(order_amount) AS avg_order_value
FROM customer_orders
WHERE order_status = 'Delivered';
```
## 2.Customer Analysis
### a. Order frequency per customer
```
SELECT 
  customer_id, 
  COUNT(*) AS total_orders
FROM customer_orders
GROUP BY customer_id
ORDER BY total_orders DESC;
```

### b. Number of repeat customers
```
SELECT 
  COUNT(*) AS repeat_customers
FROM (
  SELECT customer_id
  FROM customer_orders
  GROUP BY customer_id
  HAVING COUNT(*) > 1
) AS sub;
```
### c. Customer activity over time (monthly)

```
SELECT 
  DATE_TRUNC('month', order_date) AS month, 
  COUNT(DISTINCT customer_id) AS active_customers
FROM customer_orders
GROUP BY month
ORDER BY month;
```
## 3. Payment Status Analysis

### a. Count by payment status

```
SELECT 
  payment_status, 
  COUNT(*) AS payment_count
FROM payments
GROUP BY payment_status;
```
### b. Failed payments trend

```
SELECT 
  DATE_TRUNC('month', payment_date) AS month, 
  COUNT(*) AS failed_payments
FROM payments
WHERE payment_status = 'Failed'
GROUP BY month
ORDER BY month;
```
### c. Failed payments by customer
```
SELECT 
  co.customer_id, 
  COUNT(*) AS failed_payments
FROM payments p
JOIN customer_orders co ON p.order_id = co.order_id
WHERE p.payment_status = 'Failed'
GROUP BY co.customer_id
ORDER BY failed_payments DESC;
```

## 4. Order Details Report
### Full order-payment overview

```
SELECT 
  co.order_id,
  co.customer_id,
  co.order_date,
  co.order_status,
  co.order_amount,
  co.shipping_address,
  p.payment_id,
  p.payment_date,
  p.payment_amount,
  p.payment_method,
  p.payment_status
FROM customer_orders co
LEFT JOIN payments p ON co.order_id = p.order_id
ORDER BY co.order_date DESC;
```














