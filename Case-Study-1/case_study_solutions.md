# Case Study 1: Danny's Diner

## Solution

***

### 1. What is the total amount each customer spent at the restaurant?

````sql
SELECT s.customer_id,
       Sum(m.price) AS total_spent_by_each_customer
FROM   sales s
       JOIN menu m
         ON s.product_id = m.product_id
GROUP  BY s.customer_id
ORDER  BY s.customer_id````

#### Answer:
| Customer_id | total_spent_by_each_customer |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

- Customer A, B and C spent $76, $74 and $36 respectivly.

***

### 2. How many days has each customer visited the restaurant?

````sql
SELECT s.customer_id,
       Count(DISTINCT s.order_date) AS visited_days
FROM   sales s
GROUP  BY s.customer_id
ORDER  BY s.customer_id ````

#### Answer:
| Customer_id | visited_days |
| ----------- | ----------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |

- Customer A, B and C visited 4, 6 and 2 times respectively.

***

### 3. What was the first item from the menu purchased by each customer?

````sql
SELECT customer_id, product_name
FROM   (SELECT s.customer_id,
               m.product_name,
               Row_number()
                 OVER(
                   partition BY s.customer_id
                   ORDER BY s.order_date) AS row_num
        FROM   sales s
               JOIN menu m
                 ON s.product_id = m.product_id) ranked_items
WHERE  ranked_items.row_num = 1
 ````

#### Answer:
| Customer_id | product_name | 
| ----------- | ----------- |
| A           | curry        | 
| A           | sushi        | 
| B           | curry        | 
| C           | ramen        |

- Customer A's first order is curry and sushi.
- Customer B's first order is curry.
- Customer C's first order is ramen.

***

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

````sql
SELECT m.Product_name,
       Count(*) AS order_count
FROM   sales s
       JOIN menu m
         ON m.product_id = s.product_id
GROUP  BY s.product_id,
          m.product_name
ORDER  BY order_count DESC
LIMIT  1 
````
#### Answer:
| Product_name  | order_count | 
| ----------- | ----------- |
| ramen       | 8|


- Most purchased item on the menu is ramen which is 8 times.

***

### 5. Which item was the most popular for each customer?

````sql
WITH order_count_table
     AS (SELECT sales.customer_id,
                menu.product_name,
                Count(*) AS order_count
         FROM   dannys_diner.sales
                JOIN dannys_diner.menu
                  ON sales.product_id = menu.product_id
         GROUP  BY customer_id,
                   product_name
         ORDER  BY customer_id,
                   order_count DESC),
     popular_rank_table
     AS (SELECT *,
                Rank()
                  OVER(
                    partition BY customer_id
                    ORDER BY order_count DESC) AS rank
         FROM   order_count_table)
SELECT *
FROM   popular_rank_table
WHERE  rank = 1; 
````

#### Answer:
| Customer_id | Product_name | order_count |
| ----------- | ---------- |------------  |
| A           | ramen        |  3   |
| B           | sushi        |  2   |
| B           | curry        |  2   |
| B           | ramen        |  2   |
| C           | ramen        |  3   |

- Customer A and C's favourite item is ramen while customer B savours all items on the menu. 

***

### 6. Which item was purchased first by the customer after they became a member?

````sql
SELECT ne.customer_id,
       ne.product_name,
	ne.order_date
FROM   (SELECT s.customer_id,
               s.order_date,
               me.product_name,
               Row_number()
                 OVER(
                   partition BY s.customer_id) AS row_num
        FROM   sales s
               JOIN members m
                 ON s.customer_id = m.customer_id
               JOIN menu me
                 ON s.product_id = me.product_id
        WHERE  s.order_date >= m.join_date) ne
WHERE  ne.row_num = 1 
````


#### Answer:
| customer_id |  product_name |order_date
| ----------- | ----------  |----------  |
| A           |  curry        |2021-01-07 |
| B           |  sushi        |2021-01-11 |

After becoming a member 
- Customer A's first order was curry.
- Customer B's first order was sushi.

***

### 7. Which item was purchased just before the customer became a member?

````sql
SELECT *
INTO   temp
FROM   (SELECT s.customer_id,
               s.order_date,
               Row_number()
                 OVER(
                   partition BY s.customer_id
                   ORDER BY s.order_date DESC)
        FROM   sales s
               JOIN members m
                 ON s.customer_id = m.customer_id
        WHERE  s.order_date < m.join_date) a;

SELECT t.customer_id,
       m.product_name,
       t.order_date
FROM   temp t
       JOIN menu m
         ON m.product_id = t.product_id
WHERE  row_number = 1
ORDER  BY t.customer_id; 
````

#### Answer:
| customer_id |product_name |order_date  |
| ----------- | ----------  |---------- |
| A           |  sushi      |2021-01-01 | 
| A           |  curry      |2021-01-01 | 
| B           |   sushi     |2021-01-04 |

Before becoming a member 
- Customer A’s last order was sushi and curry.
- Customer B’s last order wassushi.

***

### 8. What is the total items and amount spent for each member before they became a member?

````sql
SELECT S.customer_id,
       Count(S.product_id) AS Items,
       Sum(M.price)        AS total_sales
FROM   sales S
       JOIN menu M
         ON m.product_id = s.product_id
       JOIN members Mem
         ON Mem.customer_id = S.customer_id
WHERE  S.order_date < Mem.join_date
GROUP  BY S.customer_id; 
````


#### Answer:
| customer_id |Items | total_sales |
| ----------- | ---------- |----------  |
| A           | 2 |  25       |
| B           | 3 |  40       |

Before becoming a member
- Customer A spent $25 on 2 items.
- Customer B spent $40 on 3 items.

***

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?

````sql
WITH points
     AS (SELECT *,
                CASE
                  WHEN product_id = 1 THEN price * 20
                  ELSE price * 10
                END AS Points
         FROM   menu)
SELECT S.customer_id,
       Sum(P.points) AS Points
FROM   sales S
       JOIN points p
         ON p.product_id = S.product_id
GROUP  BY S.customer_id; 
````


#### Answer:
| customer_id | Points | 
| ----------- | -------|
| A           | 860 |
| B           | 940 |
| C           | 360 |

- Total points for customer A, B and C are 860, 940 and 360 respectivly.

***

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?

````sql
WITH points_table
     AS (SELECT s.customer_id,
                Sum(CASE
                      WHEN m.product_id = 1 THEN ( 20 * m.price )
                      WHEN s.order_date BETWEEN me.join_date AND
                                                me.join_date + 6
                    THEN (
                      20 *
                      m.price )
                      ELSE ( m.price * 10 )
                    END) AS points
         FROM   menu m
                JOIN sales s
                  ON m.product_id = s.product_id
                JOIN members me
                  ON s.customer_id = me.customer_id
         WHERE  s.order_date < '2021-01-31'
         GROUP  BY s.customer_id)
SELECT *
FROM   points_table 
````

#### Answer:
| Customer_id | Points | 
| ----------- | ---------- |
| A           | 1370 |
| B           | 820 |

- Total points for Customer A and B are 1,370 and 820 respectivly.

### Bonus Question

#### Part 1: Join All Things

SELECT s.customer_id,
       s.order_date,
       m.product_name,
       m.price,
       CASE
         WHEN s.order_date BETWEEN me.join_date AND ME.join_date + 6 THEN 'Y'
         ELSE 'N'
       END AS member
FROM   sales s
       JOIN menu m
         ON s.product_id = m.product_id
       LEFT JOIN members me
              ON s.customer_id = me.customer_id
ORDER  BY customer_id,
          s.order_date,
          m.product_name 


#### Part 2: Rank All Things

SELECT A.customer_id,
       A.order_date,
       A.product_name,
       A.member,
       CASE
         WHEN member = 'N' THEN NULL
         ELSE Rank ()
                OVER(
                  partition BY customer_id, member
                  ORDER BY order_date)
       END AS ranking
FROM   (SELECT s.customer_id,
               s.order_date,
               m.product_name,
               m.price,
               CASE
                 WHEN me.join_date > s.order_date THEN 'N'
                 WHEN me.join_date <= s.order_date THEN 'Y'
               END AS member
        FROM   sales s
               JOIN menu m
                 ON s.product_id = m.product_id
               LEFT JOIN members me
                      ON s.customer_id = me.customer_id
        ORDER  BY customer_id,
                  s.order_date,
                  m.product_name) A 

***


