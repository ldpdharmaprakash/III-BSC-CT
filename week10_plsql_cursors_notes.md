# Week 10 -- Working with PL/SQL Cursors

## Learning Outcomes

By the end of this week, students should be able to: 1. Explain the
purpose of cursors in PL/SQL. 2. Use implicit cursor attributes. 3.
Declare, open, fetch, and close explicit cursors. 4. Process records
using cursor FOR loops. 5. Pass values through cursor parameters. 6. Use
cursor variables (`REF CURSOR`). 7. Lock rows using
`SELECT ... FOR UPDATE`. 8. Update or delete the current cursor row
using `WHERE CURRENT OF`.

## 1. Introduction to Cursors

A cursor is a pointer to the result of an SQL statement. When a query
returns multiple rows, a cursor helps PL/SQL process those rows one by
one.

**Example:** A library has many books. A cursor can retrieve the books
and display each book's details one at a time.

### Types of cursors

  -----------------------------------------------------------------------
  Type                                Purpose
  ----------------------------------- -----------------------------------
  Implicit cursor                     Created automatically by Oracle for
                                      SQL statements.

  Explicit cursor                     Declared by the programmer to
                                      process query results.

  Cursor variable                     A variable that refers to a query
                                      result set, commonly declared using
                                      `REF CURSOR`.
  -----------------------------------------------------------------------

## 2. Sample Table

The examples use the simplified `BOOK` table from the Library Management
System practical.

``` sql
CREATE TABLE BOOK (
    BOOK_ID NUMBER PRIMARY KEY,
    TITLE VARCHAR2(100),
    AVAILABLE_COPIES NUMBER
);

INSERT INTO BOOK VALUES (201, 'Database Systems', 5);
INSERT INTO BOOK VALUES (202, 'Learning Oracle SQL', 3);
INSERT INTO BOOK VALUES (203, 'Python Basics', 4);
INSERT INTO BOOK VALUES (204, 'Java Programming', 2);

COMMIT;
```

If `BOOK` already exists, do not create it again. Avoid inserting
duplicate primary keys.

Check the data:

``` sql
SELECT * FROM BOOK ORDER BY BOOK_ID;
```

Before running PL/SQL blocks in SQL\*Plus, enable output:

``` sql
SET SERVEROUTPUT ON;
```

## 3. Implicit Cursors

An implicit cursor is created automatically by Oracle when an SQL
statement such as `INSERT`, `UPDATE`, `DELETE`, or a single-row
`SELECT INTO` is executed. The programmer does not declare, open, fetch,
or close it.

### Implicit cursor attributes

  -----------------------------------------------------------------------
  Attribute                           Purpose
  ----------------------------------- -----------------------------------
  `SQL%FOUND`                         TRUE if the SQL statement affected
                                      or returned at least one row.

  `SQL%NOTFOUND`                      TRUE if the SQL statement did not
                                      affect or return any row.

  `SQL%ROWCOUNT`                      Number of rows affected or
                                      returned.

  `SQL%ISOPEN`                        Checks whether the implicit cursor
                                      is open; it is FALSE after the
                                      statement finishes.
  -----------------------------------------------------------------------

### Example 1: Check updated rows

``` sql
BEGIN
    UPDATE BOOK
    SET AVAILABLE_COPIES = AVAILABLE_COPIES + 1
    WHERE BOOK_ID = 201;

    DBMS_OUTPUT.PUT_LINE('Rows updated: ' || SQL%ROWCOUNT);

    IF SQL%FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Book stock updated successfully.');
    ELSE
        DBMS_OUTPUT.PUT_LINE('Book not found.');
    END IF;
END;
/
```

Possible output:

``` text
Rows updated: 1
Book stock updated successfully.
```

### Example 2: Check whether a book exists

``` sql
BEGIN
    UPDATE BOOK
    SET AVAILABLE_COPIES = AVAILABLE_COPIES - 1
    WHERE BOOK_ID = 999;

    IF SQL%NOTFOUND THEN
        DBMS_OUTPUT.PUT_LINE('No matching book was found.');
    END IF;
END;
/
```

Possible output:

``` text
No matching book was found.
```

**Note:** Check `SQL%FOUND`, `SQL%NOTFOUND`, or `SQL%ROWCOUNT`
immediately after the SQL statement being tested. A later SQL statement
can replace the previous implicit cursor information.

## 4. Explicit Cursors

An explicit cursor is declared by the programmer when a query returns
multiple rows that need to be processed one by one.

### Four steps

1.  **DECLARE** -- Define the cursor and its SELECT statement.
2.  **OPEN** -- Execute the query and make its result available.
3.  **FETCH** -- Retrieve one row into variables.
4.  **CLOSE** -- Release the cursor resources.

### General syntax

``` sql
DECLARE
    CURSOR cursor_name IS
        SELECT column1, column2 FROM table_name;
BEGIN
    OPEN cursor_name;
    FETCH cursor_name INTO variable1, variable2;
    CLOSE cursor_name;
END;
/
```

One `FETCH` retrieves one row. Use a loop to process all rows.

### Example: Display all books

``` sql
DECLARE
    CURSOR book_cur IS
        SELECT BOOK_ID, TITLE, AVAILABLE_COPIES
        FROM BOOK
        ORDER BY BOOK_ID;

    v_id BOOK.BOOK_ID%TYPE;
    v_title BOOK.TITLE%TYPE;
    v_copies BOOK.AVAILABLE_COPIES%TYPE;
BEGIN
    OPEN book_cur;

    LOOP
        FETCH book_cur INTO v_id, v_title, v_copies;
        EXIT WHEN book_cur%NOTFOUND;

        DBMS_OUTPUT.PUT_LINE(
            v_id || ' - ' || v_title || ' - Available: ' || v_copies
        );
    END LOOP;

    CLOSE book_cur;
END;
/
```

Possible output:

``` text
201 - Database Systems - Available: 5
202 - Learning Oracle SQL - Available: 3
203 - Python Basics - Available: 4
204 - Java Programming - Available: 2
```

### Explicit cursor attributes

  Attribute             Purpose
  --------------------- ----------------------------------------------
  `book_cur%FOUND`      TRUE if the last fetch returned a row.
  `book_cur%NOTFOUND`   TRUE if the last fetch did not return a row.
  `book_cur%ROWCOUNT`   Number of rows fetched so far.
  `book_cur%ISOPEN`     TRUE if the cursor is currently open.

### Example: Display the total number of fetched rows

``` sql
DECLARE
    CURSOR book_cur IS SELECT BOOK_ID, TITLE FROM BOOK;
    v_id BOOK.BOOK_ID%TYPE;
    v_title BOOK.TITLE%TYPE;
BEGIN
    OPEN book_cur;

    LOOP
        FETCH book_cur INTO v_id, v_title;
        EXIT WHEN book_cur%NOTFOUND;
        DBMS_OUTPUT.PUT_LINE(v_title);
    END LOOP;

    DBMS_OUTPUT.PUT_LINE('Total rows fetched: ' || book_cur%ROWCOUNT);
    CLOSE book_cur;
END;
/
```

## 5. Cursor FOR Loops

A cursor FOR loop is a simple way to process all rows returned by a
cursor. Oracle automatically opens the cursor, fetches each row, exits
when no more rows are available, and closes the cursor.

### Syntax

``` sql
FOR record_name IN cursor_name LOOP
    statements;
END LOOP;
```

### Example: Display book details

``` sql
DECLARE
    CURSOR book_cur IS
        SELECT BOOK_ID, TITLE, AVAILABLE_COPIES
        FROM BOOK
        ORDER BY BOOK_ID;
BEGIN
    FOR book_rec IN book_cur LOOP
        DBMS_OUTPUT.PUT_LINE(
            book_rec.BOOK_ID || ' - ' ||
            book_rec.TITLE || ' - ' ||
            book_rec.AVAILABLE_COPIES
        );
    END LOOP;
END;
/
```

`book_rec` is automatically created by Oracle for the loop. It contains
the columns returned by the cursor query.

### Cursor FOR loop without a separately declared cursor

``` sql
BEGIN
    FOR book_rec IN (
        SELECT BOOK_ID, TITLE FROM BOOK ORDER BY BOOK_ID
    ) LOOP
        DBMS_OUTPUT.PUT_LINE(book_rec.BOOK_ID || ' - ' || book_rec.TITLE);
    END LOOP;
END;
/
```

This is useful for short queries.

## 6. Cursor Parameters

A parameterized cursor accepts values when it is opened. The same cursor
can therefore retrieve different sets of records.

### Syntax

``` sql
CURSOR cursor_name(parameter_name datatype) IS
    SELECT columns
    FROM table_name
    WHERE column_name = parameter_name;
```

### Example: Books with a minimum number of available copies

``` sql
DECLARE
    CURSOR book_cur(p_min_copies NUMBER) IS
        SELECT BOOK_ID, TITLE, AVAILABLE_COPIES
        FROM BOOK
        WHERE AVAILABLE_COPIES >= p_min_copies
        ORDER BY BOOK_ID;
BEGIN
    DBMS_OUTPUT.PUT_LINE('Books with at least 3 available copies:');

    FOR book_rec IN book_cur(3) LOOP
        DBMS_OUTPUT.PUT_LINE(
            book_rec.BOOK_ID || ' - ' || book_rec.TITLE ||
            ' - ' || book_rec.AVAILABLE_COPIES
        );
    END LOOP;
END;
/
```

The value `3` is passed to `p_min_copies`. Only books with at least
three available copies are displayed.

### Example: Reuse the cursor with different values

``` sql
DECLARE
    CURSOR book_cur(p_min_copies NUMBER) IS
        SELECT BOOK_ID, TITLE, AVAILABLE_COPIES
        FROM BOOK
        WHERE AVAILABLE_COPIES >= p_min_copies;
BEGIN
    DBMS_OUTPUT.PUT_LINE('Minimum stock: 2');
    FOR book_rec IN book_cur(2) LOOP
        DBMS_OUTPUT.PUT_LINE(book_rec.TITLE);
    END LOOP;

    DBMS_OUTPUT.PUT_LINE('Minimum stock: 4');
    FOR book_rec IN book_cur(4) LOOP
        DBMS_OUTPUT.PUT_LINE(book_rec.TITLE);
    END LOOP;
END;
/
```

## 7. Cursor Variables (REF CURSOR)

A cursor variable is a variable that can refer to a query result set. It
can be opened for different queries during program execution.

`SYS_REFCURSOR` is a predefined weak REF CURSOR type in Oracle.

### Example

``` sql
DECLARE
    book_cv SYS_REFCURSOR;
    v_id BOOK.BOOK_ID%TYPE;
    v_title BOOK.TITLE%TYPE;
BEGIN
    OPEN book_cv FOR
        SELECT BOOK_ID, TITLE
        FROM BOOK
        ORDER BY BOOK_ID;

    LOOP
        FETCH book_cv INTO v_id, v_title;
        EXIT WHEN book_cv%NOTFOUND;
        DBMS_OUTPUT.PUT_LINE(v_id || ' - ' || v_title);
    END LOOP;

    CLOSE book_cv;
END;
/
```

### Important points

-   `REF CURSOR` is a cursor variable type.
-   `SYS_REFCURSOR` is a predefined weak REF CURSOR type.
-   It can be opened for a query at runtime.
-   The variables in `FETCH` must match the selected columns in number
    and compatible data types.

## 8. SELECT ... FOR UPDATE

`SELECT ... FOR UPDATE` selects and locks rows so that another
transaction cannot modify those locked rows until the current
transaction ends.

It is useful when a program selects a row and then updates that same
row.

### Syntax

``` sql
SELECT columns
FROM table_name
WHERE condition
FOR UPDATE;
```

### Example: Lock and update a book row

``` sql
DECLARE
    v_copies BOOK.AVAILABLE_COPIES%TYPE;
BEGIN
    SELECT AVAILABLE_COPIES
    INTO v_copies
    FROM BOOK
    WHERE BOOK_ID = 201
    FOR UPDATE;

    DBMS_OUTPUT.PUT_LINE('Current available copies: ' || v_copies);

    UPDATE BOOK
    SET AVAILABLE_COPIES = AVAILABLE_COPIES - 1
    WHERE BOOK_ID = 201;

    COMMIT;
    DBMS_OUTPUT.PUT_LINE('Book stock updated.');
END;
/
```

Explanation: 1. Oracle selects the book with ID 201. 2. `FOR UPDATE`
locks the selected row. 3. The program reduces the available copies by
one. 4. `COMMIT` saves the change and releases the lock.

A row lock is held until `COMMIT` or `ROLLBACK`. Do not keep
transactions open longer than necessary.

## 9. WHERE CURRENT OF

`WHERE CURRENT OF` is used with an explicit cursor declared with
`FOR UPDATE`. It identifies the row currently fetched by the cursor, so
the program does not need to write the primary-key condition again.

### Example: Reduce stock for books with more than three copies

``` sql
DECLARE
    CURSOR book_cur IS
        SELECT BOOK_ID, TITLE, AVAILABLE_COPIES
        FROM BOOK
        WHERE AVAILABLE_COPIES > 3
        FOR UPDATE;
BEGIN
    FOR book_rec IN book_cur LOOP
        UPDATE BOOK
        SET AVAILABLE_COPIES = AVAILABLE_COPIES - 1
        WHERE CURRENT OF book_cur;

        DBMS_OUTPUT.PUT_LINE('Updated: ' || book_rec.TITLE);
    END LOOP;

    COMMIT;
END;
/
```

Explanation: - The cursor selects books with more than three available
copies. - `FOR UPDATE` locks the selected rows. - The cursor FOR loop
processes each row. - `WHERE CURRENT OF book_cur` updates the row
currently fetched. - `COMMIT` saves the updates and releases the locks.

**Important:** The cursor must be declared with `FOR UPDATE` when using
`WHERE CURRENT OF` to update or delete its current row.

## 10. Comparison of Cursor Types and Techniques

  -----------------------------------------------------------------------
  Feature                             Main purpose
  ----------------------------------- -----------------------------------
  Implicit cursor                     Oracle automatically handles SQL
                                      statement processing.

  Explicit cursor                     Programmer controls row-by-row
                                      processing.

  Cursor FOR loop                     Simplifies explicit cursor
                                      processing.

  Cursor parameter                    Passes values to filter cursor
                                      results.

  Cursor variable                     Refers to a query result set that
                                      can be opened at runtime.

  `SELECT ... FOR UPDATE`             Locks selected rows for a later
                                      update or delete.

  `WHERE CURRENT OF`                  Identifies the current row of a FOR
                                      UPDATE cursor.
  -----------------------------------------------------------------------

## 11. Practical Exercises

### Easy Level

1.  Write a PL/SQL block to update the available copies of book ID 201
    and display `SQL%ROWCOUNT`.
2.  Write an explicit cursor to display all book IDs and titles.
3.  Write a cursor FOR loop to display all book titles.
4.  Write a parameterized cursor to display books with at least four
    available copies.

### Intermediate Level

5.  Write an explicit cursor to display book details and count the
    fetched rows.
6.  Write a cursor FOR loop to display books whose available copies are
    less than or equal to three.
7.  Write a `SYS_REFCURSOR` block to retrieve and display book IDs and
    titles.
8.  Use `SELECT ... FOR UPDATE` to lock a book row, update its available
    copies, and commit the transaction.

### Advanced Level

9.  Declare a cursor with `FOR UPDATE` to select books with more than
    two available copies. Use `WHERE CURRENT OF` to increase their
    available copies by one.
10. Use a parameterized cursor twice with different minimum-stock values
    and compare the returned records.
11. Write a cursor block that displays each book title and its stock
    status: `AVAILABLE` when copies are greater than zero and
    `OUT OF STOCK` when copies are zero.

## 12. Common Errors and Precautions

  ---------------------------------------------------------------------------------------------------------
  Error or issue                                                        Reason / solution
  --------------------------------------------------------------------- -----------------------------------
  `ORA-06550`                                                           PL/SQL syntax or compilation error.
                                                                        Check spelling, semicolons, and
                                                                        block structure.

  `ORA-01001: invalid cursor`                                           Cursor was not opened, was already
                                                                        closed, or is invalid when used.

  `ORA-01002: fetch out of sequence`                                    Fetch was attempted in an invalid
                                                                        cursor state. Check the
                                                                        OPEN/FETCH/CLOSE sequence.

  `ORA-01422: exact fetch returns more than requested number of rows`   A `SELECT INTO` returned more than
                                                                        one row. Use a cursor or restrict
                                                                        the query to one row.

  `ORA-01403: no data found`                                            A `SELECT INTO` did not find a row.
                                                                        Handle `NO_DATA_FOUND` or use a
                                                                        cursor where appropriate.

  Rows remain locked                                                    The transaction has not ended.
                                                                        Execute `COMMIT` or `ROLLBACK` when
                                                                        appropriate.
  ---------------------------------------------------------------------------------------------------------

### Good practices

-   Always close an explicitly opened cursor.
-   Use cursor FOR loops when manual cursor control is not required.
-   Use `%TYPE` to declare variables based on table column data types.
-   Check `%NOTFOUND` immediately after `FETCH`.
-   Use `COMMIT` or `ROLLBACK` to end transactions and release locks.
-   Test update and delete statements carefully before committing.

## 13. Viva Questions and Short Answers

**1. What is a cursor?**\
A cursor is a pointer to the result of an SQL statement. It helps
process query results row by row.

**2. What are the main types of cursors?**\
Implicit cursors and explicit cursors. Cursor variables are another
cursor mechanism.

**3. What is an implicit cursor?**\
An implicit cursor is created automatically by Oracle for SQL
statements.

**4. What is an explicit cursor?**\
An explicit cursor is declared by the programmer to process a query that
may return multiple rows.

**5. What are the four steps of an explicit cursor?**\
Declare, open, fetch, and close.

**6. What is `SQL%ROWCOUNT`?**\
It returns the number of rows affected or returned by the most recent
relevant SQL statement.

**7. What is `%NOTFOUND`?**\
It becomes TRUE when a cursor fetch does not return a row.

**8. What is a cursor FOR loop?**\
It automatically opens, fetches from, and closes a cursor.

**9. What is a parameterized cursor?**\
It is a cursor that accepts values when it is opened.

**10. What is a REF CURSOR?**\
A REF CURSOR is a cursor variable that can refer to a query result set.

**11. What is `SYS_REFCURSOR`?**\
It is Oracle's predefined weak REF CURSOR type.

**12. What is `SELECT ... FOR UPDATE`?**\
It selects and locks rows so they can be safely updated or deleted in
the current transaction.

**13. What is `WHERE CURRENT OF`?**\
It refers to the current row fetched by a cursor declared with
`FOR UPDATE`.

**14. Why is `COMMIT` used after an update?**\
It saves the transaction's changes and releases its row locks.

**15. What is the difference between an explicit cursor and a cursor FOR
loop?**\
An explicit cursor can be controlled manually using OPEN, FETCH, and
CLOSE. A cursor FOR loop handles these steps automatically.

## 14. Week 10 Revision Summary

-   **Implicit Cursor:** Automatically created by Oracle.
-   **Explicit Cursor:** Declared and controlled by the programmer.
-   **Cursor FOR Loop:** Automatically opens, fetches, and closes the
    cursor.
-   **Cursor Parameter:** Passes values to a cursor.
-   **Cursor Variable:** Uses `REF CURSOR` to refer to a result set.
-   **SELECT ... FOR UPDATE:** Locks selected rows.
-   **WHERE CURRENT OF:** Updates or deletes the current row of a FOR
    UPDATE cursor.

**Key idea:** Cursors allow PL/SQL programs to retrieve and process
multiple records one row at a time.
