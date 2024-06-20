# Danny's diner

welcome to danny's diner data analysis repository! this project contains sql queries used to analyze customer behavior and sales patterns at danny's diner. below are the insights we gathered using various sql queries.

## table of contents

1. [total amount spent by each customer](#total-amount-spent-by-each-customer)
2. [days each customer visited](#days-each-customer-visited)
3. [first item purchased by each customer](#first-item-purchased-by-each-customer)
4. [most purchased item on the menu](#most-purchased-item-on-the-menu)
5. [most popular item for each customer](#most-popular-item-for-each-customer)
6. [first item purchased after becoming a member](#first-item-purchased-after-becoming-a-member)
7. [item purchased before becoming a member](#item-purchased-before-becoming-a-member)
8. [total items and amount spent before becoming a member](#total-items-and-amount-spent-before-becoming-a-member)
9. [points for each customer](#points-for-each-customer)

## total amount spent by each customer

1. What is the total amount each customer spent at the restaurant?
```sql
  select customer_id, sum(price) as total_spent 
  from sales 
  inner join menu on sales.product_id = menu.product_id
  group by customer_id
  order by sum(price) desc;
```
<img width="241" alt="Screenshot 2024-06-21 at 1 39 27 AM" src="https://github.com/k-Rohit/SQL-Case-Studies/assets/93335681/8e3b5f32-8031-45dd-83fa-754bfc56311c">

2. How many days has each customer visited the restaurant?
```sql
select customer_id, count(order_date) as no_of_days_visited 
from sales
group by customer_id;
```
<img width="334" alt="Screenshot 2024-06-21 at 1 41 06 AM" src="https://github.com/k-Rohit/SQL-Case-Studies/assets/93335681/4698f80b-af50-43db-8905-18b2223fd62f">

3. What was the first item from the menu purchased by each customer?
```sql
with cte as (select s.customer_id ,m.product_name, s.product_id, 
row_number() over(partition by s.customer_id order by customer_id) as rn
from menu m inner join sales s on m.product_id = s.product_id)
select customer_id, product_name from cte where rn = 1;
```
<img width="328" alt="Screenshot 2024-06-21 at 1 42 03 AM" src="https://github.com/k-Rohit/SQL-Case-Studies/assets/93335681/28a085db-533c-4d92-8f60-82e45704d2ea">

4. What is the most purchased item on the menu and how many times did all customers purchase it?
```sql
SELECT s.product_id, m.product_name, COUNT(s.product_id) AS frequency
FROM sales s
INNER JOIN menu m ON s.product_id = m.product_id
GROUP BY s.product_id, m.product_name
ORDER BY s.product_id DESC
limit 1;
```
<img width="528" alt="Screenshot 2024-06-21 at 1 42 51 AM" src="https://github.com/k-Rohit/SQL-Case-Studies/assets/93335681/ca97b0c6-c675-4a81-8af0-b71cc18f3f63">

5. Which item was the most popular for each customer?
```sql
SELECT customer_id, product_id, product_name as most_popular
FROM ( SELECT s.customer_id,s.product_id,m.product_name,
COUNT(*) AS frequency, ROW_NUMBER() OVER (PARTITION BY s.customer_id ORDER BY COUNT(*) DESC) AS rn
FROM sales s
INNER JOIN menu m ON s.product_id = m.product_id
GROUP BY s.customer_id, s.product_id, m.product_name
) ranked
WHERE rn = 1;
```
<img width="557" alt="Screenshot 2024-06-21 at 1 43 33 AM" src="https://github.com/k-Rohit/SQL-Case-Studies/assets/93335681/62d4f698-90a5-4ff1-aaae-9d232bfa1e87">


6. Which item was purchased first by the customer after they became a member?
```sql
select customer_id, product_name as item_purchased
from ( select s.customer_id, m.product_name, s.order_date,
row_number() over (partition by s.customer_id order by s.order_date asc) as rn
from members me inner join sales s on me.customer_id = s.customer_id
inner join menu m on m.product_id = s.product_id where s.order_date > me.join_date) ranked
where rn = 1;
```
<img width="423" alt="Screenshot 2024-06-21 at 1 44 16 AM" src="https://github.com/k-Rohit/SQL-Case-Studies/assets/93335681/5988d86f-eb12-4ed3-8b62-42796499357e">


7. Which item was purchased just before the customer became a member?
```sql
SELECT s.customer_id, m.product_name as item_purchased
FROM members me
INNER JOIN sales s ON me.customer_id = s.customer_id
INNER JOIN menu m ON m.product_id = s.product_id
where order_date < join_date
order by price desc;
```
<img width="220" alt="Screenshot 2024-06-21 at 1 45 07 AM" src="https://github.com/k-Rohit/SQL-Case-Studies/assets/93335681/fe14fb8c-1583-4424-9cf6-f133108c7dc6">

8. What is the total items and amount spent for each member before they became a member?

```sql
select s.customer_id, count(s.product_id) as quantity, sum(m.price) as total_sales
from sales s
join menu m on m.product_id = s.product_id
join members mem on mem.customer_id = s.customer_id
where s.order_date < mem.join_date
group by s.customer_id;

```
<img width="227" alt="Screenshot 2024-06-21 at 1 46 42 AM" src="https://github.com/k-Rohit/SQL-Case-Studies/assets/93335681/35f6b4b5-7e2f-4d0c-ad30-99c7cdbcff26">

9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
with points as (
    select *, case when product_id = 1 then price*20
                   else price*10
               end as points
    from menu
)
select s.customer_id, sum(p.points) as points
from sales s
join points p on p.product_id = s.product_id
group by s.customer_id;

```
<img width="164" alt="Screenshot 2024-06-21 at 1 48 51 AM" src="https://github.com/k-Rohit/SQL-Case-Studies/assets/93335681/dd717622-166f-44b0-9b61-d8b241e1d68d">



