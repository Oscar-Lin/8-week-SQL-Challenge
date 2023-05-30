
## Case Study Questions

1. What is the total amount each customer spent at the restaurant?
2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
10. What is the total items and amount spent for each member before they became a member?
11. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
12. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
***

###  1. What is the total amount each customer spent at the restaurant?
```sql
select customer_id, sum(price) as "total amount"
from sales s left join menu m
on s.product_id = m.product_id
group by customer_id;
``` 
#### Result set:
| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76         |
| B           | 74         |
| C           | 36         |

***

###  2. How many days has each customer visited the restaurant?
```sql
select customer_id, sum(price) as "total amount"
from sales s left join menu m
on s.product_id = m.product_id
group by customer_id;
```
#### Result set:
| customer_id | visit_count |
| ----------- | ----------- |
| A           | 4           |
| B           | 6           |
| C           | 2           |

***

###  3. What was the first item from the menu purchased by each customer?
```sql
SELECT s.customer_id,m.product_name
FROM (
  SELECT customer_id, MIN(order_date) AS first_order_date
  FROM sales
  GROUP BY customer_id
) AS f
JOIN sales AS s ON s.customer_id = f.customer_id AND s.order_date = f.first_order_date
JOIN menu AS m ON s.product_id = m.product_id
```
#### Result set:
| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |
| C           | ramen        |

***

###  4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
select top 1 product_name,
count(*) as "number_ordered"
from sales s left join menu m
on s.product_id = m.product_id
group by product_name
order by number_ordered desc ;
```
#### Result set:
| most_purchased_item | order_count |
| ------------------- | ----------- |
| ramen               | 8           |

***

###  5. Which item was the most popular for each customer?
```sql
select customer_id, product_name, count(*) as purchase_count
from sales s 
left join menu m 
on s.product_id = m.product_id
group by s.customer_id,product_name
having COUNT(*) = (
  SELECT TOP 1 COUNT(*)
  FROM sales
  WHERE customer_id = s.customer_id
  GROUP BY product_id
  ORDER BY COUNT(*) DESC
)
order by customer_id;
```
#### Result set:
| customer_id | product_name | order_count |
| ----------- | ------------ | ----------- |
| A           | ramen        | 3           |
| B           | ramen        | 2           |
| B           | curry        | 2           |
| B           | sushi        | 2           |
| C           | ramen        | 3           |

***

###  6. Which item was purchased first by the customer after they became a member?
```sql
SELECT s.customer_id, m.product_name, s.order_date
FROM sales AS s
JOIN menu AS m ON s.product_id = m.product_id
JOIN members AS mem ON s.customer_id = mem.customer_id AND s.order_date >= mem.join_date
WHERE NOT EXISTS (
  SELECT 1
  FROM sales AS s2
  WHERE s2.customer_id = s.customer_id
    AND s2.order_date >= mem.join_date
    AND s2.order_date < s.order_date
)
ORDER BY s.customer_id, s.order_date
```
#### Result set:
| customer_id | product_name | order_date               |
| ----------- | ------------ | ------------------------ |
| A           | curry        | 2021-01-07|
| B           | sushi        | 2021-01-11|

***
###  7. Which item was purchased just before the customer became a member?
```sql
SELECT s.customer_id, m.product_name, s.order_date,mem.join_date
FROM sales  s
JOIN menu  m on s.product_id = m.product_id
JOIN members AS mem ON s.customer_id = mem.customer_id AND s.order_date = (
  SELECT MAX(order_date)
  FROM sales
  WHERE customer_id = mem.customer_id AND order_date < mem.join_date
)
ORDER BY s.customer_id, s.order_date;
```
#### Result set:
| customer_id | product_name | order_date  | join_date     |
| ----------- | ------------ | ----------- | ------------- |
| A           | curry  | 2021-01-01 | 2021-01-07|
| A           | sushi  | 2021-01-01 | 2021-01-07|
| B           | sushi        | 2021-01-04 | 2021-01-09|

***

###  8. What is the total items and amount spent for each member before they became a member?

```sql
select m.customer_id,
		count(z.product_name) as total_items,
		sum(z.price) as amount_spent
from members m
left join sales s
on m.customer_id = s.customer_id and m.join_date > s.order_date
left join menu z
on s.product_id = z.product_id
group by m.customer_id
```
#### Result set:
| customer_id | total_items | amount_spent|
| ----------- | ----------- | ------------|
| A           | 2           | 25          |
| B           | 3           | 40          |

***

###  9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
with new_cte as
(
select customer_id,
case when product_name = 'sushi' then ((count(s.product_id) * price) * 20) 
	else ((count(s.product_id) * price) * 10)
end as total_points 
from sales s
left join menu m on s.product_id = m.product_id 
group by customer_id,product_name,price
)
select customer_id,sum(total_points) as total_points
from new_cte
group by customer_id
```
#### Result set:
| customer_id | total_points |
| ----------- | -------------|
| A           | 860          |
| B           | 940          |
| C           | 360          |

***

###  10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January
```sql
with cte_mem_points as
(
select m.customer_id as customer,
		sum(case 
				when s.order_date < m.join_date then 
					case
						when u.product_name = 'sushi' then (u.price * 20)
						else (u.price * 10)
					end
				when s.order_date > dateadd(day,6,m.join_date) Then
					case
						when u.product_name = 'sushi' then (u.price * 20)
					end
				else (u.price * 20)
			end
		) as total_points_earned
from members m
join sales s
on m.customer_id = s.customer_id
join menu u
on s.product_id = u.product_id

group by m.customer_id
)

Select * 
from cte_mem_points
order by customer;

```
#### Result set:
| customer_id | customer_points |
| ----------- | --------------- |
| A           | 1370           |
| B           | 700            |

