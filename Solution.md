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

- Customer A purchased curry and sushi for the first time.
- Customer B purchased curry for the first time.
- Customer C purchased ramen for the first time.

#### 4.What is the most purchased item on the menu and how many times was it purchased by all customers??  
````sql
SELECT
  m.product_name,
  count(m.product_name) as "Number of Items Ordered"
FROM
  dannys_diner.menu m
  join dannys_diner.sales s on m.product_id = s.product_id
GROUP BY
  m.product_name
ORDER by
  count(m.product_name) desc
LIMIT
  1;
````
#### Answer:

| product_name | Number of Items Ordered |
| ------------ | ----------------------- |
| ramen        | 8                       |

- The most purchased item on the menu is ramen which is purchased 8 times.
 Tasty...!!

#### 5. Which item was the most popular for each customer?
````sql
    WITH
      preference_table AS (
        SELECT
          s.customer_id,
          m.product_name,
          count(m.product_name),
          DENSE_RANK() OVER(
            PARTITION BY s.customer_id
            ORDER BY
              count(m.product_name) DESC
          ) AS Most_preferred
        FROM
          dannys_diner.menu m
          JOIN dannys_diner.sales s ON m.product_id = s.product_id
        GROUP BY
          s.customer_id,
          m.product_name
        ORDER BY 
          s.customer_id asc,
          count(m.product_name) desc
      )
    SELECT
      customer_id,
      product_name
    FROM
      preference_table
    WHERE
      Most_preferred = 1
    ORDER BY
      customer_id;
````
#### Answer:

| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | ramen        |
| B           | curry        |
| B           | sushi        |
| C           | ramen        |

- Customer A's favourite item is ramen.
- Customer B's favourite items are ramen, curry and sushi everything avaliable in the menu.
  
#### 6. Which item was purchased first by the customer after they became a member?  
```` sql
    WITH
      first_item AS(
        SELECT
          distinct(m.product_name),
          s.customer_id,
          s.order_date,
          mm.join_date,
          DENSE_RANK() OVER(
            PARTITION BY s.customer_id
            ORDER BY
              s.order_date
          ) AS rank
        FROM
          dannys_diner.menu m
          JOIN dannys_diner.sales s on m.product_id = s.product_id
          JOIN dannys_diner.members mm on s.customer_id = mm.customer_id
        WHERE
          mm.join_date < s.order_date
        ORDER BY
          s.customer_id,
          s.order_date
      )
    SELECT
      customer_id,
      order_date,
      product_name
    FROM
      first_item
    WHERE
      rank = 1
    ORDER BY
      customer_id;
````
#### Answer:

| customer_id | order_date               | product_name |
| ----------- | ------------------------ | ------------ |
| A           | 2021-01-07 | curry        |
| B           | 2021-01-11 | sushi       |

- Customer A's first ordered "curry" after joining membership.
- Customer B's first ordered "sushi" after joining membership.







