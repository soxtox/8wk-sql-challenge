Hi this is saulen shilpakar, I have accepted the 8 week sql challenge and here is the solution for the first week including the Bonus issue. dannys diner. Below is the script for creating the schema and table as shared in the challenge.


CREATE SCHEMA dannys_diner;
SET search_path dannys_diner;

CREATE TABLE sales (
  "customer_id" VARCHAR(1),
  "order_date" DATE,
  "product_id" INTEGER
);

INSERT INTO sales
  ("customer_id", "order_date", "product_id")
VALUES
  ('A', '2021-01-01', '1'),
  ('A', '2021-01-01', '2'),
  ('A', '2021-01-07', '2'),
  ('A', '2021-01-10', '3'),
  ('A', '2021-01-11', '3'),
  ('A', '2021-01-11', '3'),
  ('B', '2021-01-01', '2'),
  ('B', '2021-01-02', '2'),
  ('B', '2021-01-04', '1'),
  ('B', '2021-01-11', '1'),
  ('B', '2021-01-16', '3'),
  ('B', '2021-02-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-07', '3');
 

CREATE TABLE menu (
  "product_id" INTEGER,
  "product_name" VARCHAR(5),
  "price" INTEGER
);

INSERT INTO menu
  ("product_id", "product_name", "price")
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');
  

CREATE TABLE members (
  "customer_id" VARCHAR(1),
  "join_date" DATE
);

INSERT INTO members
  ("customer_id", "join_date")
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');

# What is the total amount each customer spent at the restaurant?

select d.customer_id, sum(price) as total_price  from menu a
inner join sales d on
a.product_id=d.product_id
group by customer_id;
# How many days has each customer visited the restaurant?



```python
select customer_id, count(distinct order_date) days 
from sales 
group by customer_id;
```


## What was the first item from the menu purchased by each customer?
with cte_table as (
select 
a.customer_id,
b.product_name , row_number() over (partition by customer_id order by order_date) as rn 
from sales a 
inner join menu b 
on a.product_id=b.product_id)
select customer_id,product_name from cte_table where rn=1;
## What is the most purchased item on the menu and how many times was it purchased by all customers?

select top(1) product_name , total
from ( 
select product_name, count(a.product_id) total 
from sales  a 
inner join menu b
on a.product_id=b.product_id 
group by b.product_name 
) as A
order by total desc;
## Which item was the most popular for each customer?
with popular_cte as (
select customer_id,product_name, count(s.product_id) as popular, DENSE_RANK() over(partition by s.customer_id order by count(s.product_id)desc) as rn 
from sales s 
inner join menu m 
on m.product_id=s.product_id 
group by customer_id, product_name
)
select * from popular_cte where rn=1;
## Which item was purchased just before the customer became a member?
with just_before_cte as (
select m.customer_id, s.order_date, mm.product_name, DENSE_RANK() over(partition by s.customer_id order by s.order_date desc) as rn 
from members m 
inner join sales s 
on m.customer_id=s.customer_id 
inner join menu mm on 
mm.product_id=s.product_id
where s.order_date<m.join_date 
)
select customer_id, order_date, product_name from just_before_cte where rn=1;
## What is the total items and amount spent for each member before they became a member?


select s.customer_id, sum(price) as rn 
from members m 
right join sales s 
on m.customer_id=s.customer_id 
inner join menu mm on 
mm.product_id=s.product_id
where s.order_date<m.join_date
group by s.customer_id

## If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?


with point_cte as (select customer_id, s.product_id,m.product_name,
point=case 
when m.product_name='sushi' then 2*m.price
else m.price
end
from sales s 
join menu m 
on s.product_id=m.product_id)

select customer_id, sum(point)as total_point from point_cte group by customer_id;
## In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

with jan_point as (
select me.customer_id, s.product_id,m.product_name, s.order_date,
point=case 
when m.product_name='sushi' or  s.order_date between DATEADD(DAY, 0, join_date) and DATEADD(DAY, 7, join_date) then 2*m.price
else m.price
end
from sales s 
join menu m 
on s.product_id=m.product_id
right join members me on
me.customer_id=s.customer_id
)
select customer_id,  sum(point) total_point from jan_point
 where order_date <cast('2021-02-01' as date)
group by customer_id;
# Bonus
For Bonus
FOR Bonus need to create a view first.
create view recreate_table as (
select s.customer_id as customer_id,order_date,product_name, price, case when s.order_date>=me.join_date then 'Y'
else 'N' 
end member from sales s left join menu m on  s.product_id=m.product_id 
left join members me on s.customer_id=me.customer_id);
select * from recreate_tableselect *, case when member = 'Y' then RANK() over (partition by customer_id, member order by order_date asc)
else null
end ranking from recreate_table