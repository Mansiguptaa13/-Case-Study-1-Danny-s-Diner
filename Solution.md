# 🍜 Case Study #1: Danny's Diner

## Solution

#### 1. What is the total amount each customer spent at the restaurant?  
````sql
SELECT
  s.customer_id,
  sum(m.price) as amount_spent
FROM
  dannys_diner.sales s
  INNER JOIN dannys_diner.menu m ON s.product_id = m.product_id
GROUP BY
  s.customer_id
ORDER BY
  s.customer_id;
````


