{%hackmd theme-dark %}

# SC2207 SQL/Part II Notes

## Relational Algebra

* Selection: $\sigma_{A > 100} R_1$ --- select from relation $R_1$ where $A > 100$
* Projection $\Pi_{A, B( A+B \to Sum)} R_1$ --- selects columns $A, B$ from relation $R_1$, and creates a new column $Sum$ with the value $a + b$
* Union $R_1 \cup R_2$ --- union of sets of tuples
* Intersection $R_1 \cap R_2$ --- intersection of sets of tuples
* Difference $R_1 - R_2$
* Natural Join $R_1 \bowtie R_2$
* Theta Join $R_1 \bowtie_\text{ conditions} R_2$
* Cartesian Product $R_1 \times R_2$ --- same as `SELECT * FROM R_1, R_2`
* Assignment $T_1 := \sigma_{A > 100} R_1$ --- assigns a table variable
* Rename $\rho_{TableName(col1, col2)} R_1$ --- Renames table `R_1(c1, c2)` to `TableName` with columns `col1, col2`. Rho for rename.
* Distinct $\delta(R_1)$ --- Obtain only distinct tuples from $R_1$
* Group $\gamma_\text{ grp_id, MAX(val) \(\to\) max_val}$ --- Group by `grp_id`, and aggregate maximum of `val` in each group to column `max_val`.
* Aggregation only $\gamma_\text{ MAX(val) \(to\) max_val}$ --- Applies aggregation function `MAX` to `val` column without grouping.
* Left Outer Join: $R_1 \stackrel{\circ}{\bowtie}_{L\text{ condition}} R_2$
* Right Outer Join: $R_1 \stackrel{\circ}{\bowtie}_{R\text{ condition}} R_2$
* Full Outer Join: $R_1 \stackrel{\circ}{\bowtie}_\text{ condition} R_2$
* Division $R_1(A, B) \div R_2(B)$
    * Find all $A$ that $R_1$ all the $B$ in $R_2$
    * The result will have columns $\{A, B\} - \{B\}$, i.e., will contain column $A$
    * Each row in the result is an $A$ who appears together with all the $B$s in $R_2$, e.g. $Owns(Name, Product) \div AppleP(Product) = Results$:
![image](https://hackmd.io/_uploads/B16fUsAbA.png)
    Since Alice is the only person who owns both iPhone and iPad.

## Syntax Nomenclature

When specifying general syntax, the following conventions are used:

* `{a | b | c}` means either `a`, `b`, or `c` can be used.
* `[a]` means `a` is optional.
* `[a | b]` means either `a` or `b` can be used, or neither.
* `{[a] [,] [b] [,] [c]}` means any combination of `a`, `b`, or `c` can be used, separated by commas, and at least one of them must be present.
* `[[a]  [,] [b]]` means either `a`, `b`, or both (comma-separated), or neither can be present.
* `<desc>` is a placeholder describing what the identifier/object should be at this position.

## DDL (Data Definition Language)

### Tables

#### Create Table

```sql
CREATE TABLE table_name ( -- THIS USES PARENTHESES. This is NOT a curly brace!!
    <name> <type> [<constraints>],
    fixed_len_name CHAR(10), -- CHAR = fixed-length string of 10 characters
    string_name VARCHAR(255) NOT NULL, -- VARCHAR = variable-length string up to 255 characters.
                                       -- NOT NULL constraint
    int_name INT CHECK (int_name >= 0 AND int_name <= 100), -- CHECK logical conditions
    some_date DATE, -- format is yyyy-mm-dd
    dt_name DATETIME DEFAULT GETDATE(), -- default value given when not provided
                                        -- Format is yyyy-mm-dd hh:mm:ss
    float_name FLOAT,
    PRIMARY KEY (string_name, int_name), -- compound primary key
    FOREIGN KEY (string_name) REFERENCES other_table_name(string_name), -- foreign key
    CHECK (float_name >= int_name) -- CHECK constraint between multiple attributes

    -- Constraints can be given names for better error messages and so they can be
    -- referred to by name when dropping/modifying them is necessary
    CONSTRAINT my_constraint_name CHECK (int_name >= 0 AND int_name <= 100),
    CONSTRAINT my_fk_name FOREIGN KEY (string_name) REFERENCES other_table_name(string_name)
);
```

#### Drop Table

```sql
DROP TABLE table_name;
```

If the table may not exist, and you don't want an error if the table doesn't exist you can use:

```sql
DROP TABLE IF EXISTS table_name;
```

The `IF EXISTS` keyword is also used when dropping anything else (constraints, functions, views, etc...)

#### Alter Table

```sql
-- add new column
ALTER TABLE table_name
ADD column_name type [constraints];

-- drop column
ALTER TABLE table_name
DROP COLUMN column_name;

-- drop constraint
ALTER TABLE table_name
DROP CONSTRAINT constraint_name;

-- add constraint
ALTER TABLE table_name
ADD CONSTRAINT constraint_name CHECK (column_name > 0);
```

### Constraints

#### Primary Key

```sql
PRIMARY KEY (col1, col2, ...); -- multiple columns can be specified for compound primary keys
```

**Only one primary key** can be defined per table.

#### Unique (Candidate Key)

To enforce uniqueness of a column or a combination of columns:
```sql
UNIQUE (col1, col2, ...);
```

**Multiple** unique constraints can be defined per table.

#### Not Null

This constraint is usually used inline when declaring columns:
```sql
col_name type NOT NULL; -- enforce that the column cannot be NULL
```

#### Default

This constraint is usually used inline when declaring columns:
```sql
col_name type DEFAULT value; -- set a default value for the column
```

#### Foreign Key

```sql
FOREIGN KEY (col1, col2, ...) REFERENCES other_table_name (col1, col2, ...)
    [ON UPDATE {CASCADE | SET NULL}] -- what to do when the referenced key is updated
    [ON DELETE {CASCADE | SET NULL}] -- what to do when the referenced key is deleted
```

Let `FOREIGN KEY (id) REFERENCES B(id)` in table `A` in the following examples.

* **CASCADE** means that any action on `B` (update or delete) will be propagated down to `A`:
  * If `B.id` is updated, then `A.id` will change to the new value.
  * If a row is deleted from `B`, all rows in `A` that reference the deleted row in `B` will also be deleted.
* **SET NULL** means that the foreign key `A.id` will be set to `NULL` when the referenced key in `B` is updated or deleted.
  * If `B.id` is updated, then `A.id` will be set to `NULL`.
  * If a row is deleted from `B`, then `A.id` will be set to `NULL`.

#### Check

```sql
CHECK (<condition>); -- enforce a logical condition one or more columns.
```

### Assertions

:::warning
TSQL doesn't have this feature.
:::

Assertions are constraints that are not tied to a single table, and will automatically be checked whenever any table referenced in the assertion is modified.

General syntax:

```sql
CREATE ASSERTION <assertion_name> CHECK (<boolean_condition>);
```

**Example:**

```sql
CREATE ASSERTION table1_larger_than_table2
    CHECK (
        (SELECT COUNT(*) FROM table1) > (SELECT COUNT(*) FROM table2)
    )
```
The above assertion will ensure that the number of rows in `table1` is always greater than the number of rows in `table2`, and will automatically be checked every time either `table1` or `table2` is modified.

### Triggers

Triggers activate when a certain event occurs on a table.

General syntax:

```sql
CREATE TRIGGER <trigger_name>
{BEFORE | AFTER | INSTEAD OF} -- when/how is the trigger activated
{INSERT | UPDATE | DELETE [OF {[col1] [,] [col2] ...}]} -- which event(s) that triggers the trigger
            -- Can use `OF` to specify the exact columns in <table_name> to watch for changes.
ON <table_name> -- the table that the trigger is attached to
[
    REFERENCING -- optional clause(s) to refer to the new/old rows/tables in the trigger body

    -- if the trigger is an INSERT or UPDATE trigger, new_row_identifier will contain
    -- the newly added row after insertion/update
    [NEW ROW AS <new_row_identifier>] [,]

    -- if the trigger is an UPDATE or DELETE trigger, old_row_identifier will contain
    -- the existing row that was updated or deleted
    [OLD ROW AS <old_row_identifier>] [,]

    -- if the trigger is an INSERT/UPDATE trigger, <new_table_identifier> will contain
    -- the state of the table after the insert/update
    [NEW TABLE AS <new_table_identifier>] [,]

    -- if the trigger is a DELETE/UPDATE trigger, <old_table_identifier> will contain
    -- the state of the table before the delete/update
    [OLD TABLE AS <old_table_identifier>]
]
[FOR EACH {ROW | STATEMENT}] -- granularity of the trigger
[WHEN <condition>] -- the condition to check after the trigger event occurs,
                   -- and before the trigger body is executed
[BEGIN] -- BEGIN/END block is optional
    -- trigger body
[END];
```

#### Trigger activation time (`BEFORE`, `AFTER`, `INSTEAD OF`)

* **BEFORE**: The trigger is activated before the event occurs.
* **AFTER**: The trigger is activated after the event occurs.
* **INSTEAD OF**: The trigger is activated instead of the event occurring.

#### Trigger event (`INSERT`, `UPDATE`, `DELETE`)

* **INSERT**: The trigger is activated when a new row is inserted into the table.
* **UPDATE**: The trigger is activated when a row is updated.
* **DELETE**: The trigger is activated when a row is deleted.

#### Referencing clause

The `REFERENCING` clause is used to refer to the new and old rows and tables in the trigger body. The identifiers as specified in `new_row_identifier, old_row_identifier, new_table_identifier, old_table_identifier` can be used in the trigger body to refer to the new and old rows and tables.

* **New** rows and tables are applicable for **Insert** and **Update** triggers
* **Old** rows and tables are applicable for **Update** and **Delete** triggers.

#### Granularity (row or statement)

* **ROW**: The trigger is activated for each row that is affected by the event.
  * If 0 rows are affected, the trigger is not activated.
* **STATEMENT**: The trigger is activated once for the entire statement that caused the event.
  * If 0 rows are affected, the trigger **is still activated**!

#### When Condition

* The when condition is checked once the [trigger is activated](##Trigger-activation-time-BEFORE-AFTER-INSTEAD-OF)
* If the boolearn condition passes, the trigger block is executed.

#### Trigger Block

* The trigger block occurs after the trigger is activated, when the `WHEN` condition passes.
* The block should only contain `INSERT`/`UPDATE`/`DELETE` statements,

#### Mnemonic for memorizing

**T**ime **E**lapsed **R**eferencing **G**reat **C**orpus

* `CREATE TRIGGER trigger`
* **Time** --- `BEFORE | AFTER | INSTEAD OF`
* **E**vent --- `{INSERT | UPDATE | DELETE} [OF col1, col2, ...] ON table`
* **Referencing** --- `REFERENCING {NEW | OLD} {ROW | TABLE} AS ident`
* **Gr**anularity --- `FOR EACH {ROW | STATEMENT}`
* **Co**ndtion --- `WHEN boolean_condition`

### Indexes

Indexes are used to speed up queries on specific search key(s).

They are tree-structured and hash-based. Instead of looking up every row in a table, the database can use the index to find the row(s) that match the search key(s) quickly.

```sql
-- Create an Index
CREATE INDEX index_name ON table_name (col1, col2, ...);

-- Create an Index without duplicate values
CREATE UNIQUE INDEX index_name ON table_name (col1, col2, ...);

-- Drop Index
DROP INDEX index_name;
```

#### Cover of an Index

An index **covers** a query when all attributes in the `SELECT` and `WHERE` clauses of the query are indexed together as search keys. This is when the hashtable/binary tree search-ish optimizations of the index can take place.

#### Automatically Created Indexes

* **Primary Key**: A primary key constraint automatically creates a unique index on the primary key column(s), aka **primary/clustered index**.
* **Unique**: A unique constraint automatically creates a unique index on the unique column(s), aka **secondary/non-clustered index**


### Views

A view is a **lazily-evaluated**, **non-memoized** and **persistent** table expression that is defined by a query. It is re-evaluated every time it is referenced (unless `CREATE MATERIALIZED VIEW` is used).

```sql
CREATE VIEW view_name AS
    SELECT column1, column2, ... -- <select query as usual>

SELECT * FROM view_name ... -- perform queries on the view

DROP VIEW view_name; -- the view persists until explicitly dropped
```

If the view has to be referenced multiple times, it is better to create a materialized view, which is a view that is stored in memory or on disk, and is only re-evaluated when the underlying tables change. This way it is **memoized** and **lazily-evaluated**, increasing performance.


```sql
CREATE MATERIALIZED VIEW view_name AS
    SELECT column1, column2, ... -- <select query as usual>
```

### Temporary Views aka Common Table Expressions (CTEs)

If a view isn't required to persist, and re-evaluations are not necessary, a **Common Table Expression** can be used.

```sql
WITH cte_name AS (
    SELECT column1, column2, ... -- <select query as usual>
)
    SELECT * FROM cte_name ... -- perform queries referring to the CTE cte_name
```

The CTE only persists for the duration of the query, and is not stored in the database.

## DML (Data Manipulation Language/CRUD)

### Insert (Create)

#### Inserting values directly

```sql
INSERT INTO table_name (column1, column2, column3, ...)
VALUES
    (col1_1, col2_1, col3_1, ...),
    (col1_2, col2_2, col3_2, ...),
    ...;
```

#### Inserting from `SELECT` statement

```sql
INSERT INTO table_name (column1, column2, column3, ...)
SELECT column1, column2, column3, ...
FROM other_table_name
WHERE condition
-- etc.. (see Queries section)
```

### Queries (Read)

#### Basic Query Syntax

> Stars Flicker Joyfully Where Galaxies Hate Orbiting

Notice that the order of the clauses must be strictly followed:

1. `SELECT`
2. `FROM`
3. `[INNER] JOIN`, `{LEFT | RIGHT | FULL} [OUTER] JOIN`
4. `WHERE`
5. `GROUP BY`
6. `HAVING`
7. `ORDER BY`

```sql
SELECT DISTINCT TOP(3) -- DISTICT gives unique rows only, TOP(3) gives only the first 3 rows
    T.id,
    T.group_id,

    -- The following are aggregation functions for GROUP BY:
    SUM(T.val1) AS val1_sum, -- use AS to rename the column
    AVG(T.val2),
    MIN(T.val3),
    MAX(T.val4),
    COUNT(*) AS cnt1, -- count all rows (even rows with all NULL entries) per group
    COUNT(T.val5) AS cnt2, -- count non-NULL entries per group
FROM table1 AS T -- T is an alias for table1. You can also use `FROM table1 T` without `AS`
INNER JOIN table2 as T2 ON T2.id = T.id
LEFT OUTER JOIN table3 as T3 ON T3.id = T.id
RIGHT OUTER JOIN table4 as T4 ON T4.id = T.id
FULL OUTER JOIN table5 as T5 ON T5.id = T.id
WHERE T.id == ( -- scalar subquery can substitute a scalar value
    SELECT TOP(1) T6.val
    FROM (SELECT * FROM table2 WHERE [blah]) AS T6 -- subquery in FROM must have an alias
    WHERE <some condition>
    ORDER BY T6.something DESC, T6.other_thing ASC -- multiple orderings
)
GROUP BY T.id, T.group_id -- group by each unique combination of (id, group_id) tuple
HAVING
    SUM(T.val1) > 100 -- conditions on aggregated values must be in HAVING at the end
    AND
    AVG(T.val2) <> 50 -- <> means not equal to
ORDER BY T.group_id -- default order is ASC
```

:::info
`SELECT` can also be used to evaluate a scalar function to obtain the result from a function call directly, e.g:
```sql
SELECT 1 + 1 AS two;
SELECT DATENAME(YEAR, GETDATE()) as curr_year;
```
:::

#### FROM multiple tables: Cross Product


$$
    A \times B = \{\ (a, b) : a \in A, b \in B\ \}
$$

```sql
SELECT *
FROM
    table_1 AS A,
    table_2 AS B
```

The end result will have all the columns from `table_1` and `table_2` combined. Repeated rows or columns will be present in the result.

#### Joins (Bag Semantics)

**Bag Semantics** means that duplicates are allowed in the result. A **bag** is the result of a `SELECT` query, and operations that `JOIN` bags do not remove duplicates, and their columns/attributes/tuple dimensions need not be the same (unlike [Set Semantics](#set-semantics-union-intersect-except))

To use Bag Semantics syntax but with distinct rows removed (*a la* [sets](#set-semantics-union-intersect-except)), use `SELECT DISTINCT`.

##### Inner Join (Default Join)

In TSQL, `JOIN` and `INNER JOIN` is the same type of join. The default join is an inner join.

$$
    A \bowtie_\text{ condition} B = \{\ (a_1, \dots, a_n, b_1, \dots, b_n) : (a_1, \dots) \in A,\ (b, \dots) \in B, \text{ and condition is met}\ \}
$$

I.e., an inner join is the set of all tuples in the cross product of the two tables that satisfy the boolean condition. If either table does not have a matching tuple, the tuple is not included in the result.

```sql
SELECT *
FROM
    table_1 AS A
JOIN table_2 AS B
    ON A.id = B.id + 3 -- boolean condition can be anything u want
```

Alternatively, if the boolean condition is the trivial equality `A.col1 = B.col1 AND A.col2 = B.col2`, you can use `USING(col1, col2)` as a shorthand, which also removes duplicate columns in the result:

```sql
SELECT *
FROM
    table_1 AS A
JOIN table_2 AS B
    USING(col1, col2) -- similar to ON A.col1 = B.col1 AND A.col2 = B.col2, removes duplicate columns
```

The `USING` keyword can also be used for all other joins (except `NATURAL JOIN`).

:::warning
The `USING` clause is **NOT** supported in TSQL (Microsoft Transact-SQL), but for some weird reason, it is in the slides, even though the labs are in TSQL!? To be safe, don't use this syntax in the exams.
:::

##### Full (Outer) Join

$$
    A \stackrel{\circ}{\bowtie}_\text{ condition} B = \{\ (a, b) : a \in A \text{ or null } \land b \in B \text{ or null, and condition is met}\ \}
$$

The rows which have matching tuples on the join condition will be paired together, and any other rows that do not have matching conditions will be paired with `NULL` values in the other table's columns.

```sql
SELECT * FROM A
FULL OUTER JOIN B -- alternatively, just `FULL JOIN` will do in TSQL.
                  -- The slides use `FULL OUTER JOIN`, so play it safe
    ON A.id = B.id
```

##### Left (Outer) Join

$$
    A \stackrel{\circ}{\bowtie}_{L \text{ condition}} B = \{\ (a, b) : a \in A \text{ or null } \land b \in B \text{ or null, and condition is met}\ \}
$$

Same as inner join, plus any rows in $A$ that do not have a matching tuple in $B$ will be paired with `NULL` values in $B$'s columns. However, rows in $B$ that do not have a matching tuple in $A$ will not be included in the result.

```sql
SELECT * FROM A
LEFT OUTER JOIN B ON A.id = B.id
```

##### Right (Outer Join)

$$
    A \stackrel{\circ}{\bowtie}_{R \text{ condition}} B = \{\ (a, b) : a \in A \text{ or null } \land b \in B \text{ or null, and condition is met}\ \}
$$

Same as inner join, plus any rows in $B$ that do not have a matching tuple in $A$ will be paired with `NULL` values in $A$'s columns. However, rows in $A$ that do not have a matching tuple in $B$ will not be included in the result.

```sql
SELECT * FROM A
RIGHT OUTER JOIN B ON A.id = B.id
```

##### Natural Join (Not in TSQL)

$$
    A \bowtie B = \{\ (a, b) : a \in A, b \in B, \text{ for all matching col names, } a_{c_i} = b_{c_i}\}
$$

This will perform an inner join automatically on all columns that have the same name in both tables.

:::warning
This is not supported in TSQL. Why is this in the notes??
:::

#### Set Semantics (`UNION`, `INTERSECT`, `EXCEPT`)

**Set Semantics** means that duplicates are not allowed in the result. A **set** is the result of a `SELECT` query, and operations that `UNION`, `INTERSECT`, or `EXCEPT` bags remove duplicates, and their columns/attributes/tuple dimensions must be exactly the same (unlike [Bag Semantics](#joins-bag-semantics))

To use Bag Semantics syntax but with duplicates allowed, attach `ALL` after the keywords, i.e.: `UNION ALL`, `INTERSECT ALL`, `EXCEPT ALL`.

:::danger
Unlike joins, the columns present in the left and right operands of the set operation must be exactly the same.
:::

```sql
-- general syntax
SELECT col1, col2 FROM A
{UNION [ALL] | INTERSECT [ALL] | EXCEPT [ALL]} -- choose either one, ALL is optional
SELECT col1, col2 FROM B
```

* `UNION` returns all rows from both tables, removing duplicates, unless `UNION ALL` is used.
    $$
        A \cup B = \{\ (a, b) : a \in A \text{ or } b \in B\ \}
    $$
* `INTERSECT` returns only rows that are present in both tables.
    $$
        A \cap B = \{\ (a, b) : a \in A \text{ and } b \in B\ \}
    $$
* `EXCEPT` returns only rows that are present in the first table but not in the second table
    $$
        A - B = \{\ (a, b) : a \in A \text{ and } b \notin B\ \}
    $$

### Update

`UPDATE`-`SET` is the straightforward direct way to set new values.

```sql
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;
```

Scalar subqueries can be used for values in the `SET` clause.

```sql
UPDATE Beer
SET Beer.price = (
    SELECT AVG(Wine.price) FROM Wine
    WHERE Wine.maker = Beer.Maker
)
```

If we wish to update a column based on the results of a query, we can use `FROM`:

```sql
UPDATE table_name
SET table_name.column1 = T2.value1, table_name.column2 = T2.value2, ...
FROM other_table_name as T2
WHERE table_name.id = T2.id;
```

:::warning
`UPDATE`-`SET`-`FROM` wasn't covered in the slides, so better not use it to be safe.
:::

### Delete

Delete rows from a table based on a condition.

```sql
DELETE FROM table_name
WHERE condition;
```

## Datatype Literals & Built-ins

### Strings (`CHAR`, `VARCHAR`)

**STRINGS ARE SINGLE QUOTES ONLY**. Double quotes are used to delimit/escape identifiers (table names/functions/columns) with non-conventional characters.

### Dates (`DATE`, `DATETIME`)

```sql
-- DATE
'2021-09-30' -- yyyy-mm-dd

-- DATETIME
'2021-09-30 12:34:56' -- yyyy-mm-dd hh:mm:ss

-- built-ins

GETDATE() -- returns the current date and time

SELECT DATENAME(year|month|dayofyear|day|week|weekday|hour|minute|second|millisecond, GETDATE()) -- returns a human-readable string representation of the specific part of the date

SELECT DATEPART(year|month|dayofyear|day|week|weekday|hour|minute|second|millisecond, GETDATE()) -- returns the specific part of the date as an integer (all one-based) (e.g., nth month, nth week, etc)
```

- **year**, **month**, **day** are the explicit `'yyyy-mm-dd'` components
- **week** is the nth week of the year (one-indexed)
- **weekday** is the day of the week, starting from Sunday (1 = Sunday, 2 = Monday, ..., 7 = Saturday)
- **hour**, **minute**, **second** are the explicit `'hh:mm:ss'` components, self explanatory
- Note that `year(), month(), day(), ...` are also functions that returns the associated numerical value of a `DATETIME` or `DATE`.

## Control Flow

### If

:::info
This wasn't taught in the slides, but the 23/24 S1 Past Year Paper Suggested that it was necessary.
:::

## Important Operators

### `<>` Operator

The `<>` operator is used to check if two values are not equal. Do not use `!=`!

### Logical Operators

- `AND`: Logical AND
- `OR`: Logical OR
- `NOT`: Logical NOT

### `IN` Operator

The `IN` operator is used to check if a value is in a set of values.

```sql
-- IN a hardcoded list of values:
SELECT * FROM A
WHERE A.id IN (1, 2, 3);

-- IN a non-scalar subquery:
SELECT * FROM A
WHERE A.id IN (
    SELECT B.id FROM B WHERE B.val > 10
);
```

Its negation is `NOT IN`.

### `LIKE` Operator

String pattern matching.

- `%`: Wildcard for any number of characters
- `_`: Wildcard for a single character

E.g.,

* `LIKE '__cd%'` will match any string whose 3rd and 4th characters are `cd`.
* `LIKE 'a__%a'` will match any string that starts with `a`, ends with `a`, and has at least 2 characters in between the two `a`'s.

```sql
SELECT * FROM A
WHERE A.name LIKE 'a%'; -- all names starting with 'a'
```

Its negation is `NOT LIKE`.

### `BETWEEN` Operator

The `BETWEEN` operator is used to check if a value is within a range.

```sql
SELECT * FROM A
WHERE A.id BETWEEN 1 AND 10; -- all ids between 1 and 10 (inclusive)
```

Its negation is `NOT BETWEEN`, and will return false for the same inclusive range.

### `EXISTS` Operator

The `EXISTS` operator is used to check if a subquery returns any rows.

```sql
SELECT * FROM A
WHERE EXISTS (
    SELECT * FROM B WHERE B.id = A.id
);
```

Its negation is `NOT EXISTS`.

### `IS NULL` Operator

The `IS NULL` operator is used to check if a value is `NULL`.

```sql
SELECT * FROM A
WHERE A.val IS NULL;
```

Its negation is `IS NOT NULL`.

## XML

### Structured vs Unstructured data

| Structured | Unstructured |
| --- | --- |
| Table-Formatting | Free-form |
| Must specify schema (types, constraints, attributes) | No schema |
| Structure and meaning is rigidly-defined | Structure and meanings must be inferred |

### XML Syntax

```xml
<tag attr1 = "some attribute">
    <inner> Something </inner>
    <other> Something else </other>
    <escape>
        <!-- The following symbols: < > & " ' must be escaped -->
        &lt; &gt; &amp; &quot; &apos;
    </escape>

</tag>
```

| Elements | Attributes |
| --- | --- |
| Ordered | Unordered |
| Can repeat tags | Unique attributes per tag |
| May be nested | Must be atomic |

### Document Type Definition (DTD)

#### Element Type Syntax

General Syntax:
```xml
<!ELEMENT element-name (element-content)>
```

**element-name** is the name of the tag `<element-name>`

**element-content** is a comma-separated list of subelements that may/must appear inside this element:

```xml
<!ELEMENT element-name (sub1, (sub2+ | sub3 | sub4*), sub5?)>
```

**The quantifiers `+ * ?` are exactly as per regex.**

* `+` One or more of such subelements
* `*` Zero or more of such subelements
* `?` Zero or one of such subelements
* Otherwise, one and only one of such elements

**The match group `(a|b|c)` is also like regex**
* Either `a`, `b`, or `c`.

There are also special element-contents:

* `ANY` (no parentheses): Anything goes for subelements.
* `EMPTY` (no parentheses): Should not contain any subelements (leaf node)
* `(#PCDATA)`: **Parsed Character Data** to be parsed by XML parsed (can inculde outer text)

```xml
<!ELEMENT book (title, (author+ | editor), publisher?)>
<!--
    This means that book must contain 3 inner elements:
    - Only one title element
    - Either (At least one author element), or only one editor
    - Optional publisher element
 -->

<!ELEMENT title (#PCDATA)>
<!--
    The title should contain XML parsed text data
    (nested xml is allowed also)
 -->

<!ELEMENT author EMPTY> <!-- Should have nothing inside it -->
<!ELEMENT publisher ANY> <!-- Anything you want inside -->

<!-- Example that satisfies the above: -->

<book>
    <title>A nice title <i>with italics</i></title>
    <!-- Multiple author tags with no subelements -->
    <author/> <author/> <author/>
    <publisher> <!-- Can be anything -->
        <whoever>
            <whatever></whatever>
        </whoever>
    </publisher>
</book>

```

#### Attribute Type Syntax

General Syntax:

```xml!
<!ATTLIST element-name
    attr-name attr-type {"default-value" | #REQUIRED | #IMPLIED | #FIXED "value" } >
```

**`attr-type`s:**
* `CDATA`: Character data (string)
* `(a|b|c)`: Either "a", "b", or "c" literally
* `ID`: Unique ID
* `IDREF`: ID referencing another element
* `IDREFS`: List of ids refering other elements
* `NMTOKEN`: A valid XML element name
* `NMTOKENS`: A list of valid XML element names
* `ENTITY`: An entity
* `ENTITIES`
* `NOTATION`
* `xml:`: A predefined xml value

**attribute value**:
* `"default-value"`: The attribute will have default value of `"default-value"`
* `#REQUIRED`: Mandatory
* `#IMPLIED`: Optional
* `#FIXED "value"`: The attribute must always be equal to `"value"`

The **default value** of the attribute can be specified at the end.

```xml
<!ATTLIST book
    year CDATA #REQUIRED
    price CDATA #IMPLIED
    curr (EUR|USD|SGD) #FIXED "EUR"
    index IDREFS "">

<!ATTLIST author
    name CDATA #REQUIRED>
```

**Example** using full DTD:

```xml
<!ELEMENT book (title, (author+ | editor), publisher?)>
<!ELEMENT title (#PCDATA)>
<!ELEMENT author EMPTY>
<!ELEMENT publisher ANY>
<!ATTLIST book
    year CDATA #REQUIRED
    price CDATA #IMPLIED
    curr (EUR|USD|SGD) #FIXED "EUR"
    index IDREFS "">
<!ATTLIST author
    name CDATA #REQUIRED>

<!-- The following XML object satisfies the above DTD: -->
<book year="2024" price="19.99" curr="EUR" index="3">
    <title>My <i>fancy</i> title</title>
    <author name="Bob Reynolds"/>
    <author name="Joe Mama" />
    <publisher>
        <idk></idk>
    </publisher>
</book>
```
