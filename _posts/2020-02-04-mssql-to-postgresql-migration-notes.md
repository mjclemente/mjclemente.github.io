---
date: 2020-02-04
published: true
title: "Notes on Migrating from MS SQL Server to PostgreSQL"
layout: post
tags: [postgres,postgresql,mssql,sql server, microsoft sql server,migration]
---
I'm not a database guru or SQL sherpa, but from time to time I do fill the role of de facto DBA. The following are some notes and observations from a recent, successful, migration from Microsoft SQL Server to PostgreSQL. Is it everything you need to know? Absolutely not. But there are some points and resources that will hopefully be helpful.
<!--more-->

* TOC
{:toc}
## A Note on Versions

Many of the articles that I came across comparing SQL Server and Postgres, aside from those that were useless from a practical standpoint, were outdated. That is, comparing PG 9.3 with SQL Server 2014 just isn't going to paint an accurate picture of where they stand today (February 2020). So, for context, this was a migration from a dedicated instance of MS SQL Server 2016, as well as cloud-based [Azure SQL Databases](https://azure.microsoft.com/en-us/services/sql-database/) to [DigitalOcean's managed PostgreSQL hosting](https://www.digitalocean.com/products/managed-databases-postgresql/), running PostgreSQL v11. 

## Moving the Database

I thought this would be the hard part, but the actual migration of the data from one database engine to the other turned out to be rather straightforward, thanks to the aptly named tool: [MS SQL to PostgreSQL](https://www.convert-in.com/mss2pgs.htm). The data transfers was fast, it retained foreign keys and indexes, and allowed a high degree of control over the operation and handling of tables and columns. A few dry runs to testing servers were necessary to work out the exact settings and process, but we were very happy with the results.[^1] The more labor intensive part of changing database engines was updating our application code to be PostgreSQL compatible.

## Updating Application Code for PostgreSQL

While not comprehensive, here are some of the differences between SQL Server and Postgres that we needed to account for in our applications.[^2] In no particular order:

### Lowercase table and column names
PostgreSQL handles the casing of identifiers differently than SQL Server, which is to say, *case matters*. There are [numerous](https://github.com/ontop/ontop/wiki/Case-sensitivity-for-SQL-identifiers) [discussions](https://dev.to/lefebvre/dont-get-bit-by-postgresql-case-sensitivity--457i) about this online, so I won't repeat them. The approach we took was to lowercase all identifers. At the database level, this was done by the data transfer tool mentioned earlier, but within the application, we had to do it manually. No magic tool here. We worked our way through the code base, lowercasing as we went. Visual Studio Code's shortcut **Command-K-L** (⌘+K+L) came in very handy.

### Switch from TOP to LIMIT

Postgres, like MySQL, applies limits to the result set at the end of the SQL statement using `LIMIT`, while SQL Server does this at the outset, using `TOP`. So we had to go through updating queries from `SELECT TOP 1` to `SELECT ... LIMIT 1`. It wasn't hard; it just takes a while.

### Use RETURNING with INSERT to retrieve identity
When an INSERT statement generates an identity value, applications frequently have need to retrieve and use that generated identifier. For example, if you insert a new client, you probably want to know the client's ID. MS SQL Server provides `SCOPE_IDENTITY() ` for this purpose, and some programming languages will return the generated identifier value automatically following an INSERT query. With PostgreSQL, you need to modify your INSERT statement, adding a line that specifies the column value you need returned. Here's how that looks: 

```sql
INSERT INTO client(firstname, lastname)
VALUES ('Testy', 'McTester')
RETURNING id;
-- returns the value generated for this record in the id column
```

  The reason for this has to do with Postgres apparently not having a concept of a table's "identity". You can read more about it [here on Stack Overflow](https://stackoverflow.com/questions/2944297/postgresql-function-for-last-inserted-id).

### Use `CONCAT` instead of `+`
Postgres does not allow the use of the `+` operator for string concatenation, so queries composed like the example here will fail:

```sql
SELECT firstname + ' ' + lastname AS fullname
```

This syntax will need to be rewritten, using `CONCAT`, as shown here:

```sql
SELECT CONCAT(firstname, ' ', lastname) AS fullname
```

While putting this together, I learned that `CONCAT` is [supported in SQL Server](https://docs.microsoft.com/en-us/sql/t-sql/functions/concat-transact-sql?view=sql-server-ver15) - I just hadn't ever used it. While a bit strange at first, I prefer the `CONCAT` syntax now; I find the resulting code to be cleaner - easier to read, write, and modify.

### Accounting for the Boolean data type

In SQL Server, the *bit* data type is typically used as a stand in for boolean values. Postgres provides a true boolean data type, unsurprisingly named *boolean*. However, because they are fundementally different data types (though meant to convey the same thing), they behave in subtly different ways.  

For example, the bit data type can be compared to integers, as well as boolean string values, as seen in these SQL Server examples:

```sql
-- SQL Server
-- bit to integer comparison (valid)
WHERE status = 1

-- bit to boolean string comparison (valid)
WHERE status = 'true'
```

However, SQL Server does not allow columns of the bit data type to be actually used as booleans:

```sql
-- SQL Server
-- bit to boolean comparison (not valid)
WHERE status = TRUE

-- bit as boolean implying truth (not valid)
WHERE status
```

On the other hand, the PostgreSQL boolean data type *cannot be compared with integers*; it results in the following error: `ERROR: operator does not exist: boolean = integer` (which we saw a lot of) - that is, the following is invalid in Postgres:

```sql
-- Postgres
-- boolean to integer comparison (not valid)
WHERE status = 1
```

However, as you would expect, the boolean data type in Postgres does function as a true boolean, so the following examples are valid in Postgres:

```sql
-- Postgres
-- boolean to boolean comparison (valid)
WHERE status = TRUE

-- boolean auto cast to boolean string (valid)
WHERE status = 'TRUE'
WHERE status = 'T'

-- boolean implying true/false (valid)
WHERE status
WHERE NOT status
```

All of which is a long-winded way to say that there were instances in our codebase where we were comparing boolean columns to 1 or 0, and we needed to update them to use the actual booleans TRUE or FALSE.

### Different reserved keywords
I don't have a comprehensive list of the different reserved keywords - just a note that this difference can cause issues. In our case, we had a column named `offset`, which was acceptable in SQL Server, but which we needed to quote in order to use in Postgres, as the keyword is reserved. For reference, here's the list of [SQL Server reserved keywords](https://docs.microsoft.com/en-us/sql/t-sql/language-elements/reserved-keywords-transact-sql?view=sql-server-ver15), and here are the [Postgres SQL keywords](https://www.postgresql.org/docs/current/sql-keywords-appendix.html).

### Formatting dates and times:

This is a minor difference, but even minor differences need to be changed. When formatting dates, as in a SELECT statement, the function and syntax used are different. While SQL Server used `FORMAT`, Postgres uses  `to_char`, as in this example:

```sql
-- SQL Server 
SELECT 
FORMAT(lastmodified,'M/d/yy h:mm tt') AS readabledatetime

-- Postgres
SELECT 
to_char(lastmodified,'FMMM/FMDD/YY HH12:MI AM') AS readabledatetime

-- Example Output: 8/1/19 8:21 PM
```

Here are the [Postgres docs on formatting](https://www.postgresql.org/docs/11/functions-formatting.html), which you'll need to review if this type of change impacts you. I was impressed with how configurable the formatting options were - for instance, note that in the example above, the `FM` prefix is used to supress leading 0s.

### Switch from `STUFF... FOR XML PATH('')` to `string_agg`

If you've never needed to use SQL Server's `STUFF` with `FOR XML PATH('')` to concatenate the results of a subquery as a list... then count yourself lucky and just skip this section. If you have, then you know that the syntax is not straightforward and can be difficult to parse.[^3] Here's an example of what I'm talking about (along with a [SQL fiddle](http://sqlfiddle.com/#!18/c0020d/4/0) if you want to follow along):

```sql
SELECT
  a.author,
  STUFF((SELECT ', ' + title
  FROM author_title
  WHERE authorid = a.authorid
  FOR xml PATH ('')), 1, 2, ''
  ) AS works
FROM author a
```

Thankfully, PostgreSQL uses a more intuitive function, `string_agg` to provide this functionality, so here's how you would [rewrite](http://sqlfiddle.com/#!15/c065b/2/0) the above example:

```sql
SELECT
  a.author,
  (SELECT string_agg(title, ', ')
  FROM author_title
  WHERE authorid = a.authorid) AS works
FROM author a
```

Additionally, the `string_agg` function actually allows you to dispense with the subquery and use a JOIN/GROUP BY instead, so you could also [rewrite](http://sqlfiddle.com/#!15/c065b/3/0) the example as:

```sql
SELECT
  a.author,
  string_agg(at.title, ', ') AS works
FROM author a
LEFT OUTER JOIN author_title at
  ON a.authorid = at.authorid
GROUP BY a.author
```

While database incompatibilities are always work to resolve, cases like this are nice, in that the changes at least feel like improvements. That said, I also found out that `string_agg` is now [available in SQL Server](https://docs.microsoft.com/en-us/sql/t-sql/functions/string-agg-transact-sql?view=sql-server-ver15), as of SQL Server 2017. 

## Final Notes

Obviously, I haven't covered all of the differences between Microsoft SQL Server and PostreSQL. The process and issues you encounter will depend highly on your application architecture. For example, we don't use common table expressions (CTEs), which apparently [can cause performance issues](https://www.brentozar.com/archive/2018/08/two-important-differences-between-sql-server-and-postgresql/) in all but the latest (v12) versions of Postgres. Bottom line - you can do all the reading you want about incompatibilities, but there's no substitute for testing, testing, testing your application code.

For a further discussion of this topic, I'd recommend this Reddit thread: [Coming from SQL Server](https://www.reddit.com/r/PostgreSQL/comments/ay7tw5/coming_from_sql_server/); it touches on some of the benefits of moving to Postgres as well, which I really don't have the space to explore here. If you've made a similar migration, or just have more familiarity with database engines, please share you insights in the comments!

____

[^1]: The biggest issue that we encountered was actually resolved with the latest update to the app ([v4.5](https://www.convert-in.com/mss2pgs_changes.htm)) - in earlier versions, when transferring tables with a primary key comprised of multiple columns, the order of the columns within the key/index was sometimes changed. I should also note that it's not a free tool. But, at $49 (with a limited, free trial), we found it well worth the time it saved.
[^2]: And yes, I know that a query-builder library or ORM would mitigate much of this, but it's still good to know.
[^3]: And if you're looking for a little more clarity, here's a fairly detailed Stack Overflow explanation: [How Stuff and 'For Xml Path' work in Sql Server](https://stackoverflow.com/questions/31211506/how-stuff-and-for-xml-path-work-in-sql-server)

