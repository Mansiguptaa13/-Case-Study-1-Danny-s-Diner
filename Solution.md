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

#### 4. What is the most purchased item on the menu and how many times was it purchased by all customers??  
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

#### 7. Which item was purchased just before the customer became a member?  
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
              s.order_date desc
          ) as rank
        FROM
          dannys_diner.menu m
          JOIN dannys_diner.sales s ON m.product_id = s.product_id
          JOIN dannys_diner.members mm ON s.customer_id = mm.customer_id
        WHERE
          mm.join_date > s.order_date
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
| A           | 2021-01-01 | curry        |
| A           | 2021-01-01 | sushi        |
| B           | 2021-01-04| sushi        |

- Customer A's last order before becoming a member includes curry and sushi.
- Customer B's last order before becoming a member is sushi.

#### 8. What is the total items and amount spent for each member before they became a member?
```` sql
SELECT
  s.customer_id,
  count(s.product_id) AS total_item,
  sum(m.price) AS total_amount
FROM
  dannys_diner.menu m
  JOIN dannys_diner.sales s ON m.product_id = s.product_id
  JOIN dannys_diner.members mm ON s.customer_id = mm.customer_id
WHERE
  mm.join_date > s.order_date
GROUP BY
  s.customer_id
ORDER BY
  s.customer_id;
````

#### Answer: 
| customer_id | total_item | total_amount |
| ----------- | ---------- | ------------ |
| A           | 2          | 25           |
| B           | 3          | 40           |

- Customer A's purchased 2 itens and spent $25 before becoming a member.
- Customer B's purchased 3 itens and spent $40 before becoming a member.

#### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
````sql
    SELECT
      s.customer_id,
      sum(
        CASE
          WHEN product_name = 'sushi' THEN price * 20
          ELSE price * 10
        END
      ) Total_points
    FROM
      dannys_diner.sales s
      JOIN dannys_diner.menu m ON m.product_id = s.product_id
    GROUP BY
      s.customer_id
    ORDER BY
      s.customer_id;
````
#### Answer:

| customer_id | total_points |
| ----------- | ------------ |
| A           | 860          |
| B           | 940          |
| C           | 360          |

- Total points earned by Customer A is 860.
- Total points earned by Customer B is 940.
- Total points earned by Customer C is 360.

#### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
````sql
WITH dates_cte AS (
  SELECT 
    customer_id, 
    join_date, 
    join_date + 7 AS valid_date, 
    DATE_TRUNC( 'month', '2021-01-31'::DATE)  + interval '1 month'   - interval '1 day' AS last_date
  FROM dannys_diner.members
)

SELECT 
  sales.customer_id, 
  SUM(CASE
    WHEN menu.product_name = 'sushi' THEN 20 * menu.price
    WHEN sales.order_date BETWEEN dates.join_date-1 AND dates.valid_date THEN 20 * menu.price
    ELSE 10 * menu.price END) AS points_earned
FROM dannys_diner.sales
JOIN dates_cte AS dates
  ON sales.customer_id = dates.customer_id
  AND sales.order_date <= dates.last_date
JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id;
````
#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 1370 |
| B           | 940 |

- Total points earned by Customer A is 1370.
- Total points earned by Customer B is 940.
  

## Bonus Questions Solution
#### 1. The following questions are related creating basic data tables that Danny and his team can use to quickly derive insights without needing to join the underlying tables using SQL.
````sql
SELECT
  s.customer_id,
  s.order_date,
  mn.product_name,
  mn.price,
  CASE
    WHEN s.order_date >= mb.join_date THEN 'Y'
    ELSE 'N'
  END AS member
FROM
  dannys_diner.sales s
  LEFT JOIN dannys_diner.menu mn ON mn.product_id = s.product_id
  LEFT JOIN dannys_diner.members mb ON mb.customer_id = s.customer_id
ORDER BY
  s.customer_id,
  s.order_date;
````

#### Answer:

| customer_id | order_date               | product_name | price | member |
| ----------- | ------------------------ | ------------ | ----- | ------ |
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-10| ramen        | 12    | Y      |
| A           | 2021-01-11| ramen        | 12    | Y      |
| A           | 2021-01-11| ramen        | 12    | Y      |
| B           | 2021-01-01| curry        | 15    | N      |
| B           | 2021-01-02 | curry        | 15    | N      |
| B           | 2021-01-04 | sushi        | 10    | N      |
| B           | 2021-01-11 | sushi        | 10    | Y      |
| B           | 2021-01-16| ramen        | 12    | Y      |
| B           | 2021-02-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-07 | ramen        | 12    | N      |

#### 2. Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.
````sql
WITH
  classification_table AS (
    SELECT
      s.customer_id,
      s.order_date,
      mn.product_name,
      mn.price,
      mb.join_date,
      CASE
        WHEN s.order_date >= mb.join_date THEN 'Y'
        ELSE 'N'
      END AS member
    FROM
      dannys_diner.sales s
      LEFT JOIN dannys_diner.menu mn ON mn.product_id = s.product_id
      LEFT JOIN dannys_diner.members mb ON mb.customer_id = s.customer_id
    ORDER BY
      s.customer_id,
      s.order_date
  )
SELECT
  customer_id,
  order_date,
  product_name,
  price,
  member,
  CASE
    WHEN member = 'N' THEN null
    ELSE DENSE_RANK() OVER(
      PARTITION BY customer_id,
      member
      ORDER BY
        order_date
    )
  END AS ranking
FROM
  classification_table;
````

#### Answer: 

| customer_id | order_date               | product_name | price | member | ranking |
| ----------- | ------------------------ | ------------ | ----- | ------ | ------- |
| A           | 2021-01-01 | sushi        | 10    | N      | null     |
| A           | 2021-01-01| curry        | 15    | N      | null     |
| A           | 2021-01-07 | curry        | 15    | Y      | 1       |
| A           | 2021-01-10 | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| B           | 2021-01-01 | curry        | 15    | N      |null     |
| B           | 2021-01-02 | curry        | 15    | N      |null     |
| B           | 2021-01-04 | sushi        | 10    | N      |null     |
| B           | 2021-01-11 | sushi        | 10    | Y      | 1       |
| B           | 2021-01-16 | ramen        | 12    | Y      | 2       |
| B           | 2021-02-01| ramen        | 12    | Y      | 3       |
| C           | 2021-01-01 | ramen        | 12    | N      | null    |
| C           | 2021-01-01 | ramen        | 12    | N      |  nul    |
| C           | 2021-01-07 | ramen        | 12    | N      | null    |
