---
date: 2021-05-11
published: true
title: "Group By or Order By Column Position in a SQL Query"
layout: post
tags: [sql,til]
---
File this under interesting SQL features that I just learned - you can `GROUP BY` and/or `ORDER BY` the numeric column position in your `SELECT` statement, rather than writing out the full column identifier. If that's unclear, an example should help clarify.
<!--more-->

For a decade (or more), I've specified column identifiers in my queries when grouping and/or ordering, like so:

```sql
SELECT
    extract(year FROM b.completed_at) AS yyyy,
    g.name,
    count(b.book_id) AS the_count
FROM
    book AS b
    INNER JOIN genre AS g ON b.genre_id = g.genre_id
GROUP BY
    yyyy, g.name
ORDER BY
    yyyy DESC, g.name
```

In the above example, the query results are grouped by `yyyy, g.name`, and the ordering notation is similar. Today I learned that this can also be expressed using positional identifiers, like so:

```sql
SELECT
    extract(year FROM b.completed_at) AS yyyy,
    g.name,
    count(b.book_id) AS the_count
FROM
    book AS b
    INNER JOIN genre AS g ON b.genre_id = g.genre_id
GROUP BY
    1, 2
ORDER BY
    1 DESC, 2
```

The `1` and `2` in the `GROUP BY` indicate the column position in the `SELECT`. If you run [Example 1](https://www.db-fiddle.com/f/du2YmLpyqgaoqbBsFMeVnC/3) and [Example 2](https://www.db-fiddle.com/f/6sd9FZKSK7CSHpwY9L42aH/1), you'll see the results are the same.

For what it's worth, I think this is possible in most database engines, but I could be wrong about that. If I'm wrong, let me know!

## Takeaways

I'm not saying that this approach is preferable. In most cases it is not. First, positional indicators obscure what is being done in the query, and second, they are very brittle. That is, simple refactoring can easily result in unintended consequences, as any column order change can alter the grouping/ordering.

That said, it's helpful to know and I suppose there may be clever uses that I haven't considered yet, perhaps with dynamically generated queries.

Honestly, however, the biggest takeaway for me is that it's a convenient shorthand when I'm running ad-hoc, one-off queries. Recently, I wrote a query ended with this:

```sql
GROUP BY 1,2,3,4
ORDER BY 1,2,3,4
```

That's pretty succinct, if nothing else. The more you know!

## Example Database

For my own reference, and in the event the DB Fiddle links die one day, here's the code for the example database:

```sql
-- Schema (PostgreSQL v13)

CREATE TABLE genre (
  genre_id serial not null
    constraint genre_pk
      primary key,
  name varchar(255) not null
);

CREATE TABLE book (
    book_id serial not null
      constraint book_pk
      primary key,
    genre_id int not null
      constraint books_type_type_id_fk
      references genre,
    name varchar(255) not null,
    started_at date not null,
    completed_at date not null
);

INSERT INTO genre VALUES(DEFAULT, 'Classic');
INSERT INTO genre VALUES(DEFAULT, 'Humor');
INSERT INTO genre VALUES(DEFAULT, 'Drama');
INSERT INTO genre VALUES(DEFAULT, 'Biography');

INSERT INTO book VALUES (DEFAULT, 4, 'Pilgrim at Tinker Creek', '2019-08-20', '2019-12-24');
INSERT INTO book VALUES (DEFAULT, 1, 'David Copperfield', '2020-01-01', '2020-03-15');
INSERT INTO book VALUES (DEFAULT, 1, 'A Tale of Two Cities', '2020-03-16', '2020-08-17');
INSERT INTO book VALUES (DEFAULT, 1, 'The Scarlet Letter', '2020-08-18', '2020-12-31');
INSERT INTO book VALUES (DEFAULT, 3, 'Long Day''s Journey into Night', '2020-11-02', '2020-11-28');
INSERT INTO book VALUES (DEFAULT, 2, 'We are in a Book!', '2021-01-01', '2021-01-01');
INSERT INTO book VALUES (DEFAULT, 2, 'A Confederacy of Dunces', '2021-01-08', '2021-09-16');
INSERT INTO book VALUES (DEFAULT, 1, 'Pride and Prejudice', '2021-04-11', '2021-10-17');
INSERT INTO book VALUES (DEFAULT, 1, 'Beloved', '2021-10-18', '2022-01-02');
```
