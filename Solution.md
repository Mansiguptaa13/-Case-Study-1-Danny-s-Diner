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

#### 3. What was the first item from the menu purchased by each customer?  
````sql
WITH ordered_sales AS (
  SELECT 
    sales.customer_id, 
    sales.order_date, 
    menu.product_name,
    DENSE_RANK() OVER(
      PARTITION BY sales.customer_id 
      ORDER BY sales.order_date) AS rank
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
)

SELECT 
  customer_id, 
  product_name
FROM ordered_sales
WHERE rank = 1
GROUP BY customer_id, product_name;
````
#### Answer:

| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |





