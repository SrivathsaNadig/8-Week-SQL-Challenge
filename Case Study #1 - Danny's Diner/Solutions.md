# Case Study #1 - Danny's Diner

# Case Study Questions

1. What is the total amount each customer spent at the restaurant?

```sql
SELECT customer_id,SUM(price) as "Total Spent"
FROM dannys_diner.menu M JOIN
dannys_diner.sales S ON S.product_id=M.product_id
group by customer_id;
```

1. How many days has each customer visited the restaurant?

```sql
SELECT customer_id,count(distinct order_date) "No of Visits" from sales group by 1;
```

1. What was the first item from the menu purchased by each customer?

```sql
SELECT * FROM(
SELECT 
	CUSTOMER_ID,
	ORDER_DATE,
	PRODUCT_NAME,
	RANK() OVER(PARTITION BY CUSTOMER_ID ORDER BY ORDER_DATE ASC) AS RNK
FROM dannys_diner.SALES S 
JOIN dannys_diner.MENU M ON S.product_id=M.product_id)
WHERE RNK=1;

```

1. What is the most purchased item on the menu and ****how many times was it purchased by all customers?

```sql
SELECT --TOP 1
	PRODUCT_NAME,
	count(ORDER_DATE) 'orders',
FROM dannys_diner.SALES S 
JOIN dannys_diner.MENU M ON S.product_id=M.product_id)
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1;
```

1. Which item was the most popular for each customer?

```sql
WITH CTE AS(SELECT 
	PRODUCT_NAME,
	CUSTOMER_ID,
	count(ORDER_DATE) 'orders',
	RANK() OVER(PARTITION BY CUSTOMER_ID ORDER BY count(ORDER_DATE) DSC) AS RNK
FROM dannys_diner.SALES S 
JOIN dannys_diner.MENU M ON S.product_id=M.product_id)
GROUP BY 1,2)

SELECT * FROM CTE WHERE RNK = 1;
```

1. Which item was purchased first by the customer after they became a member?

```sql
WITH CTE AS(
SELECT
	M1.customer_id,
    product_name,
    JOIN_DATE,
    ORDER_DATE,
	RANK() OVER(PARTITION BY M1.CUSTOMER_ID ORDER BY ORDER_DATE ASC) RNK
FROM dannys_diner.SALES S 
INNER JOIN dannys_diner.MENU M ON S.product_id=M.product_id
INNER JOIN dannys_diner.MEMBERS M1 ON S.customer_id=M1.customer_id
WHERE ORDER_DATE>=JOIN_DATE)

SELECT
	customer_id,
    product_name
FROM 
    CTE
WHERE 
	RNK=1;

```

1. Which item was purchased first by the customer before they became a member?

```sql
WITH CTE AS(
SELECT
	M1.customer_id,
    product_name,
    JOIN_DATE,
    ORDER_DATE,
	RANK() OVER(PARTITION BY M1.CUSTOMER_ID ORDER BY ORDER_DATE desc) RNK
FROM dannys_diner.SALES S 
INNER JOIN dannys_diner.MENU M ON S.product_id=M.product_id
INNER JOIN dannys_diner.MEMBERS M1 ON S.customer_id=M1.customer_id
WHERE ORDER_DATE<JOIN_DATE)

SELECT
	customer_id,
    product_name
FROM 
    CTE
WHERE 
	RNK=1;

```

1. What is the total items and amount spent for each member before they became a member?

```sql
SELECT
	M1.customer_id,
  COUNT(product_name),
  SUM(PRICE)  
FROM dannys_diner.SALES S 
INNER JOIN dannys_diner.MENU M ON S.product_id=M.product_id
INNER JOIN dannys_diner.MEMBERS M1 ON S.customer_id=M1.customer_id
WHERE ORDER_DATE<JOIN_DATE
GROUP BY 1,2;

```

1. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql
with cte as(
SELECT
    m1.customer_id,
    m.product_name,
    m.price,
    CASE  
        WHEN m.product_name = 'sushi' THEN m.price * 2 * 10
        ELSE price*10
    END AS score
FROM dannys_diner.SALES s 
INNER JOIN dannys_diner.MENU m ON s.product_id = m.product_id
INNER JOIN dannys_diner.MEMBERS m1 ON s.customer_id = m1.customer_id)

select
customer_id,
SUM(SCORE) 'SCORE'
from cte
GROUP BY 1;

```

1. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```sql
SELECT 
  S.customer_id, 
  SUM(
    CASE 
      WHEN S.order_date BETWEEN MEM.join_date AND (MEM.join_date + INTERVAL '6 days') THEN M.price * 10 * 2 
      WHEN M.product_name = 'sushi' THEN M.price * 10 * 2 
      ELSE M.price * 10 
    END
  ) AS points 
FROM 
  MENU AS M 
  INNER JOIN SALES AS S ON S.product_id = M.product_id
  INNER JOIN MEMBERS AS MEM ON MEM.customer_id = S.customer_id 
WHERE 
  DATE_TRUNC('month', S.order_date) = '2021-01-01' 
GROUP BY 
  S.customer_id;

```

```sql
SELECT 
  S.customer_id, 
  SUM(
    CASE 
      WHEN S.order_date BETWEEN MEM.join_date AND (MEM.join_date + INTERVAL '6 days') THEN M.price * 10 * 2 
      WHEN M.product_name = 'sushi' THEN M.price * 10 * 2 
      ELSE M.price * 10 
    END
  ) AS points 
FROM 
  MENU AS M 
  INNER JOIN SALES AS S ON S.product_id = M.product_id
  INNER JOIN MEMBERS AS MEM ON MEM.customer_id = S.customer_id 
WHERE 
  DATE_TRUNC('month', S.order_date) = '2021-01-01' 
GROUP BY 
  S.customer_id;
```