---
title: 'PostgreSQL refresh 01: basic query'
date: 2023-09-11T16:20:48+10:00
tags:
- sql
draft: false
---

```sql
-- select
select first_name, last_name, email
from customer;

-- distinct: both the following two forms work
select distinct rating
from film;
select distinct(rating)
from film;

-- count
select count(distinct first_name ) 
from actor;
-- readable
select count(distinct(first_name)) 
from actor;
-- this doesn't work, parentheses are required
-- select count distinct first_name
-- from actor;

-- where
select email
from customer
where first_name = 'Nancy' and last_name = 'Thomas';

-- limit
select customer_id
from payment
order by payment_date ASC
limit 10;

-- between and
--- both inclusive
--- not between and
--- when used for timestamp, it includes the time part
select count(*)
from payment
where amount between 5 and 6;

select count(*)
from payment
where amount not between 5 and 6;

select *
from payment
where payment_date between '2007-05-12' and '2007-05-15'
order by payment_date asc;

-- in, not in
select *
from city
where city in ('Hoshiarpur', 'Hino');
information_schema
select *
from city
where city not in ('Hoshiarpur', 'Hino');

-- like, ilike
--- like: case sensitive
--- ilike: case insensitive
--- like: match any characters
--- _: match only one character
select *
from customer
where first_name like 'M%' and last_name ilike 's%';

select *
from customer
where first_name like '_her%';
```
