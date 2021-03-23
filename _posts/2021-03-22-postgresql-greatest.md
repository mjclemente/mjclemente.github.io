---
published: true
title: "PostgreSQL - Only Update If Greater Than"
layout: post
tags: [postgresql,til]
---
A quick note on a very convenient PostgreSQL function that I  learned today - `GREATEST` - which can be used when you want a database column updated only if the incoming value is greater (more recent) than the existing value in the column.
<!--more-->

I'm coming from a MS SQL Server background, so I wasn't familiar with `GREATEST` / `LEAST`, which apparently are available in most other database engines, though they are not part of the SQL standard.[^1] From the [PostgreSQL docs](https://www.postgresql.org/docs/13/functions-conditional.html#FUNCTIONS-GREATEST-LEAST):

> The GREATEST and LEAST functions select the largest or smallest value from a list of any number of expressions.

While this is helpful to know, what I found even more useful was that you can use `GREATEST` to compare *the current value of a column* with an incoming value. Because the greater of the two is returned, this comparison can be used to conditionally update a column. That is, `GREATEST` functions almost like an `if` statement - update this column only if the provided value is greater than the current value: 

```sql
-- comparing numeric values

UPDATE my_table
SET max_age = GREATEST(max_age, :new_value)
WHERE id = :id

-- max_age is changed only if :new_value is larger
```

As you'd expect, `GREATEST` can be used to compare numeric values, but I was very surprised to learn that it also works for comparing timestamps (which you wouldn't necessarily realize just by reading the documentation):

```sql
-- comparing date/times!

UPDATE my_table
SET last_seen_at = GREATEST(last_seen_at, :new_value)
WHERE id = :id

-- last_seen_at is changed only if :new_value is more recent
```

This was my use case; I had a table with columns tracking a value's total number of occurrences and the date/time of the latest. The logs being parsed were not in chronological order and came from various sources. Using `GREATEST` simplified the logic needed during the update process - I could use a single query to increment the count every time, but only change the "most recent" date column when a newer timestamp appeared:

```sql
UPDATE my_table
SET total_count = total_count + 1,
last_seen_at = GREATEST(last_seen_at, :new_value)
WHERE id = :id
```

Credit where it's due - this [post on EBD](https://www.enterprisedb.com/postgres-tutorials/examples-using-greatest-and-least-functions-postgresql) goes into a lot of detail, with examples, about how you can use `GREATEST` and `LEAST` - I found it practical and informative. Apparently these functions even work for strings - in effect, providing alphabetical comparison (though I don't know if that's necessarily the "right tool for the job"). Just another crazy thing that you can do with PostgreSQL.

____

[^1]: Here's the `GREATEST` / `LEAST` documentation for [MySQL](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html), [MariaDB](https://mariadb.com/kb/en/greatest/), [Oracle](https://docs.oracle.com/database/121/SQLRF/functions078.htm#SQLRF00645), and [Db2](https://www.ibm.com/support/knowledgecenter/SSEPGG_11.5.0/com.ibm.db2.luw.sql.ref.doc/doc/r0052623.html) - with credit to this [StackExchange post](https://dba.stackexchange.com/questions/187090/does-sql-server-support-greatest-and-least-if-not-what-is-the-common-workaround) for those links.
