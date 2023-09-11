---
title: 'PostgreSQL refresh 03: group by'
date: 2023-09-11T16:22:08+10:00
tags:
- sql
draft: false
---

```sql
-- most common aggregate functions
--- COUNT(): simply returns the number of rows
--- AVG(): returns a floating point, you can use ROUND() to specify precision after the decimal
--- MAX(): returns the maximum element
--- MIN(): returns the minimum element
--- SUM(): returns the sum
--- aggregate functions can appear in SELECT, HAVING and ORDER BY clauses, but not WHERE statements
select min(replacement_cost), max(replacement_cost), round(avg(replacement_cost), 2), sum(replacement_cost)
from film;
 
-- group by: aggregate on one or more columns
--- in the SELECT statement, columns must either have an aggregate function or be in the GROUP BY call
--- WHERE statements should not refer to the aggregation result, should use HAVING to filter on aggregation result
--- can use ORDER BY to sort results based on the aggregate, but make sure to reference the entire aggregate function

-- the total amount per customer per staff
select customer_id, staff_id, sum(amount)
from payment
group by customer_id, staff_id
order by customer_id, staff_id, sum(amount);

select date(payment_date), sum(amount)
from payment
group by date(payment_date)
order by date(payment_date), sum(amount) desc;

select staff_id, count(*)
from payment
group by staff_id
order by count(*) desc;

select rating, round(avg(replacement_cost), 2)
from film
group by rating;

select customer_id, sum(amount)
from payment
group by customer_id
order by sum(amount) desc
limit 5;

-- HAVING: filter on the aggregate
--- we can use filter on non-aggregate columns in WHERE because they're valid before GROUP BY
--- we can NOT use filter on aggregate columns in WHERE because they're valid AFTER GROUP BY
select customer_id, sum(amount)
from payment 
where customer_id not in (184, 477, 550)
group by customer_id
having sum(amount) > 100
order by sum(amount) desc;

select customer_id, count(*)
from payment
group by customer_id
having count(*) >= 40;

select customer_id, sum(amount)
from payment
where staff_id = 2
group by customer_id
having sum(amount) > 110;
```