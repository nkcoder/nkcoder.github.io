---
title: 'PostgreSQL refresh 04: join'
date: 2023-09-11T16:22:28+10:00
tags:
- sql
draft: false
---

```sql
-- AS
--- it's an alias of a column or an aggregate
--- it's executed at the very end of a query, so we cannot use the alias inside a WHERE operator
select count(*) as num_transactions
from payment;

select customer_id, sum(amount) as total_spent
from payment
group by customer_id
having sum(amount) > 100;       -- cannot use 'total_spent' here because it doesn't exist yet

-- INNER JOIN
--- results with a set of records that match in both tables
--- the order of the tables to INNER JOIN doesn't matter
--- if you use just JOIN without the INNER, PostgreSQL will treat it as an INNER JOIN
select payment_id, customer.customer_id, first_name, last_name      -- we need to prefix the table if the join column appears in SELECT
from customer inner join payment on customer.customer_id = payment.customer_id;

---------------------------------------------------------------------------------------------------------------------

-- FULL [OUTER] JOIN
--- OUTER is optional
--- grab everything in both tables, columns that don't match will be filled with NULL
---- if table A has 200 records, table B has 300 records, and the matched records is 150, then the result will be:
---- 150 + (200 - 150) + (300 - 150) = 350
--- the table order doesn't matter
--- use WHERE to get rows unique to either table (rows not found in both tables), this is opposite to INNER JOIN

--- get everything from customer and payment
--- rows in both tables, rows in only one table and the value for the other table fields are null
select *
from customer full outer join payment
on customer.customer_id = payment.customer_id;

--- do we have any customer without payment, or any payment without customer?
--- this is opposite to INNER JOIN
select *
from customer full outer join payment
on customer.customer_id = payment.customer_id
where customer.customer_id is null or payment.payment_id is null;

---------------------------------------------------------------------------------------------------------------------

-- A LEFT [OUTER] JOIN B
--- OUTER is optional
--- return records in table A, if there is no match with table B, the the column values are null
--- the table order does matter
--- use WHERE to get rows unique to table A

--- get all films and their inventory (even the films don't have inventory)
select film.film_id, film.title, inventory.inventory_id, inventory.store_id
from film left join inventory on film.film_id = inventory.film_id;

--- do we have any film that are not linked to inventory?
select film.film_id, film.title, inventory.inventory_id, inventory.store_id
from film left join inventory on film.film_id = inventory.film_id
where inventory.film_id is null;

---------------------------------------------------------------------------------------------------------------------

-- A RIGHT [OUTER] JOIN B
--- OUTER is optional
--- essentially the same as LEFT JOIN, except the tables are switched
--- return records in table B, if there is no match with table A, the the column values are null

--- do we have any film that are not linked to inventory?
select film.film_id, film.title, inventory.inventory_id, inventory.store_id
from inventory right join film on film.film_id = inventory.film_id
where inventory.film_id is null;

---------------------------------------------------------------------------------------------------------------------

-- UNION
--- combine the result-set of two or more SELECT statements
--- directly concatenate two results together
--- the columns in SELECT statements MUST be exactly the same
select customer_id
from payment
union
select customer_id
from customer;

---------------------------------------------------------------------------------------------------------------------

-- self join is using standard join to join two copies of the same table on different columns.
--- find the film pair whose film length is the same
select f1.title, f2.title
from film as f1
inner join film as f2
on f1.film_id != f2.film_id     -- if f1.film_id = f2.film_id, then the same film will be mapped to itself
and f1.length = f2.length;
```
