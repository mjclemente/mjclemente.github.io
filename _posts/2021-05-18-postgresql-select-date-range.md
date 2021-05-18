---
published: true
title: "TIL: The Easiest Way to Select the Last 30 Days (or Any Interval) in PostgreSQL"
layout: post
tags: [sql,til]
---
Thanks to my ignorance, PostgreSQL is an ongoing source of TILs. Today, I learned about using `interval` to easily select a range of time.
<!--more-->

While reviewing data from a logging table, I needed to select records from the past 30 days. On a whim, I decided to see if PostgreSQL provided any clever ways to do this. My searches lead me to learn about a new data type: `interval`.[^1] Here's a link to [the docs](https://www.postgresql.org/docs/current/datatype-datetime.html#DATATYPE-INTERVAL-INPUT), as well as [the post](https://mode.com/blog/postgres-sql-date-functions/#Finding%20events%20relative%20to%20the%20present%20time%20with%20NOW()%20and%20CURRENT_DATE) I stumbled upon that demonstrated how to write this type of query. Let's take a closer look.

## Previous Approach Using the Specific Date

My typical approach for selecting data based on a time interval is first to calculate the specific date/time implied by the interval and then query based on whether records are from before or after that date.[^2] So, to select data from within the past 30 days, I'd determine that 30 days ago was April 18, and the I'd write the query something like this:

```sql
-- Example 1
SELECT
    *
FROM
    book
WHERE
    completed_at > '04/18/2021 00:00:00'
```

## New Approach Using `interval`

What I learned today was that this can also be written using an `interval`. Here's what it looks like:

```sql
-- Example 2
SELECT
    *
FROM
    book
WHERE
    completed_at > now() - interval '30 day'
```

The code `now() - interval '30 day'` automatically calculates the date 30 days in the past. My response to learning this was 🤯! Yes, it merited my first use of an emoji in a sentence. That's so cool!

The available units for the `interval` data type are: `microsecond`, `millisecond`, `second`, `minute`, `hour`, `day`, `week`, `month`, `year`, `decade`, `century`, `millennium`, all of which can be abbreviated or written as plurals.

Now, there's apparently a lot more that's possible with `interval`, most of which I obviously don't know. But here are two more features I found: 

* You can combine units expressively. So, for example, you can declare a very specific time interval like this:

  ```sql
  WHERE completed_at > now() - interval '1 year 137 days 12 hours'
  ```

	The combination of accuracy (down the the microsecond) and natural language syntax is really powerful. I don't have a use case for this yet, but I was impressed by it.

* Within `INSERT` statements, you can combine `interval` and `now()` to generate timestamps at a specifically timed interval in the past or future. I did this when creating the second example table:

  ```sql
  INSERT INTO book
      VALUES (DEFAULT, 1, 'We Have Always Lived in the Castle', now() - interval '28 day', now());
  ```

  Inserting future timestamps could be useful when you want to schedule a later event based on the current time, like a welcome email being sent one day after a new user signs up.

That's it for today!

## Examples

[Run Example 1](https://www.db-fiddle.com/f/7Saft5dUeJmEEQkDU2YvNT/7).

[Run Example 2](https://www.db-fiddle.com/f/m1HjF2doC5GuA5a1WoSAss/5).

For my own reference, and in the event the DB Fiddle links die one day, here's the code for the Example 2 database:

```sql
-- Schema (PostgreSQL v13)
CREATE TABLE genre (
    genre_id serial NOT NULL CONSTRAINT genre_pk PRIMARY KEY,
    name varchar(255) NOT NULL
);

CREATE TABLE book (
    book_id serial NOT NULL CONSTRAINT book_pk PRIMARY KEY,
    genre_id int NOT NULL CONSTRAINT books_type_type_id_fk REFERENCES genre,
    name varchar(255) NOT NULL,
    started_at date NOT NULL,
    completed_at date NOT NULL
);

INSERT INTO genre VALUES (DEFAULT, 'Classic');
INSERT INTO genre VALUES (DEFAULT, 'Humor');
INSERT INTO genre VALUES (DEFAULT, 'Drama');
INSERT INTO genre VALUES (DEFAULT, 'Biography');
INSERT INTO genre VALUES (DEFAULT, 'History');

INSERT INTO book
    VALUES (DEFAULT, 4, 'Pilgrim at Tinker Creek', now() - interval '1001 day', now() - interval '875 day');
INSERT INTO book
    VALUES (DEFAULT, 1, 'David Copperfield', now() - interval '867 day', now() - interval '794 day');
INSERT INTO book
    VALUES (DEFAULT, 1, 'A Tale of Two Cities', now() - interval '793 day', now() - interval '639 day');
INSERT INTO book
    VALUES (DEFAULT, 1, 'The Scarlet Letter', now() - interval '638 day', now() - interval '503 day');
INSERT INTO book
    VALUES (DEFAULT, 3, 'Long Day''s Journey into Night', now() - interval '562 day', now() - interval '536 day');
INSERT INTO book
    VALUES (DEFAULT, 2, 'We are in a Book!', now() - interval '867 day', now() - interval '502 day');
INSERT INTO book
    VALUES (DEFAULT, 2, 'A Confederacy of Dunces', now() - interval '495 day', now() - interval '243 day');
INSERT INTO book
    VALUES (DEFAULT, 1, 'Pride and Prejudice', now() - interval '401 day', now() - interval '212 day');
INSERT INTO book
    VALUES (DEFAULT, 1, 'Beloved', now() - interval '211 day', now() - interval '135 day');
INSERT INTO book
    VALUES (DEFAULT, 5, '1776', now() - interval '134 day', now() - interval '78 day');
INSERT INTO book
    VALUES (DEFAULT, 1, 'Perelandra', now() - interval '80 day', now() - interval '67 day');
INSERT INTO book
    VALUES (DEFAULT, 4, 'The World''s Largest Man', now() - interval '66 day', now() - interval '28 day');
INSERT INTO book
    VALUES (DEFAULT, 1, 'We Have Always Lived in the Castle', now() - interval '28 day', now());
```

____

[^1]: New, as in "new to me". The docs show it being a part of PostgreSQL going back to at least version 7.1.
[^2]: In the interest of accuracy, I suppose I should say that typically it's my application that's actually doing the calculating, but the point is that the query is based on the date.