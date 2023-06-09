## 8-week-SQL-Challenge
<img src="https://8weeksqlchallenge.com/images/case-study-designs/1.png" alt="Image" width="450" height="450">

View the case study [here](https://8weeksqlchallenge.com/case-study-1/)

## Intro
This Project was my attempt on the Week 1 Challenge this was done in Microsoft SQL server
## Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.
## Data/Table Used
- Table 1:Sales: The sales table captures all customer_id level purchases with an corresponding order_date and product_id information for when and what menu items were ordered.
- Table 2:Menu:The menu table maps the product_id to the actual product_name and price of each menu item.
- Table 3:Members: The members table captures the join_date when a customer_id joined the beta version of the Danny’s Diner loyalty program.

## Entity Relationship Diagram
![alt text](https://github.com/olin1233/8-week-SQL-Challenge/blob/main/Week%201%20Danny's%20Diner/ERD%20Danny%20Diner.jpg)

## Case Study Questions
1. What is the total amount each customer spent at the restaurant?
2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
8. What is the total items and amount spent for each member before they became a member?
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
