---
title: 'PostgreSQL refresh 06: date and time'
date: 2023-09-11T16:23:29+10:00
tags:
- sql
draft: false
---

```sql
-- PostgreSQL time data types
--- TIME: contains only time
--- DATE: contains only date
--- TIMESTAMP: contains date and time
--- TIMESTAMPTZ: contains date, time and timezone
--- related functions and operators: TIMEZONE, NOW, TIMEOFDAY, CURRENT_TIME, CURRENT_DATE

--- show the values of all configuration parameters, with descriptions
SHOW ALL;

--- display the current setting of run-time parameters
SHOW TIMEZONE;

--- date and time functions
SELECT now(), timeofday(), CURRENT_TIME, CURRENT_DATE, CURRENT_TIMESTAMP;

------------------------------------------------------------------------------------------

-- extract information from a time based data type using
--- EXTRACT(): retrieves subfields such as year/month/hour from date/time values
--- AGE(): subtract argument from current_date
--- TO_CHAR(): convert various data types (date/time, numeric) to formatted strings, and convert
---- from formatted strings to specific data types.

SELECT extract(year from payment_date) as payment_year,
       extract(quarter from payment_date) as payment_quarter,
       extract(month from payment_date) as payment_month,
       extract(day from payment_date) as payment_day,
       extract(hour from payment_date) as payment_hour,
       extract(minute from payment_date) as payment_minute,
       extract(second from payment_date) as payment_second,
       extract(century from payment_date) as payment_century,
       extract(dow from payment_date) as payment_dow
FROM payment;

--- example result: 16 years 4 mons 22 days 1 hours 34 mins 13.003423 secs
SELECT age(payment_date)
FROM payment;

--- example result: 02-15-2007,THURSDAY /FEBRUARY /2007,   7.99
SELECT to_char(payment_date, 'mm-dd-YYYY'),
       to_char(payment_date, 'DAY/MONTH/YYYY'),
       to_char(amount, '999D00')
FROM payment;

--- During which months did payments occur? Format your answer to return back the full month name.
SELECT distinct to_char(payment_date, 'MONTH')
FROM payment;

--- How many payments occurred on a Monday using extract()
SELECT count(*)
FROM payment
WHERE extract(dow from payment_date) = 1;  -- The day of the week as Monday (1) to Sunday (7)

--- How many payments occurred on a Monday using to_char()
SELECT count(*)
FROM payment
WHERE to_char(payment_date, 'D') = '2';    -- day of the week, Sunday (1) to Saturday (7)

-- date()
select b.starttime, f.name
from bookings as b inner join facilities f on f.facid = b.facid
where f.name like 'Tennis Court%'
and date(b.starttime) = '2012-09-21';
```