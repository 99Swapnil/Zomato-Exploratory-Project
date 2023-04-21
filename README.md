# Zomato-Exploratory-Project
Zomato Exploratory Project

use zomato;
drop table if exists users;
create table users 
		(
        user_id integer not null,
        signup_date date
        );
insert into users (user_id,signup_date)
values (1,'2019-09-04'),
	   (2,'2015-01-15'),
       (3,'2014-11-04');

drop table if exists product;
create table product 
		(
        product_id integer not null, 
		product_name varchar(10) not null,
        price integer
        );
insert into product(product_id,product_name, price)
values (1,"p1", 980),
	   (2,"p2", 870),
       (3,"p3", 330);

drop table if exists goldusers_signup;
create table goldusers_signup
	(
    user_id int not null,
	gold_signup_date date 
    );
insert into goldusers_signup(user_id,gold_signup_date)
values (1,'2017-09-22'),
	   (3,'2017-04-21');

use zomato;
drop table if exists sales;       
create table sales
		(user_id int not null,
		 created_at date,
		 product_id int not null);
insert into sales(user_id,created_at, product_id)
values(1,'2017-04-19',2),
	  (3,'2019-12-18',1),
      (2,'2020-07-20',3),
      (1,'2019-10-23',2),
      (1,'2018-03-19',3),
      (3,'2016-12-20',2),
      (1,'2016-09-11',1),
      (1,'2016-05-20',3),
      (2,'2017-09-24',1),
      (1,'2017-03-11',2),
      (1,'2016-03-11',1),
      (3,'2016-11-10',1),
      (3,'2017-12-07',2),
      (3,'2016-12-15',2),
      (2,'2017-11-08',2),
      (2,'2018-09-10',3);

#Start of Project 
         
select * from sales;
select * from goldusers_signup;
select * from product;
select * from users;

#1. What is the total amount each customer spent on zomato?

select sales.user_id,
	   sum(product.price) as Total_Amount_Spent
from sales
inner join product 
on sales.product_id=product.product_id
group by user_id 
order by user_id asc;

#2. How many days has each customer visited zomato?

select sales.user_id,
	   count(distinct sales.created_at) as Number_of_days
from sales
group by user_id;

# 3. What was the first product purchased by each customer?

select* from 
(select *,rank() 
over(partition by user_id order by created_at) rnk 
from sales)sales
where rnk =1;

#4.1. What is the most purchased item of the menu and how many times it has been purchased by the customer?

select sales.product_id,
       count(sales.product_id) as Number_of_times_ordered
from sales 
group by product_id
order by count(product_id) desc;

#4.2. What is the most purchased item of the menu and how many times it has been purchased by the customer?

select *
from sales
where product_id=
(select sales.product_id
from sales
group by product_id
limit 1);

#4.3. What is the most purchased item of the menu and how many times it has been purchased by the customer?

select user_id,
count(product_id) as Number
from sales
where product_id=
	(select product_id
	 	 from sales 
	 group by product_id
	 limit 1)
group by user_id 
order by user_id asc;

#Interpretation of 4.3: User 1 ordered product_id 2 three times and so on.

#5. Which item was the most popular for each customer?

select*
from 
	(select *,
	rank()
	over(partition by user_id order by cnt desc)rnk 
	from 
		(select user_id,
		product_id,
		count(product_id) as cnt
		from sales
		group by user_id,
		product_id
		order by user_id asc)sales)
	sales
where rnk = 1;

#6. Which item was purchased first by the customer after they became a member?

select * 
from 
	(select c.*,rank() 
	over (partition by user_id order by created_at) rnk 
	from
		(select a.user_id, a.created_at, a.product_id,b.gold_signup_date
		 from sales a
		 inner join goldusers_signup b
		 on a.user_id=b.user_id
		 and a.created_at>=b.gold_signup_date) c)
	d
where rnk = 1;

#7. Which item was purchased just before the customer became a member?

select *
from 
	(select c.*,rank() over(partition by user_id order by created_at desc)rnk
	 from
		(select a.user_id, a.created_at, a.product_id, b.gold_signup_date
		 from sales a 
		 inner join goldusers_signup b
		 on a.user_id=b.user_id
		 and a.created_at<=b.gold_signup_date)c)
	 d 
where rnk = 1;

#8. What is the total orders and amount spent for each member before they become a member?

select user_id, count(created_at) order_purchased, sum(price) total_amt_spent
from 
	(select c.*,d.price from 
		(select a.user_id, a.created_at, a.product_id, b.gold_signup_date
		 from sales a 
		 inner join goldusers_signup b
		 on a.user_id=b.user_id
		 and a.created_at<=b.gold_signup_date)c
	inner join product d 
	on c.product_id=d.product_id)e
group by user_id 
order by user_id asc;

#9. If buying each product generates points for eg Rs.5 = 2 zomato points and each product has different purchasing points for eg p1 Rs.5= 1 zomato point, for p2 Rs.10=5 zomato point and p3 Rs. 5=1 zomato point, calculate the points collected by each customer and for which product most points have been given till now?

#First part

use zomato;
select g.*, Total_points_earned*2.5 Total_money_earned
from
	(select user_id, sum(Total_Points) Total_points_earned
	from 
		(select e.*,  floor(Amount/ points) Total_Points 
		 from
			(select d.*, case when product_id=1 then 5 
			 when product_id=2 then 2
			 when product_id=3 then 5 
             else 0 end as points from
									 (select c.user_id, c.product_id, sum(price) Amount
									  from 
										(select a.user_id,
										 a.product_id,
										 b.price
										 from sales a
										 inner join product b
										 on a.product_id=b.product_id) c
			group by user_id, product_id
			order by user_id)d)e)
		f 
group by user_id)g;

#Extended Part

select e.*,  floor(Amount/ points) Total_Points 
from
(select d.*, case when product_id=1 then 5 
			when product_id=2 then 2
			when product_id=3 then 5 
            else 0 end as points from
(select c.user_id, c.product_id, sum(price) Amount
from 
(select a.user_id,
	   a.product_id,
       b.price
from sales a
inner join product b
on a.product_id=b.product_id) c
group by user_id, product_id
order by user_id)d)e;

#Second Part
use zomato; select *
from 
(select g.*, rank() over(order by Total_points_earned desc)rnk from
(select product_id, sum(Total_Points) Total_points_earned
from 
(select e.*,  floor(Amount/ points) Total_Points 
from
(select d.*, case when product_id=1 then 5 
			when product_id=2 then 2
			when product_id=3 then 5 
            else 0 end as points 
from
(select c.user_id, c.product_id, sum(price) Amount
from 
(select a.user_id,
	   a.product_id,
       b.price
from sales a
inner join product b
on a.product_id=b.product_id) c
group by user_id, product_id
order by user_id)d)e)f 
group by product_id)g)h
where rnk = 1;

#10. In the first one year after a customer joins the gold program(including their joining date)
 #irrespective of what the customer has purchased they earn 5 zomato points for every 10 rs spent 
 #who earned more 1 or 3 nd what was their points earnings in their first year?
 
# 1 zp = 2 Rs
#0.5 zp=1 Rs.
 
use zomato;
select c.*,d.price*0.5 total_points_earned from 
(select a.user_id, a.created_at, a.product_id,b.gold_signup_date
from sales a
inner join goldusers_signup b
on a.user_id=b.user_id
and a.created_at>=b.gold_signup_date 
and created_at<= DATE_ADD(gold_signup_date, interval 1 year))c
inner join product d on c.product_id=d.product_id;

#11. rank all the txns of the customers

select*,rank() over (partition by user_id order by created_at)rnk 
from sales;

#12. rank all the txns for each member whenever they are a zomato gold member for every non gold member txn mark as na

select c.*, case when gold_signup_date is null then 'NA' else 
rank() over(partition by user_id order by created_at desc)end as rnk 
from
(select a.user_id, a.created_at, a.product_id,b.gold_signup_date
from sales a
left join goldusers_signup b
on a.user_id=b.user_id
and a.created_at>=b.gold_signup_date)c;
