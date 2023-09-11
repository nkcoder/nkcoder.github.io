---
title: 'PostgreSQL refresh 02: string functions'
date: 2023-09-11T16:21:48+10:00
tags:
- sql
draft: false
---

```sql

--- String Functions and Operators
--- doc: https://www.postgresql.org/docs/9.1/functions-string.html
--- operators: ||
--- functions: lower(), upper(), substring(), position(), trim(), btrim(), length(), left(), convert(), ltrim(), right(), rtrim()

SELECT first_name || ' ' || last_name AS customer_name
FROM customer;

select lower(left(first_name, 1)) || lower(last_name) || '@gmail.com'
from customer;

select rtrim(email, '.org'), position('@' in email), substring(email from position('@' in email))
from customer;

-- to_char() and left()
select b.starttime, f.name
from bookings as b inner join facilities f on f.facid = b.facid
where to_char(b.starttime, 'YYYY-mm-dd') = '2012-09-21'
and left(f.name, 12) = 'Tennis Court';

-- substring()
select b.starttime, f.name
from bookings as b inner join facilities f on f.facid = b.facid
where to_char(b.starttime, 'YYYY-mm-dd') = '2012-09-21'
  and substring(f.name from 1 for 12) = 'Tennis Court';

-- ||
select distinct b.starttime
from bookings as b inner join members m on m.memid = b.memid
where (m.firstname || ' ' || m.surname) = 'David Farrell';
```