
## Week 6 -- Implementing SQL Operators and Joins

### 1. SQL operators

  -----------------------------------------------------------------------
  Operator group          Operators / examples    Purpose
  ----------------------- ----------------------- -----------------------
  Arithmetic              \+ - \* /               Calculate values

  Comparison              = \<\> \> \< \>= \<=    Compare values

  Logical                 AND, OR, NOT            Combine conditions

  Special                 IN, BETWEEN, LIKE, IS   Match a list, range,
                          NULL                    pattern or NULL

  Set                     UNION, INTERSECT, MINUS Combine query results
  -----------------------------------------------------------------------

### 2. Special operator examples

``` sql
SELECT title FROM BOOK WHERE price BETWEEN 300 AND 800;\nSELECT title FROM BOOK WHERE category_id IN (1, 2);\nSELECT title FROM BOOK WHERE title LIKE 'D%';\nSELECT member_name FROM MEMBER WHERE email IS NULL;
```

LIKE uses % for zero or more characters and \_ for exactly one
character. BETWEEN includes both boundary values. NULL must be tested
with IS NULL or IS NOT NULL, not = NULL.

### 3. Why joins are needed

Related information is stored in separate normalized tables. A join
combines columns from related tables so a report can show readable
information, such as a book title with the member who borrowed it.

### 4. INNER JOIN

``` sql
SELECT i.issue_id, m.member_name, b.title, i.issue_date FROM BOOK_ISSUE i INNER JOIN MEMBER m ON i.member_id = m.member_id INNER JOIN BOOK b ON i.book_id = b.book_id;
```

INNER JOIN returns rows where the join condition matches in both tables.

### 5. OUTER JOIN

``` sql
SELECT m.member_id, m.member_name, i.issue_id FROM MEMBER m LEFT OUTER JOIN BOOK_ISSUE i ON m.member_id = i.member_id;
```

LEFT OUTER JOIN returns all rows from the left table and matching rows
from the right table. Unmatched right-side columns display NULL. RIGHT
OUTER JOIN and FULL OUTER JOIN are other forms supported by Oracle9i.

### 6. SELF JOIN

A self join joins a table to itself. The supplied library design does
not contain a manager/member hierarchy, so use a simple demonstration
with a book table to compare books in the same category.

``` sql
SELECT b1.title AS book_one, b2.title AS book_two, b1.category_id FROM BOOK b1 JOIN BOOK b2 ON b1.category_id = b2.category_id AND b1.book_id < b2.book_id;
```

### 7. NATURAL JOIN

``` sql
SELECT category_id, category_name, title FROM CATEGORY NATURAL JOIN BOOK;
```

NATURAL JOIN automatically joins columns with the same name and
compatible data types. Use it carefully: an unintended same-named column
can change the join condition. An explicit ON clause is usually clearer
for teaching and maintenance.

### 8. CROSS JOIN

``` sql
SELECT c.category_name, a.author_name FROM CATEGORY c CROSS JOIN AUTHOR a;
```

CROSS JOIN returns the Cartesian product: every category is paired with
every author. If there are 3 categories and 4 authors, the result
contains 12 rows.

### 9. Set operators

Set operators combine the results of two SELECT statements. Each SELECT
must return the same number of columns, in the same order, with
compatible data types.

``` sql
SELECT member_id FROM BOOK_ISSUE\nUNION\nSELECT member_id FROM BOOK_RETURN;\n\nSELECT member_id FROM BOOK_ISSUE\nINTERSECT\nSELECT member_id FROM BOOK_RETURN;\n\nSELECT member_id FROM BOOK_ISSUE\nMINUS\nSELECT member_id FROM BOOK_RETURN;
```

-   UNION returns distinct rows from both queries.
-   INTERSECT returns rows common to both queries.
-   MINUS returns rows from the first query that are absent from the
    second query.

### 10. Week 6 practice

-   Find books priced between two values.
-   Find books whose title begins with a chosen letter.
-   Display members and their issue transactions using INNER JOIN.
-   Display all members, including those with no issue, using LEFT OUTER
    JOIN.
-   Generate category-author pairs using CROSS JOIN.
-   Compare member IDs in issue and return records using UNION,
    INTERSECT and MINUS.

## Practical checking and expected output guidance

The exact rows and numeric totals depend on the sample records inserted
by each class. The notes therefore state the expected result shape or
SQL\*Plus status message. Before showing a fixed result table, insert
the same sample data on every lab computer and verify the output in your
Oracle installation.

## End-of-week deliverables

-   Week 4: DML practice script and sample records in all required
    tables.
-   Week 5: SQL function demonstrations and grouped reports.
-   Week 6: operator queries, join queries and set-operation
    demonstrations.
