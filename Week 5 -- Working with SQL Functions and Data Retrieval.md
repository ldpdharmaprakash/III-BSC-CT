# Online Library Management System -- (Weeks 5)


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
