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
SELECT product_id,count(distinct customer_id) "No of Custs" from sales
group by 1
having count(distinct customer_id) = (select count(distinct customer_id) from sales);
```

1. What is the most purchased item on the menu and **how many times was it purchased by all customers**?

```sql
SELECT product_id,customer_id,count(product_id) from sales group by 1,2
order by 1,2;
```