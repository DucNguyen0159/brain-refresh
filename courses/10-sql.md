# SQL, MySQL, PostgreSQL, and Supabase

Course 10 of 15. Previous: [09-flask.md](09-flask.md). Next: [11-firebase-firestore.md](11-firebase-firestore.md).

SQL is the language for relational tables. MySQL and PostgreSQL are two servers that speak it. Supabase is hosted PostgreSQL plus login and row-level policies. Learn the language before the product.

## Contents

1. [Tables](#1-tables)
2. [Select, filter, and sort](#2-select-filter-and-sort)
3. [Insert, update, and delete](#3-insert-update-and-delete)
4. [Aggregates and groups](#4-aggregates-and-groups)
5. [Joins](#5-joins)
6. [Union and row numbers](#6-union-and-row-numbers)
7. [Constraints, indexes, and transactions](#7-constraints-indexes-and-transactions)
8. [MySQL and PostgreSQL](#8-mysql-and-postgresql)
9. [Supabase](#9-supabase)

## 1. Tables

A table has named columns and many rows. A column has a type. A row is one record.

```sql
CREATE TABLE homes (
  id          integer PRIMARY KEY,
  city        text    NOT NULL,
  price       integer NOT NULL,
  beds        integer,
  listed_on   date
);
```

| Word | Meaning |
| --- | --- |
| Row | One record |
| Column | One field |
| `PRIMARY KEY` | The value that identifies the row. Unique, not null |
| `NOT NULL` | The column must have a value |
| `NULL` | Unknown or absent. Not the same as `0` or `''` |
| Schema | A namespace of tables. PostgreSQL uses them directly. MySQL often uses "database" for the same idea |
| Foreign key | A column that points at a primary key in another table |

| Type you will meet | Holds |
| --- | --- |
| `integer` / `int` | Whole numbers |
| `numeric(12, 2)` or `decimal` | Exact decimals. Money prefers these over floating point |
| `text` / `varchar(n)` | Text. `varchar` has a limit |
| `boolean` | True or false |
| `date` | A calendar date |
| `timestamp` | A date and time |

## 2. Select, filter, and sort

```sql
SELECT id, city, price
FROM homes
WHERE city = 'Irvine'
  AND price <= 1500000
  AND beds >= 3
ORDER BY price ASC
LIMIT 10 OFFSET 0;
```

| Clause | Job |
| --- | --- |
| `SELECT` | Which columns. `*` means all of them |
| `DISTINCT` | Drop duplicate result rows |
| `FROM` | Which table |
| `WHERE` | Keep rows that pass. Runs before grouping |
| `ORDER BY` | Sort. `ASC` is the default. `DESC` is high to low |
| `LIMIT` | How many rows |
| `OFFSET` | How many to skip. Page 2 of size 10 is `LIMIT 10 OFFSET 10` |
| `AS` | A result name: `price AS list_price` |

| Condition | Meaning |
| --- | --- |
| `= <> < > <= >=` | Comparisons. `<>` is "not equal" |
| `AND` / `OR` / `NOT` | Combine tests. `AND` binds tighter than `OR`. Use parentheses |
| `IN (1, 2, 3)` | Equal to one of the list |
| `BETWEEN 1 AND 10` | Inclusive on both ends |
| `LIKE 'Irv%'` | `%` is any text. `_` is one character |
| `IS NULL` / `IS NOT NULL` | The only correct null tests. `= NULL` is not true |
| `city ILIKE 'irvine'` | Case-insensitive `LIKE`. PostgreSQL |

`NULL` fails ordinary comparisons. `NULL = NULL` is not true. `WHERE beds >= 3` drops rows whose `beds` is null.

`ORDER BY 2` sorts by the second selected column. Prefer the column name.

## 3. Insert, update, and delete

```sql
INSERT INTO homes (id, city, price, beds)
VALUES (1, 'Irvine', 999000, 3);

UPDATE homes
SET price = 989000
WHERE id = 1;

DELETE FROM homes
WHERE id = 1;
```

| Statement | Danger |
| --- | --- |
| `UPDATE` without `WHERE` | Changes every row |
| `DELETE` without `WHERE` | Deletes every row |
| `INSERT` omitting a `NOT NULL` column with no default | Fails |
| Several rows | `INSERT INTO homes (id, city, price) VALUES (2, 'Tustin', 800000), (3, 'Irvine', 700000);` |

`RETURNING *` is PostgreSQL. It gives back the row you just wrote. MySQL uses a separate `SELECT`.

## 4. Aggregates and groups

```sql
SELECT city,
       COUNT(*) AS homes,
       ROUND(AVG(price), 0) AS avg_price,
       MIN(price) AS cheapest
FROM homes
WHERE price > 0
GROUP BY city
HAVING COUNT(*) >= 5
ORDER BY homes DESC;
```

| Function | Result |
| --- | --- |
| `COUNT(*)` | Rows, including nulls |
| `COUNT(beds)` | Rows where `beds` is not null |
| `SUM` / `AVG` / `MIN` / `MAX` | Over the group. `AVG` ignores nulls |
| `ROUND(n, 0)` | Rounded number |

Every selected column is either inside an aggregate or listed in `GROUP BY`.

| Clause | When it filters |
| --- | --- |
| `WHERE` | Rows, before the group |
| `HAVING` | Groups, after the aggregate |

`NULLIF(sqft, 0)` turns `0` into null so a later division does not divide by zero. `AVG` then skips those rows.

`CASE` is an expression:

```sql
SELECT id,
       CASE
         WHEN price < 500000 THEN 'low'
         WHEN price < 1000000 THEN 'mid'
         ELSE 'high'
       END AS band
FROM homes;
```

## 5. Joins

A join matches rows from two tables.

```sql
SELECT h.city, a.name
FROM homes AS h
JOIN agents AS a ON a.id = h.agent_id
WHERE h.city = 'Irvine';
```

| Join | Keeps |
| --- | --- |
| `JOIN` or `INNER JOIN` | Only rows that match on both sides |
| `LEFT JOIN` | Every left row. Right columns are null when nothing matches |
| `RIGHT JOIN` | Every right row. You can usually rewrite it as a `LEFT JOIN` by swapping the tables |
| `FULL OUTER JOIN` | Every row from both. PostgreSQL has it. MySQL does not, in the same form |
| `CROSS JOIN` | Every combination. Rare on purpose |

A left join plus `WHERE right.id IS NULL` finds left rows with no match.

A subquery is a query inside a query:

```sql
SELECT city, price
FROM homes
WHERE price > (SELECT AVG(price) FROM homes);
```

`IN (SELECT id FROM ...)` and `EXISTS (SELECT 1 FROM ... WHERE ...)` are the other common shapes. `EXISTS` stops at the first match.

## 6. Union and row numbers

`UNION` stacks the rows of two queries. The queries must return the same number of columns, in the same order, with compatible types.

```sql
SELECT city, price FROM homes
UNION
SELECT city, price FROM sold_homes;
```

| | `UNION` | `UNION ALL` |
| --- | --- | --- |
| Duplicate rows | Removed | Kept |
| Extra work | A pass to find the duplicates | None. The second result is appended |
| Use it when | The combined list should not repeat a row | You already know the rows are distinct, or you want every row |

Column names come from the first query. An `ORDER BY` at the end sorts the combined result.

`ROW_NUMBER()` numbers rows without collapsing them the way `GROUP BY` does. Current MySQL and PostgreSQL both have it.

```sql
SELECT city, price
FROM (
  SELECT city,
         price,
         ROW_NUMBER() OVER (PARTITION BY city ORDER BY price) AS n
  FROM homes
) ranked
WHERE n = 1;
```

| Piece | Meaning |
| --- | --- |
| `OVER (...)` | The window. The function looks across these rows |
| `PARTITION BY city` | Start the count over for each city. Omit it and the count runs over the whole result |
| `ORDER BY price` | Number 1 is the lowest price in that city |
| `WHERE n = 1` | Keep the first row of each city. The filter has to sit outside, because `WHERE` cannot see a window result from the same `SELECT` |

## 7. Constraints, indexes, and transactions

```sql
CREATE TABLE offers (
  id       integer PRIMARY KEY,
  home_id  integer NOT NULL REFERENCES homes (id),
  amount   integer NOT NULL CHECK (amount > 0),
  buyer    text    NOT NULL,
  UNIQUE (home_id, buyer)
);

CREATE INDEX offers_home_id ON offers (home_id);
```

| Constraint | Enforces |
| --- | --- |
| `PRIMARY KEY` | Unique and not null. One per table |
| `UNIQUE` | No duplicate values in those columns |
| `NOT NULL` | A value is required |
| `CHECK` | A boolean expression |
| `REFERENCES` | The foreign key must exist in the other table |
| `ON DELETE CASCADE` | Deleting the parent deletes the children. Dangerous and sometimes what you want |

An index speeds a lookup and slows writes a little. Index columns you filter or join on. A leading wildcard `LIKE '%Irvine'` cannot use a normal index.

A transaction is a group of statements that succeed or fail together.

```sql
BEGIN;
UPDATE homes SET price = price - 10000 WHERE id = 1;
INSERT INTO price_changes (home_id, amount) VALUES (1, -10000);
COMMIT;
```

`ROLLBACK` undoes the transaction. If the second statement fails and you roll back, the first update does not remain.

Isolation levels exist so two transactions do not read each other's half-finished work. The default on both servers is enough until you are debugging a specific race.

## 8. MySQL and PostgreSQL

The statements above are the shared language. These are the differences you hit in practice.

| Topic | MySQL | PostgreSQL |
| --- | --- | --- |
| Auto id | `id integer PRIMARY KEY AUTO_INCREMENT` | `id integer PRIMARY KEY GENERATED ALWAYS AS IDENTITY` |
| Case-insensitive pattern | Often the collation, or `LOWER(city) = LOWER(?)` | `ILIKE` |
| Boolean | `true` / `false`, or tinyint in old tables | Real `boolean` |
| String quotes | Single quotes for text | Single quotes for text. Double quotes for identifiers |
| Limit | `LIMIT 10 OFFSET 0` | The same |
| Returning written rows | Not the same statement | `RETURNING *` |
| Placeholder in app drivers | Often `?` or `%s` | `$1`, `$2` in raw drivers |
| Upsert | `ON DUPLICATE KEY UPDATE` | `ON CONFLICT ... DO UPDATE` |
| JSON | `JSON` | `jsonb` is the one to store |
| Text | `VARCHAR(n)`, `TEXT` | `text`, or `varchar(n)` |

| Job | MySQL | PostgreSQL |
| --- | --- | --- |
| Glue strings | `CONCAT(city, ', ', state)` | `city \|\| ', ' \|\| state`, or `concat(city, ', ', state)` |
| First non-null | `IFNULL(a, b)`, or `COALESCE` | `COALESCE(a, b)` |
| Current timestamp | `NOW()` | `now()`, or `CURRENT_TIMESTAMP` |
| Format a timestamp | `DATE_FORMAT(sold_at, '%Y-%m-%d')` | `to_char(sold_at, 'YYYY-MM-DD')` |
| Parse a date | `STR_TO_DATE('2026-10-27', '%Y-%m-%d')` | `to_date('2026-10-27', 'YYYY-MM-DD')` |
| Join a group into text | `GROUP_CONCAT(city ORDER BY city SEPARATOR ', ')` | `string_agg(city, ', ' ORDER BY city)` |
| Integer division | `5 DIV 2` | `5 / 2` is already integer when both sides are integers |

Parameters are how application code sends values. The SQL text keeps a placeholder. The driver sends the value separately. Building SQL by gluing a user's sentence into the string lets that sentence change the statement.

```sql
SELECT id, city FROM homes WHERE city = ? AND price <= ?;
```

The application pushes `'Irvine'`, then `1500000`. The order matches the placeholders.

`EXPLAIN` shows how the server plans to run a query. You do not need to memorize plans. You need to know the command exists when a query is slow.

## 9. Supabase

Supabase gives you a PostgreSQL database, a login system, and an API. The anon key in a browser app is public. Row Level Security is what stops that key from reading every row.

```sql
alter table teams enable row level security;

create policy "own teams"
on teams
for all
using (auth.uid() = user_id)
with check (auth.uid() = user_id);
```

| Piece | Meaning |
| --- | --- |
| `enable row level security` | With no policies, the anon and authenticated roles see nothing |
| `using` | Which existing rows a read or update can see |
| `with check` | Which rows an insert or update may write |
| `auth.uid()` | The id of the signed-in user |
| `for all` | Select, insert, update, and delete. You can split them |

```javascript
const { data, error } = await supabase
  .from("teams")
  .select("id, name")
  .eq("user_id", userId);
```

| Client call | SQL idea |
| --- | --- |
| `.from("teams")` | The table |
| `.select("id, name")` | The columns |
| `.eq("city", "Irvine")` | `WHERE city = ` |
| `.lte("price", 1500000)` | `<=` |
| `.order("price")` | `ORDER BY` |
| `.limit(10)` | `LIMIT` |
| `.insert({ name })` | `INSERT` |
| `.update({ name }).eq("id", id)` | `UPDATE ... WHERE` |
| `.delete().eq("id", id)` | `DELETE ... WHERE` |

Check `error` on every call. A failed policy looks like an error or an empty result, depending on the operation. It does not throw a JavaScript exception by itself.

Auth methods you will call: `signUp`, `signInWithPassword`, `signOut`, `getSession`, `resetPasswordForEmail`. The session contains the user id that `auth.uid()` sees.

Deleting a user can delete their rows when the foreign key says `on delete cascade`.

Service-role keys bypass RLS. They belong on a server you control, never in a browser.

## Drill

1. Write a `SELECT` of city and price for one city, under a max price, ordered by price, ten rows.
2. Write a `GROUP BY city` that returns the count and the average price.
3. Say what `USING (auth.uid() = user_id)` allows.
4. Say what `UNION` drops that `UNION ALL` keeps, and which row `ROW_NUMBER()` marks as `1` inside a city.
