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
#### Answer:
| customer_id  | amount_spent |
| -------------| ------------ | 
|      A       |      76      | 
|      B       |      74      | 
|      C       |      36      | 

- Customer A spent $76.
- Customer B spent $74.
- Customer C spent $36.

#### 2. How many days has each customer visited the restaurant?  
````sql
SELECT
  customer_id,
  count(distinct(order_date)) as total_visits
FROM
  dannys_diner.sales
GROUP BY
  customer_id
ORDER BY
  customer_id;
````
#### Answer:
| customer_id  | total_visits |
| -------------| ------------ | 
|      A       |      4       | 
|      B       |      6       | 
|      C       |      2       | 

- Customer A visited the restaurant 4 times.
- Customer B visited the restaurant 6 times.
- Customer C visited the restaurant 2 times.




