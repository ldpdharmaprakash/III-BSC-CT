# Online Library Management System -- (Weeks 4)

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

