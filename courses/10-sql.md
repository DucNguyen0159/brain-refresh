# SQL, MySQL, PostgreSQL, and Supabase

Course 10 of 15. Previous: [09-flask.md](09-flask.md). Next: [11-firebase-firestore.md](11-firebase-firestore.md).

SQL is the language for relational tables. MySQL and PostgreSQL are two servers that speak it. Supabase is hosted PostgreSQL plus login and row-level policies. Learn the language before the product.

Every section has the same parts: the concept, the terms, the methods, several cases with what the query returns, common mistakes, a tip, and a practice with a solution. The shared examples use this table:

```sql
CREATE TABLE homes (
  id    integer PRIMARY KEY,
  city  text    NOT NULL,
  price integer NOT NULL,
  beds  integer
);
INSERT INTO homes (id, city, price, beds) VALUES
  (1, 'Irvine', 900000, 3),
  (2, 'Irvine', 1100000, 4),
  (3, 'Tustin', 800000, 2),
  (4, 'Tustin', 900000, NULL);
```

`NULL` is unknown. It is not `0` and not an empty string.

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

### Concept

A table has named columns and many rows. A column has a type. A row is one record. `PRIMARY KEY` identifies the row: unique and not null. `NOT NULL` means the column must have a value. A foreign key points at a primary key in another table. A schema is a namespace of tables. PostgreSQL uses schemas directly. MySQL often uses the word database for the same idea.

### Terms

| Word | Meaning |
| --- | --- |
| Row | One record |
| Column | One field |
| `PRIMARY KEY` | The value that identifies the row |
| `NOT NULL` | A value is required |
| `NULL` | Unknown or absent |
| Schema | A namespace of tables |
| Foreign key | A column that must match a key in another table |

### Methods

| Type | Holds |
| --- | --- |
| `integer` / `int` | Whole numbers |
| `numeric(12, 2)` or `decimal` | Exact decimals. Prefer these for money |
| `text` / `varchar(n)` | Text. `varchar` has a limit |
| `boolean` | True or false |
| `date` | A calendar date |
| `timestamp` | A date and a time |

### Examples

**Case: a table and four rows**

The `CREATE TABLE` and `INSERT` at the top of this course.

Result: four rows. Ids `1` through `4`. Row `4` has `beds` of `NULL`.

Why: `beds` has no `NOT NULL`, so a missing bed count is allowed. `city` and `price` are required.

**Case: a second row with the same id**

```sql
INSERT INTO homes (id, city, price, beds) VALUES (1, 'Anaheim', 1, 1);
```

Result: the insert fails. The primary key `1` already exists.

Why: a primary key is unique. The original Irvine row stays.

**Case: a null city**

```sql
INSERT INTO homes (id, city, price) VALUES (5, NULL, 1);
```

Result: the insert fails. `city` is `NOT NULL`.

Why: `NULL` is not a city. Omitting the column fails the same way unless the column has a default.

**Case: null is not zero**

```sql
SELECT id FROM homes WHERE beds IS NULL;
SELECT id FROM homes WHERE beds = 0;
```

Result: the first query returns id `4`. The second returns no rows.

Why: unknown is not the number zero. `beds = NULL` is also not the right test. Use `IS NULL`.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `NULL` stored to mean zero | Filters and averages skip it | Store `0` when you mean zero |
| Two rows with the same primary key | The second insert fails | Use a new id, or an identity column |
| Money as a float | Totals drift | `numeric` or integer cents |
| `varchar` used without a reason you can name | You hit the length limit later | `text`, unless the limit is real |

### Tip

Decide what missing means before you create the column. Unknown is `NULL`. Zero is `0`. Required is `NOT NULL`.

### Practice

Create `homes` as in the course header and insert one more row: id `5`, city `Anaheim`, price `700000`, beds `3`.

### Solution

```sql
INSERT INTO homes (id, city, price, beds) VALUES (5, 'Anaheim', 700000, 3);
```

Result: the table has five rows. Selecting id `5` returns Anaheim.

## 2. Select, filter, and sort

### Concept

`SELECT` chooses columns. `FROM` names the table. `WHERE` keeps rows and runs before grouping. `ORDER BY` sorts. `LIMIT` and `OFFSET` page. `NULL` fails ordinary comparisons, so `WHERE beds >= 3` drops the row whose beds are null. `NULL = NULL` is not true. `AND` binds tighter than `OR`. Parentheses make the grouping obvious.

### Terms

| Term | Meaning |
| --- | --- |
| Predicate | A condition that is true, false, or unknown |
| `DISTINCT` | Drop duplicate result rows |
| `ASC` / `DESC` | Low to high, or high to low. `ASC` is the default |
| `OFFSET` | How many rows to skip |
| `ILIKE` | Case-insensitive `LIKE`. PostgreSQL |

### Methods

| Clause or test | Job |
| --- | --- |
| `SELECT id, city` | Those columns. `*` is all of them |
| `WHERE` | Keep rows that pass |
| `ORDER BY price ASC` | Sort. Prefer the column name over `ORDER BY 2` |
| `LIMIT 10 OFFSET 10` | Page 2 when the page size is 10 |
| `AS list_price` | A name for the result column |
| `IN (1, 2, 3)` | Equal to one of the list |
| `BETWEEN 1 AND 10` | Inclusive on both ends |
| `LIKE 'Irv%'` | `%` is any text. `_` is one character |
| `IS NULL` | The null test |

### Examples

**Case: filter and sort**

```sql
SELECT id, city, price
FROM homes
WHERE city = 'Irvine' AND price <= 1500000 AND beds >= 3
ORDER BY price ASC;
```

Result: id `1` at `900000`, then id `2` at `1100000`.

Why: both Irvine rows have beds of at least 3 and a price under the cap. `ASC` puts the lower price first.

**Case: null beds drop out of `>=`**

```sql
SELECT id FROM homes WHERE beds >= 3;
SELECT id FROM homes WHERE beds IS NULL;
```

Result: the first query returns ids `1` and `2`. It does not return id `4`. The second returns id `4`.

Why: a comparison with `NULL` is unknown, and `WHERE` keeps only true rows.

**Case: `AND` binds tighter than `OR`**

```sql
SELECT id FROM homes
WHERE city = 'Irvine' OR city = 'Tustin' AND price < 850000;
```

Result: ids `1`, `2`, and `3`.

Why: this is `city = 'Irvine' OR (city = 'Tustin' AND price < 850000)`. Irvine is kept at any price. Only the cheaper Tustin row, id `3`, is added. Parentheses are how you say a different grouping.

**Case: `LIKE` and a page**

```sql
SELECT city FROM homes WHERE city LIKE 'Irv%' ORDER BY id LIMIT 1 OFFSET 1;
```

Result: one row, `Irvine`, which is the second Irvine row (id `2`).

Why: `%` matches the rest of the word. `OFFSET 1` skips the first match. `LIKE 'irv%'` does not match `Irvine` on a case-sensitive collation. PostgreSQL `ILIKE 'irv%'` does.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `WHERE beds = NULL` | No rows | `IS NULL` |
| `OR` mixed with `AND` and no parentheses | The `AND` applies only to its neighbor | Parentheses |
| `ORDER BY 2` after you reorder the select list | You sort a different column | Name the column |
| `LIKE '%Irvine'` expected to use an index | A leading `%` cannot use a plain index | `Irvine%` when the prefix is what you know |

### Tip

Read `WHERE` as "keep this row only if the test is true." Unknown is not true.

### Practice

Select city and price for `Tustin` under `900000` inclusive, ordered by price, one row.

### Solution

```sql
SELECT city, price FROM homes
WHERE city = 'Tustin' AND price <= 900000
ORDER BY price
LIMIT 1;
```

Result: `Tustin`, `800000`. The `900000` Tustin row is second and the limit drops it.

## 3. Insert, update, and delete

### Concept

`INSERT` adds rows. `UPDATE` changes rows. `DELETE` removes rows. Without `WHERE`, `UPDATE` and `DELETE` affect every row. An insert that omits a `NOT NULL` column with no default fails. `RETURNING *` is PostgreSQL and gives back the row you just wrote. MySQL uses a separate `SELECT`.

### Terms

| Term | Meaning |
| --- | --- |
| `INSERT` | Add rows |
| `UPDATE` | Change existing rows |
| `DELETE` | Remove rows |
| `WHERE` on a write | Which rows. Omit it and you mean all of them |
| `RETURNING` | PostgreSQL sends the written row back |

### Methods

| Statement | Effect |
| --- | --- |
| `INSERT INTO homes (id, city, price, beds) VALUES (5, 'Anaheim', 700000, 3)` | One row |
| Two tuples in `VALUES` | Two rows in one statement |
| `UPDATE homes SET price = 1 WHERE id = 1` | One row's price |
| `DELETE FROM homes WHERE id = 1` | That row is gone |
| `RETURNING *` | PostgreSQL only, on the write |

### Examples

**Case: insert one row**

```sql
INSERT INTO homes (id, city, price, beds) VALUES (5, 'Anaheim', 700000, 3);
SELECT city, price FROM homes WHERE id = 5;
```

Result: `Anaheim`, `700000`.

Why: the column list and the value list line up by position.

**Case: update one price**

```sql
UPDATE homes SET price = 989000 WHERE id = 1;
SELECT price FROM homes WHERE id = 1;
```

Result: `989000`. The other prices stay.

Why: `WHERE id = 1` limits the write. `SET` names the new value.

**Case: update with no `WHERE`**

```sql
UPDATE homes SET price = 1;
```

Result: every row's price becomes `1`, including Tustin and both Irvine rows.

Why: there is no filter. The statement is valid. It is almost never what you want on a table you care about.

**Case: delete one row, then a missing insert**

```sql
DELETE FROM homes WHERE id = 5;
INSERT INTO homes (id, city) VALUES (6, 'Anaheim');
```

Result: id `5` is gone if it existed. The insert of id `6` fails because `price` is `NOT NULL` and has no default.

Why: `DELETE` without a match deletes zero rows and does not fail. `INSERT` that skips a required column fails.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `UPDATE` or `DELETE` with no `WHERE` | Every row changes or disappears | `WHERE` on the key. Begin a transaction if you are unsure |
| Column and value lists of different lengths | The statement fails | One value per named column |
| `RETURNING` on MySQL | Syntax error | A follow-up `SELECT` |
| Deleting by city when several rows share it | All of those rows go | Delete by primary key |

### Tip

Write the `WHERE` before you write `SET` or `DELETE`. Then read it once as "how many rows should this touch?"

### Practice

Set the price of id `3` to `810000` and select that price.

### Solution

```sql
UPDATE homes SET price = 810000 WHERE id = 3;
SELECT price FROM homes WHERE id = 3;
```

Result: `810000`.

## 4. Aggregates and groups

### Concept

An aggregate folds many rows into one value. `GROUP BY` makes one result row per group. Every selected column is either inside an aggregate or listed in `GROUP BY`. `WHERE` filters rows before the group. `HAVING` filters groups after the aggregate. `COUNT(*)` counts rows, including nulls. `COUNT(beds)` skips null beds. `AVG` ignores nulls. `NULLIF(sqft, 0)` turns `0` into null so a later division does not divide by zero.

### Terms

| Term | Meaning |
| --- | --- |
| Aggregate | `COUNT`, `SUM`, `AVG`, `MIN`, `MAX` |
| Group | Rows that share a key |
| `WHERE` | Row filter, before grouping |
| `HAVING` | Group filter, after the aggregate |
| `CASE` | An expression that picks a value |

### Methods

| Function | Result |
| --- | --- |
| `COUNT(*)` | Rows in the group |
| `COUNT(beds)` | Rows whose `beds` is not null |
| `SUM` / `AVG` / `MIN` / `MAX` | Over the group. `AVG` skips nulls |
| `ROUND(n, 0)` | Rounded number |
| `NULLIF(a, b)` | `NULL` when `a` equals `b`, otherwise `a` |

### Examples

**Case: count and average by city**

```sql
SELECT city, COUNT(*) AS homes, ROUND(AVG(price), 0) AS avg_price
FROM homes
GROUP BY city
ORDER BY city;
```

Result: Irvine, `2` homes, average `1000000`. Tustin, `2` homes, average `850000`.

Why: each city is one group. `AVG` uses both prices in the group. `ROUND(..., 0)` drops the fraction.

**Case: `COUNT(*)` versus `COUNT(beds)`**

```sql
SELECT city, COUNT(*) AS rows, COUNT(beds) AS known_beds
FROM homes
GROUP BY city
ORDER BY city;
```

Result: Irvine is `2` and `2`. Tustin is `2` and `1`, because one Tustin bed count is null.

Why: `COUNT(*)` counts the row. `COUNT(beds)` skips null.

**Case: `WHERE` versus `HAVING`**

```sql
SELECT city, COUNT(*) AS homes
FROM homes
WHERE price > 0
GROUP BY city
HAVING COUNT(*) >= 2
ORDER BY homes DESC;
```

Result: both cities, each with `2`, if every sample price is positive. A city with one row would disappear in `HAVING`, even though `WHERE` kept that row.

Why: `WHERE` drops rows first. `HAVING` drops groups. `HAVING price > 0` is the wrong place for a row test.

**Case: `CASE` labels a row**

```sql
SELECT id,
       CASE
         WHEN price < 500000 THEN 'low'
         WHEN price < 1000000 THEN 'mid'
         ELSE 'high'
       END AS band
FROM homes
WHERE id = 3;
```

Result: id `3`, band `mid`, because `800000` is not under `500000` and is under `1000000`.

Why: `CASE` is an expression. It does not filter. The first true `WHEN` wins.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Selecting `city` without `GROUP BY city` or an aggregate | The query is rejected | Group by the column or aggregate it |
| `WHERE COUNT(*) >= 2` | The query is rejected. The aggregate does not exist yet | `HAVING` |
| `AVG(beds)` expected to treat null as zero | Those rows are skipped | `AVG(COALESCE(beds, 0))` when zero is what you mean |
| Dividing by a `sqft` of `0` | The division fails or becomes infinite | `NULLIF(sqft, 0)` |

### Tip

`WHERE` sees rows. `HAVING` sees the numbers you just computed.

### Practice

Return each city and its cheapest price, ordered by city.

### Solution

```sql
SELECT city, MIN(price) AS cheapest
FROM homes
GROUP BY city
ORDER BY city;
```

Result: Irvine `900000`, Tustin `800000`.

## 5. Joins

### Concept

A join matches rows from two tables. `JOIN` and `INNER JOIN` keep only matches. `LEFT JOIN` keeps every left row and fills the right side with null when nothing matches. A left join plus `WHERE right.id IS NULL` finds left rows with no match. `FULL OUTER JOIN` exists in PostgreSQL and not in the same form in MySQL. A subquery is a query inside a query. `EXISTS` stops at the first match.

### Terms

| Term | Meaning |
| --- | --- |
| Inner join | Both sides match |
| Left join | All left rows |
| Match condition | The `ON` clause, not a filter you meant for one table |
| Unmatched | Right columns are null |
| `EXISTS` | True if the inner query finds a row |

### Methods

| Join | Keeps |
| --- | --- |
| `JOIN` / `INNER JOIN` | Matches only |
| `LEFT JOIN` | Every left row |
| `RIGHT JOIN` | Every right row. Usually rewrite it as `LEFT JOIN` |
| `FULL OUTER JOIN` | Both sides. PostgreSQL |
| `CROSS JOIN` | Every combination |

Add this table for the examples:

```sql
CREATE TABLE agents (id integer PRIMARY KEY, name text NOT NULL);
INSERT INTO agents (id, name) VALUES (10, 'Ada'), (11, 'Bea');
ALTER TABLE homes ADD COLUMN agent_id integer;
UPDATE homes SET agent_id = 10 WHERE id IN (1, 2);
```

Ids `3` and `4` have no agent.

### Examples

**Case: an inner join drops homes with no agent**

```sql
SELECT h.id, a.name
FROM homes AS h
JOIN agents AS a ON a.id = h.agent_id
ORDER BY h.id;
```

Result: ids `1` and `2`, both with `Ada`. Tustin does not appear.

Why: an inner join requires a match. Null `agent_id` matches nothing.

**Case: a left join keeps the unmatched homes**

```sql
SELECT h.id, a.name
FROM homes AS h
LEFT JOIN agents AS a ON a.id = h.agent_id
ORDER BY h.id;
```

Result: ids `1` and `2` with `Ada`. Ids `3` and `4` with a null name.

Why: the left table is `homes`. Missing agents become null. They are not dropped.

**Case: left rows with no match**

```sql
SELECT h.id
FROM homes AS h
LEFT JOIN agents AS a ON a.id = h.agent_id
WHERE a.id IS NULL
ORDER BY h.id;
```

Result: ids `3` and `4`.

Why: the `WHERE` runs after the join. A null agent id means the left row had no match. Putting `a.id IS NULL` in the `ON` clause instead would not filter the same way.

**Case: a subquery and `EXISTS`**

```sql
SELECT city, price FROM homes
WHERE price > (SELECT AVG(price) FROM homes);

SELECT id FROM homes AS h
WHERE EXISTS (
  SELECT 1 FROM agents AS a WHERE a.id = h.agent_id
);
```

Result: the average of `900000`, `1100000`, `800000`, and `900000` is `925000`. The first query returns the Irvine row at `1100000`. The second returns ids `1` and `2`.

Why: the average is one number, so the comparison is a row filter. `EXISTS` is true as soon as one agent matches. It does not multiply homes when several agents would match.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| A condition in `ON` that you meant as a filter of the left table | A left join still returns the left row, with nulls | Filter the left table in `WHERE` |
| `WHERE a.name = 'Ada'` after a left join | The unmatched rows disappear. It became an inner join | Put that test in `ON`, or accept the inner-join meaning |
| Expecting `FULL OUTER JOIN` in MySQL | Syntax error | `LEFT JOIN` plus a query for the other side |
| A join with no `ON` written as a comma | A cross product | `JOIN ... ON` |

### Tip

Say which table is allowed to have leftovers. That table is the left side of a `LEFT JOIN`.

### Practice

List home ids that have no agent, using the tables in this section.

### Solution

The third example.

Result: ids `3` and `4`.

## 6. Union and row numbers

### Concept

`UNION` stacks two queries and removes duplicate rows. `UNION ALL` keeps duplicates. The two queries must return the same number of columns, in the same order, with compatible types. Column names come from the first query. `ROW_NUMBER()` numbers rows without collapsing them. The filter on that number sits outside the query that computes it, because `WHERE` cannot see a window result from the same `SELECT`.

### Terms

| Term | Meaning |
| --- | --- |
| `UNION` | Stack and drop duplicate rows |
| `UNION ALL` | Stack and keep every row |
| Window | A set of rows a function looks across |
| `PARTITION BY` | Restart the numbering for each group |
| `n = 1` | The first row in that window order |

### Methods

| Piece | Meaning |
| --- | --- |
| `UNION` | Extra pass to remove duplicates |
| `UNION ALL` | Append. Use it when duplicates are impossible or wanted |
| `OVER (PARTITION BY city ORDER BY price)` | Number 1 is the lowest price in that city |
| `WHERE n = 1` | Outside the inner query |

### Examples

**Case: `UNION` drops a repeated row**

```sql
SELECT city, price FROM homes WHERE id = 1
UNION
SELECT city, price FROM homes WHERE id = 1;
```

Result: one row, Irvine `900000`.

Why: both sides returned the same city and price. `UNION` keeps one copy.

**Case: `UNION ALL` keeps both**

```sql
SELECT city FROM homes WHERE id = 1
UNION ALL
SELECT city FROM homes WHERE id = 1;
```

Result: two rows, both `Irvine`.

Why: `ALL` appends. It does not look for duplicates.

**Case: the column counts must match**

```sql
SELECT city, price FROM homes
UNION
SELECT city FROM homes;
```

Result: the query fails. The first side has two columns and the second has one.

Why: `UNION` lines the columns up by position, not by name.

**Case: the cheapest home in each city**

```sql
SELECT city, price
FROM (
  SELECT city, price,
         ROW_NUMBER() OVER (PARTITION BY city ORDER BY price) AS n
  FROM homes
) ranked
WHERE n = 1
ORDER BY city;
```

Result: Irvine `900000`, Tustin `800000`.

Why: numbering restarts for each city. The lowest price is `1` because of `ORDER BY price`. Both cities have a `900000`, and Irvine's `900000` is its cheapest, so it is kept. Tustin's `900000` is number `2` and dropped. `WHERE n = 1` cannot move into the inner `SELECT`.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| `UNION` when you needed every sale | Duplicate city/price pairs disappear | `UNION ALL` |
| Different column counts | The query fails | Same width, compatible types |
| `WHERE ROW_NUMBER() ... = 1` in the same select | The query fails | Wrap it and filter outside |
| `ORDER BY` only inside one side of a `UNION` | It does not sort the combined result on every server | `ORDER BY` after the union |

### Tip

`UNION` is a duplicate filter you pay for. Use `ALL` unless you can name the duplicate you want removed.

### Practice

Stack the city of id `1` with itself using `UNION ALL`, and say how many rows come back.

### Solution

```sql
SELECT city FROM homes WHERE id = 1
UNION ALL
SELECT city FROM homes WHERE id = 1;
```

Result: two rows.

## 7. Constraints, indexes, and transactions

### Concept

A constraint rejects a bad write. An index speeds a lookup and slows writes a little. Index columns you filter or join on. A leading wildcard `LIKE '%Irvine'` cannot use a normal index. A transaction is a group of statements that succeed or fail together. `ROLLBACK` undoes it. If the second statement fails and you roll back, the first update does not remain.

### Terms

| Term | Meaning |
| --- | --- |
| `PRIMARY KEY` | Unique and not null. One per table |
| `UNIQUE` | No duplicates in those columns |
| `CHECK` | A boolean expression on the row |
| `REFERENCES` | The foreign key must exist |
| `BEGIN` / `COMMIT` / `ROLLBACK` | Start, keep, or undo a transaction |

### Methods

| Clause | Effect |
| --- | --- |
| `REFERENCES homes (id)` | `home_id` must be an existing home |
| `CHECK (amount > 0)` | Rejects zero and negative amounts |
| `UNIQUE (home_id, buyer)` | One row per pair |
| `ON DELETE CASCADE` | Deleting the parent deletes the children |
| `CREATE INDEX offers_home_id ON offers (home_id)` | An index for that column |

### Examples

**Case: a check rejects zero**

```sql
CREATE TABLE offers (
  id      integer PRIMARY KEY,
  home_id integer NOT NULL REFERENCES homes (id),
  amount  integer NOT NULL CHECK (amount > 0),
  buyer   text    NOT NULL,
  UNIQUE (home_id, buyer)
);
INSERT INTO offers (id, home_id, amount, buyer) VALUES (1, 1, 0, 'Ada');
```

Result: the insert fails. `amount > 0` is false.

Why: `CHECK` runs on the write. The row is not stored.

**Case: a foreign key**

```sql
INSERT INTO offers (id, home_id, amount, buyer) VALUES (1, 99, 500000, 'Ada');
```

Result: the insert fails. Home `99` does not exist.

Why: `REFERENCES` looks up the parent. A home id of `1` would succeed.

**Case: the unique pair**

```sql
INSERT INTO offers (id, home_id, amount, buyer) VALUES (1, 1, 500000, 'Ada');
INSERT INTO offers (id, home_id, amount, buyer) VALUES (2, 1, 510000, 'Ada');
```

Result: the first insert succeeds. The second fails. The same buyer cannot offer twice on the same home. A different buyer on home `1` would succeed.

Why: the unique constraint is the pair, not `buyer` alone.

**Case: rollback undoes the first update**

```sql
BEGIN;
UPDATE homes SET price = price - 10000 WHERE id = 1;
INSERT INTO offers (id, home_id, amount, buyer) VALUES (1, 1, 0, 'Ada');
ROLLBACK;
```

Result: the offer insert fails the check. After `ROLLBACK`, home `1` still has its original price.

Why: both statements are one transaction. Rolling back drops the price change along with the failed insert. `COMMIT` is what would keep them, and you do not commit after a failed statement.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| An index expected to enforce uniqueness | Duplicates are still allowed | A `UNIQUE` constraint |
| `ON DELETE CASCADE` on a parent people delete | Child rows vanish | Cascade only when the children are meaningless alone |
| `COMMIT` after a statement you meant to undo | The change is kept | `ROLLBACK` |
| A leading-wildcard `LIKE` expected to use the index | The plan scans the table | Search by prefix, or a different index type |

### Tip

A constraint is a rule the server enforces on every client. An application check is only the client you remembered to write.

### Practice

Insert an offer of `500000` from `Bea` on home `1`, then say what a second offer from `Bea` on home `1` does.

### Solution

```sql
INSERT INTO offers (id, home_id, amount, buyer) VALUES (2, 1, 500000, 'Bea');
INSERT INTO offers (id, home_id, amount, buyer) VALUES (3, 1, 520000, 'Bea');
```

Result: the first insert succeeds. The second fails on `UNIQUE (home_id, buyer)`.

## 8. MySQL and PostgreSQL

### Concept

The earlier statements are the shared language. The differences show up in auto ids, case-insensitive patterns, booleans, upserts, JSON, and a few functions. Application code should send values as parameters. The SQL text keeps a placeholder. The driver sends the value separately. Gluing a user's sentence into the SQL string lets that sentence change the statement. `EXPLAIN` shows the plan when a query is slow.

### Terms

| Term | Meaning |
| --- | --- |
| Placeholder | `?` or `$1`. Not the user's text pasted into the statement |
| Upsert | Insert, or update when the key already exists |
| `EXPLAIN` | The server's plan. You do not memorize it |
| Identity | A generated primary key |

### Methods

| Topic | MySQL | PostgreSQL |
| --- | --- | --- |
| Auto id | `AUTO_INCREMENT` | `GENERATED ALWAYS AS IDENTITY` |
| Case-insensitive pattern | Collation, or `LOWER(city) = LOWER(?)` | `ILIKE` |
| Returning written rows | A separate `SELECT` | `RETURNING *` |
| Placeholder | Often `?` | `$1`, `$2` in raw drivers |
| Upsert | `ON DUPLICATE KEY UPDATE` | `ON CONFLICT ... DO UPDATE` |
| JSON | `JSON` | `jsonb` |
| Glue strings | `CONCAT(city, ', ', state)` | `city \|\| ', ' \|\| state` |
| First non-null | `IFNULL(a, b)` or `COALESCE` | `COALESCE(a, b)` |
| Group text | `GROUP_CONCAT` | `string_agg` |

### Examples

**Case: parameters stay out of the SQL text**

```sql
SELECT id, city FROM homes WHERE city = ? AND price <= ?;
```

Result: with the values `Irvine` and `1500000`, the rows are ids `1` and `2`.

Why: the driver binds those values. A city string that contains a quote cannot close the statement and add another clause.

**Case: string glue**

```sql
SELECT CONCAT(city, ' ', price) FROM homes WHERE id = 3;
```

Result: on MySQL the cell is `Tustin 800000`. PostgreSQL can use `city || ' ' || price` for the same text.

Why: `||` is not MySQL's default string glue unless a mode turns it on. `CONCAT` is the portable call in this course's MySQL examples.

**Case: the first non-null**

```sql
SELECT id, COALESCE(beds, 0) AS beds FROM homes WHERE id = 4;
```

Result: id `4`, beds `0`.

Why: the stored value is still null. `COALESCE` only changes this result. `IFNULL` is the MySQL-only spelling of the two-argument case.

**Case: an upsert shape**

PostgreSQL:

```sql
INSERT INTO homes (id, city, price, beds) VALUES (1, 'Irvine', 1, 3)
ON CONFLICT (id) DO UPDATE SET price = EXCLUDED.price;
```

MySQL:

```sql
INSERT INTO homes (id, city, price, beds) VALUES (1, 'Irvine', 1, 3)
ON DUPLICATE KEY UPDATE price = VALUES(price);
```

Result: if you ran either statement, home `1` would keep its city and its price would become `1`. Do not run it against data you meant to keep. The point is the syntax, not the price `1`.

Why: the key already exists, so the insert becomes an update. The conflict target is the primary key.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| Building SQL with the user's city pasted in | A quote can change the statement | A placeholder |
| `ILIKE` on MySQL | Syntax error, unless you added it | `LOWER` on both sides, or the collation |
| `RETURNING` on MySQL | Syntax error | Select the row afterward |
| `||` on MySQL | Often numeric, or a syntax error, not string glue | `CONCAT` |

### Tip

Write the shared `SELECT` first. Look up the dialect only for identity columns, upserts, and the functions in the table.

### Practice

Select id `4` with `COALESCE(beds, 0)` and state the beds value in the result.

### Solution

```sql
SELECT COALESCE(beds, 0) AS beds FROM homes WHERE id = 4;
```

Result: `0`. The table still stores null.

## 9. Supabase

### Concept

Supabase gives you PostgreSQL, login, and an API. The anon key in a browser app is public. Row Level Security is what stops that key from reading every row. With RLS enabled and no policies, the anon and authenticated roles see nothing. `using` decides which existing rows a read or update can see. `with check` decides which rows an insert or update may write. `auth.uid()` is the signed-in user's id. The service-role key bypasses RLS and belongs on a server, never in a browser.

### Terms

| Term | Meaning |
| --- | --- |
| RLS | Row Level Security |
| Policy | A rule for one operation |
| Anon key | Public. Safe only because policies exist |
| Service role | Bypasses RLS. Server only |
| `error` | The client returns it. A failed call does not throw by itself |

### Methods

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

Auth calls: `signUp`, `signInWithPassword`, `signOut`, `getSession`, `resetPasswordForEmail`. The session contains the user id that `auth.uid()` sees.

### Examples

**Case: enable RLS and add a policy**

```sql
alter table teams enable row level security;

create policy "own teams"
on teams
for all
using (auth.uid() = user_id)
with check (auth.uid() = user_id);
```

Result: a signed-in user can read and write rows whose `user_id` equals their id. Other users' rows are invisible. With RLS on and this policy missing, the same user would see nothing.

Why: `using` filters reads. `with check` rejects an insert that would create a row for someone else.

**Case: a select through the client**

```javascript
const { data, error } = await supabase
  .from("teams")
  .select("id, name")
  .eq("user_id", userId);
```

Result: `data` is the rows the policy allows, or an empty list. `error` is `null` on success.

Why: `.eq` is the filter. It is not a substitute for the policy. A user who changes `userId` in the browser still cannot read another user's rows if `using` checks `auth.uid()`.

**Case: a failed policy**

An insert whose `user_id` is not `auth.uid()`.

Result: `error` is set, or `data` is empty, depending on the operation. The promise does not reject just because the policy said no.

Why: you have to read `error` on every call. A bare `await` that only looks at `data` treats a refusal like an empty success.

**Case: the service role**

A server client created with the service-role key.

Result: the query can read every row, including rows the user's policy would hide.

Why: that key bypasses RLS. It does not go in a browser bundle. The anon key is the one that ships to the client.

### Common mistakes

| Mistake | What happens | Fix |
| --- | --- | --- |
| RLS off because the anon key "is secret" | The key is in the browser. Anyone can read the table | Enable RLS and write policies |
| Checking only `data` | A refused write looks empty | Read `error` |
| `using` without `with check` | A user may insert a row they cannot read, or write a row for someone else | Set both when the policy covers writes |
| Service-role key in the frontend | RLS is skipped for every visitor | Keep it on the server |

### Tip

Turn RLS on before you put the anon key in an app. An empty policy list means no rows, which is the safe failure.

### Practice

Say what `using (auth.uid() = user_id)` allows a signed-in user to see.

### Solution

Rows in that table whose `user_id` equals the signed-in user's id. Other rows are filtered out. `with check` is still required to stop them from writing a row with someone else's `user_id`.

## Drill

1. Write a `SELECT` of city and price for one city, under a max price, ordered by price, ten rows.
2. Write a `GROUP BY city` that returns the count and the average price.
3. Say what `USING (auth.uid() = user_id)` allows.
4. Say what `UNION` drops that `UNION ALL` keeps, and which row `ROW_NUMBER()` marks as `1` inside a city.

### Solution 1

```sql
SELECT city, price
FROM homes
WHERE city = 'Irvine' AND price <= 1500000
ORDER BY price
LIMIT 10;
```

Result: Irvine `900000`, then Irvine `1100000`.

### Solution 2

```sql
SELECT city, COUNT(*) AS homes, ROUND(AVG(price), 0) AS avg_price
FROM homes
GROUP BY city
ORDER BY city;
```

Result: Irvine `2` and `1000000`. Tustin `2` and `850000`.

### Solution 3

The signed-in user can see rows whose `user_id` is their `auth.uid()`. Section 9's practice is the same answer.

### Solution 4

`UNION` drops duplicate rows. `UNION ALL` keeps them. Inside a city, `ROW_NUMBER() ... ORDER BY price` marks the lowest price as `1`.
