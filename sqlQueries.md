# SQL Query Documentation

## Overview
This document provides a collection of commonly used SQL queries for various use cases, such as retrieving the latest records, managing variations, handling order and invoice mappings, and cleaning up old logs. These queries are designed for general and dynamic use in any relational database.

---

## Queries and Use Cases

### 1. Fetching the Most Recently Modified Records
```sql
SELECT *
FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY entity_id ORDER BY last_modified DESC) AS rk
    FROM entity_table
    WHERE last_modified > '2025-01-28T00:00:00.000Z' 
) AS ranked
WHERE rk = 1
LIMIT 10;
```
**Use Case:**
- Retrieves the most recently modified record for each unique entity.
- Useful for tracking updates and ensuring the latest data is available.

**Optimization Tips:**
- Ensure `last_modified` is indexed for performance improvements.
- Use dynamic date parameters instead of hardcoded values.

---

### 2. Fetching Latest Variation of an Entity
```sql
SELECT * 
FROM (
    SELECT *, 
           ROW_NUMBER() OVER (PARTITION BY entity_id ORDER BY variation_id DESC) AS rk
    FROM entity_variations
    WHERE entity_reference = '12345'
) AS ranked
WHERE rk = 1
ORDER BY variation_id DESC
LIMIT 100;
```
**Use Case:**
- Retrieves the latest variation of an entity.
- Useful in scenarios where multiple versions exist and only the latest one is required.

**Optimization Tips:**
- Ensure `variation_id` is indexed and unique.
- Replace the hardcoded entity reference with a parameterized input.

---

### 3. Mapping Orders to Invoices
```sql
SELECT orders.order_id, orders.customer_id, invoices.invoice_id, invoices.total_amount
FROM orders
JOIN invoices ON orders.order_id = invoices.order_id
LIMIT 100;
```
**Use Case:**
- Links order records with corresponding invoices for tracking payments.
- Useful in e-commerce and financial applications.

**Optimization Tips:**
- Ensure `order_id` is indexed in both tables.
- Consider additional filtering to narrow down results.

---

### 4. Fetching Latest Transactions for Each Customer
```sql
SELECT *
FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY transaction_date DESC) AS rk
    FROM transactions
    WHERE transaction_date >= CURRENT_DATE - INTERVAL '30 DAYS'
) AS ranked
WHERE rk = 1
LIMIT 10;
```
**Use Case:**
- Retrieves the most recent transaction per customer.
- Useful for financial reports and user activity tracking.

**Optimization Tips:**
- Index `transaction_date` for faster queries.
- Use dynamic date ranges instead of hardcoded values.

---

### 5. Deleting Old Logs
```sql
DELETE FROM logs WHERE created_date < NOW() - INTERVAL '90 DAYS';
```
**Use Case:**
- Cleans up old logs to optimize database storage.
- Helps maintain database performance and reduces unnecessary data storage.

**Optimization Tips:**
- Schedule this query to run periodically using a cron job or database scheduler.
- Use partitioning for large log tables to improve deletion efficiency.

---

### 6. Fetching Orders in Specific Stages
```sql
SELECT ranked.order_id, ranked.last_modified, order_status.stage_code
FROM (
    SELECT order_id, last_modified, 
           ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY last_modified DESC) AS rk 
    FROM orders
) AS ranked
INNER JOIN order_status ON ranked.order_id = order_status.order_id
WHERE ranked.rk = 1
AND order_status.stage_code IN ('Processing', 'Shipped') 
ORDER BY ranked.last_modified DESC  
LIMIT 10;
```
**Use Case:**
- Retrieves the latest order statuses for tracking.
- Useful in order management systems for monitoring progress.

**Optimization Tips:**
- Ensure `last_modified` and `stage_code` are indexed.
- Consider additional filters to narrow down results.

---

### 7. Fetching Top N Customers Based on Purchase Amount
```sql
SELECT customer_id, SUM(total_amount) AS total_spent
FROM orders
GROUP BY customer_id
ORDER BY total_spent DESC
LIMIT 10;
```
**Use Case:**
- Identifies top customers based on their total spending.
- Useful for loyalty programs and targeted marketing.

**Optimization Tips:**
- Ensure `total_amount` is indexed and properly stored as a numeric type.
- Use additional filters such as date ranges if needed.

---

### 8. Fetching Orders with No Invoices
```sql
SELECT orders.order_id, orders.customer_id
FROM orders
LEFT JOIN invoices ON orders.order_id = invoices.order_id
WHERE invoices.order_id IS NULL;
```
**Use Case:**
- Identifies orders that have not yet been invoiced.
- Useful for accounting and order tracking.

**Optimization Tips:**
- Index `order_id` in both tables to improve query performance.
- Consider filtering by order date to limit large datasets.

---

## Conclusion
This document provides a reference for essential SQL queries for general database operations. For optimal performance, ensure indexes are properly defined, queries are parameterized, and regular maintenance tasks are scheduled.

