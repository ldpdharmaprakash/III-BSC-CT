# Online Library Management System -- Teaching Notes (Weeks 4--6)

Project: Online Library Management System\
Environment: Oracle9i and SQL\*Plus

## Teaching sequence

Weeks 4--6 continue with the same seven-table library database created
in Week 3. First insert sample records, then retrieve and summarize
data, and finally combine related tables using operators and joins.

## Week 4 -- Implementing SQL Data Manipulation Commands

### 1. DML overview

DML changes the records stored in tables. INSERT adds rows, UPDATE
changes existing rows, and DELETE removes selected rows. SELECT
retrieves rows and is commonly taught alongside DML in this weekly plan.

### 2. INSERT

Syntax: INSERT INTO table_name (column1, column2) VALUES (value1,
value2);

``` sql
INSERT INTO CATEGORY (category_id, category_name, description) VALUES (1, 'Computer Science', 'Computing books');\nINSERT INTO AUTHOR (author_id, author_name, biography) VALUES (1, 'A. Kumar', 'Database author');
```

Expected result in SQL\*Plus: 1 row created. (Displayed after each
successful INSERT.)

### 3. UPDATE

``` sql
UPDATE BOOK SET available_copies = 5 WHERE book_id = 101;
```

Expected result: 1 row updated, if BOOK_ID 101 exists. Always use a
WHERE clause when only selected rows should change.

### 4. DELETE

``` sql
DELETE FROM MEMBER WHERE member_id = 105;
```

Expected result: 1 row deleted only if the member exists and no related
BOOK_ISSUE row prevents deletion. A foreign-key constraint may reject
deletion of a parent row that is referenced by a child row.

### 5. SELECT, WHERE and ORDER BY

``` sql
SELECT book_id, title, price FROM BOOK WHERE price > 500 ORDER BY price DESC;
```

WHERE filters rows. ORDER BY sorts the result. ASC is ascending
(default); DESC is descending.

### 6. Arithmetic expressions

``` sql
SELECT title, price, price * 0.10 AS discount FROM BOOK;
```

Arithmetic operators include +, -, \* and /. The expression is
calculated for each returned row. The example displays a 10% amount; it
does not update the stored price.

### 7. CASE expression

``` sql
SELECT title, available_copies, CASE WHEN available_copies > 0 THEN 'Available' ELSE 'Not Available' END AS book_status FROM BOOK;
```

CASE returns a value based on conditions. In a SELECT statement it can
display a derived status without changing the table.

### 8. Substitution variables in SQL\*Plus

``` sql
SELECT book_id, title FROM BOOK WHERE category_id = &category_no;\nACCEPT p_member NUMBER PROMPT 'Enter member ID: '\nSELECT issue_id, book_id, issue_date FROM BOOK_ISSUE WHERE member_id = &p_member;
```

The ampersand (&) asks SQL*Plus to substitute a value when the statement
runs. ACCEPT can define a named SQL*Plus variable for a prompt.
Substitution is text replacement, not a bind variable.

### 9. Transaction control

COMMIT makes the current transaction permanent. ROLLBACK cancels
uncommitted changes. SAVEPOINT marks a point to which a transaction can
be rolled back. Use COMMIT only after checking the changes.

``` sql
UPDATE BOOK SET available_copies = available_copies - 1 WHERE book_id = 101;\nSAVEPOINT after_update;\n-- Check the affected data before deciding\nCOMMIT;
```

### 10. Week 4 practice

-   Insert at least three categories, three authors, three publishers
    and five members/books.
-   Update a book's available copy count.
-   Retrieve books above a selected price.
-   Sort members by name.
-   Use CASE to display book availability.
-   Run a query using a substitution variable.

## Week 5 -- Working with SQL Functions and Data Retrieval

### 1. What is a SQL function?

A function accepts an input value and returns a result. Oracle provides
built-in functions for numeric calculations, text manipulation, dates,
data conversion and group calculations.

### 2. Number functions

  Function      Purpose                    Example
  ------------- -------------------------- ----------------------------
  ROUND(n, d)   Rounds a number            ROUND(125.678, 2) → 125.68
  TRUNC(n, d)   Truncates decimal places   TRUNC(125.678, 2) → 125.67
  MOD(m, n)     Returns remainder          MOD(17, 5) → 2
  ABS(n)        Absolute value             ABS(-25) → 25

``` sql
SELECT book_id, price, ROUND(price, 0) AS rounded_price, MOD(book_id, 2) AS remainder FROM BOOK;
```

### 3. Character functions

  ------------------------------------------------------------------------
  Function                Purpose                 Example
  ----------------------- ----------------------- ------------------------
  UPPER(text)             Uppercase               UPPER('oracle') → ORACLE

  LOWER(text)             Lowercase               LOWER('SQL') → sql

  INITCAP(text)           Initial capital letters INITCAP('online
                                                  library') → Online
                                                  Library

  LENGTH(text)            Character count         LENGTH('BOOK') → 4

  SUBSTR(text, start,     Extracts part of text   SUBSTR('DATABASE',1,4) →
  length)                                         DATA

  INSTR(text, search)     Finds position          INSTR('ORACLE','A') → 3

  TRIM(text)              Removes                 TRIM(' SQL ') → SQL
                          leading/trailing spaces 
  ------------------------------------------------------------------------

``` sql
SELECT member_id, UPPER(member_name) AS member_name_upper, LENGTH(member_name) AS name_length FROM MEMBER;
```

### 4. Date functions

  ------------------------------------------------------------------------------
  Function                  Purpose                 Example
  ------------------------- ----------------------- ----------------------------
  SYSDATE                   Current database date   SYSDATE
                            and time                

  ADD_MONTHS(date,n)        Adds months             ADD_MONTHS(SYSDATE, 1)

  MONTHS_BETWEEN(d1,d2)     Difference in months    MONTHS_BETWEEN(SYSDATE,
                                                    join_date)

  LAST_DAY(date)            Last day of month       LAST_DAY(SYSDATE)

  NEXT_DAY(date,'MONDAY')   Next named weekday      NEXT_DAY(SYSDATE,'MONDAY')
  ------------------------------------------------------------------------------

``` sql
SELECT member_id, member_name, join_date, ADD_MONTHS(join_date, 12) AS renewal_date FROM MEMBER;
```

### 5. Conversion functions

TO_CHAR converts a number or date to character format. TO_DATE converts
character data to a DATE using a format model. TO_NUMBER converts
character data to a number. Use explicit formats to avoid ambiguity.

``` sql
SELECT TO_CHAR(SYSDATE, 'DD-MON-YYYY') AS today FROM DUAL;\nSELECT TO_DATE('31-DEC-2026', 'DD-MON-YYYY') AS sample_date FROM DUAL;\nSELECT TO_NUMBER('1250') + 50 AS total FROM DUAL;
```

### 6. Aggregate (group) functions

  Function      Purpose
  ------------- ---------------------
  COUNT(\*)     Counts rows
  SUM(column)   Adds numeric values
  AVG(column)   Calculates average
  MIN(column)   Finds minimum
  MAX(column)   Finds maximum

``` sql
SELECT COUNT(*) AS total_books, AVG(price) AS average_price, MIN(price) AS lowest_price, MAX(price) AS highest_price FROM BOOK;
```

### 7. GROUP BY and HAVING

GROUP BY creates groups of rows with the same value. Aggregate functions
calculate a result for each group. HAVING filters groups after grouping;
WHERE filters individual rows before grouping.

``` sql
SELECT category_id, COUNT(*) AS book_count FROM BOOK GROUP BY category_id;\nSELECT category_id, COUNT(*) AS book_count FROM BOOK GROUP BY category_id HAVING COUNT(*) > 1 ORDER BY category_id;
```

### 8. Week 5 practice

-   Display book titles in uppercase.
-   Display the length of each member name.
-   Show a date one year after each member's join date.
-   Count books in each category.
-   Find the average price of books by category.
-   Display only categories containing more than one book.

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
