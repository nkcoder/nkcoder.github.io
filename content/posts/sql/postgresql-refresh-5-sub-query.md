---
title: 'PostgreSQL refresh 05: sub-query'
date: 2023-09-11T16:23:01+10:00
tags:
- sql
draft: false
---

```sql
-- Sub query
--- sub queries are executed first. 
--- it is unwise to write a subquery that has side effects (such as calling sequence functions)

select title, rental_rate
from film
where rental_rate > (select avg(rental_rate) from film);

-- Exists

EXISTS (subquery)

-- The subquery is evaluated to determine whether it returns any rows. If it returns at least one row, the result of EXISTS is “true”; if the subquery returns no rows, the result of EXISTS is “false”.
-- Since the result depends only on whether any rows are returned, and not on the contents of those rows, the output list of the subquery is normally unimportant. A common coding convention is to write all EXISTS tests in the form `EXISTS(SELECT 1 WHERE ...)`. 
-- For each row in the surrounding query, only if `EXISTS(subquery)` evaluates true, the row is returned.

-- for each customer row, if `exists(subquery)` is true, which means its payment amount > 10, the row is returned.
select first_name, last_name
from customer as c
where exists(
    select *
    from payment as p
    where c.customer_id = p.customer_id
    and p.amount > 10
          );

-- can use `NOT EXISTS()` to reverse the condition
select first_name, last_name
from customer as c
where not exists(
    select *
    from payment as p
    where c.customer_id = p.customer_id
    and p.amount > 10
          );

-- IN

expression IN (subquery)

-- The right-hand side is a parenthesized subquery, which must return exactly one column.

select film_id, title
from film
where film_id in (
    select film_id
    from inventory inner join rental on inventory.inventory_id = rental.inventory_id
    where rental.rental_date > '2005-05-29' and rental.rental_date < '2005-05-31'
    )
order by film_id;

-- not in
select film_id, title
from film
where film_id not in (
    select film_id
    from inventory inner join rental on inventory.inventory_id = rental.inventory_id
    where rental.rental_date > '2005-05-29' and rental.rental_date < '2005-05-31'
    )
order by film_id;

-- ANY/SOME

--- expression operator ANY (subquery)
--- expression operator SOME (subquery)

--- The right-hand side is a parenthesized subquery, which must return exactly one column.
--- The left-hand expression is evaluated and compared to each row of the subquery result using the given operator, which must yield a Boolean result. 
--- The result of ANY is “true” if any true result is obtained. The result is “false” if no true result is found (including the case where the subquery returns no rows).

--- `SOME` is a synonym for `ANY`. `IN` is equivalent to `= ANY`.

--- IN is equivalent to = ANY
select film_id, title
from film
where film_id = any (
    select film_id
    from inventory inner join rental on inventory.inventory_id = rental.inventory_id
    where rental.rental_date > '2005-05-29' and rental.rental_date < '2005-05-31'
    )
order by film_id;

--- SOME is equivalent to ANY
select film_id, title
from film
where film_id > some (
    select film_id
    from inventory inner join rental on inventory.inventory_id = rental.inventory_id
    where rental.rental_date > '2005-05-29' and rental.rental_date < '2005-05-31'
    and film_id > 600
    )
order by film_id;

-- ALL

--- expression operator ALL (subquery)

--- The right-hand side is a parenthesized subquery, which must return exactly one column. 
--- The result of ALL is “true” if all rows yield true (including the case where the subquery returns no rows). The result is “false” if any false result is found.

select film_id, title
from film
where film_id <= ALL (
    select film_id
    from inventory inner join rental on inventory.inventory_id = rental.inventory_id
    where rental.rental_date > '2005-05-29' and rental.rental_date < '2005-05-31'
    and film_id > 100 and film_id < 200
    )
order by film_id;
```
