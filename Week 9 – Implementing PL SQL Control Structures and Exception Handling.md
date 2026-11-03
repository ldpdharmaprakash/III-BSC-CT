
# Week 9 – Implementing PL/SQL Control Structures and Exception Handling

## 1. Learning objectives

Students will develop PL/SQL programs using:

- IF and CASE statements.
- Basic LOOP, WHILE LOOP, and FOR LOOP constructs.
- Exception handling techniques.
- Transaction control statements such as `COMMIT`, `ROLLBACK`, and `SAVEPOINT`.

## 2. CASE statement

A CASE statement selects one execution path based on a condition or expression.

### 2.1 Simple CASE

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_CHOICE NUMBER := 2;
BEGIN
    CASE V_CHOICE
        WHEN 1 THEN DBMS_OUTPUT.PUT_LINE('Add Book');
        WHEN 2 THEN DBMS_OUTPUT.PUT_LINE('Issue Book');
        WHEN 3 THEN DBMS_OUTPUT.PUT_LINE('Return Book');
        ELSE DBMS_OUTPUT.PUT_LINE('Invalid Choice');
    END CASE;
END;
/
```

Expected output: `Issue Book`.

### 2.2 Searched CASE

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_MARK NUMBER := 76;
BEGIN
    CASE
        WHEN V_MARK >= 90 THEN DBMS_OUTPUT.PUT_LINE('Grade A+');
        WHEN V_MARK >= 80 THEN DBMS_OUTPUT.PUT_LINE('Grade A');
        WHEN V_MARK >= 70 THEN DBMS_OUTPUT.PUT_LINE('Grade B');
        WHEN V_MARK >= 50 THEN DBMS_OUTPUT.PUT_LINE('Grade C');
        ELSE DBMS_OUTPUT.PUT_LINE('Fail');
    END CASE;
END;
/
```

Expected output: `Grade B`.

## 3. LOOP statement

A basic LOOP repeats statements until an `EXIT` condition is met.

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_NUM NUMBER := 1;
BEGIN
    LOOP
        DBMS_OUTPUT.PUT_LINE('Number: ' || V_NUM);
        V_NUM := V_NUM + 1;
        EXIT WHEN V_NUM > 5;
    END LOOP;
END;
/
```

Expected output:

```text
Number: 1
Number: 2
Number: 3
Number: 4
Number: 5
```

## 4. WHILE LOOP

A WHILE LOOP repeats while its condition is true. The condition is checked before each iteration.

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_NUM NUMBER := 1;
BEGIN
    WHILE V_NUM <= 5 LOOP
        DBMS_OUTPUT.PUT_LINE('Number: ' || V_NUM);
        V_NUM := V_NUM + 1;
    END LOOP;
END;
/
```

Expected output: numbers 1 through 5, each on a separate line.

## 5. FOR LOOP

A FOR LOOP repeats for a specified integer range. The loop index is declared automatically and is read-only inside the loop.

```sql
SET SERVEROUTPUT ON;
BEGIN
    FOR V_NUM IN 1..5 LOOP
        DBMS_OUTPUT.PUT_LINE('Number: ' || V_NUM);
    END LOOP;
END;
/
```

Expected output: numbers 1 through 5, each on a separate line.

### 5.1 Reverse FOR LOOP

```sql
SET SERVEROUTPUT ON;
BEGIN
    FOR V_NUM IN REVERSE 1..5 LOOP
        DBMS_OUTPUT.PUT_LINE('Number: ' || V_NUM);
    END LOOP;
END;
/
```

Expected output: numbers 5 through 1.

## 6. Nested loops

```sql
SET SERVEROUTPUT ON;
BEGIN
    FOR I IN 1..3 LOOP
        FOR J IN 1..2 LOOP
            DBMS_OUTPUT.PUT_LINE('I = ' || I || ', J = ' || J);
        END LOOP;
    END LOOP;
END;
/
```

Expected output:

```text
I = 1, J = 1
I = 1, J = 2
I = 2, J = 1
I = 2, J = 2
I = 3, J = 1
I = 3, J = 2
```

## 7. Exception handling

An exception is a runtime error or an exceptional condition that changes the normal flow of a PL/SQL program. The `EXCEPTION` section handles such conditions.

General structure:

```sql
BEGIN
    -- Executable statements
EXCEPTION
    WHEN exception_name THEN
        -- Handling statements
    WHEN OTHERS THEN
        -- General error handling
END;
/
```

### 7.1 ZERO_DIVIDE

```sql
SET SERVEROUTPUT ON;
DECLARE
    A NUMBER := 10;
    B NUMBER := 0;
    V_RESULT NUMBER;
BEGIN
    V_RESULT := A / B;
    DBMS_OUTPUT.PUT_LINE('Result: ' || V_RESULT);
EXCEPTION
    WHEN ZERO_DIVIDE THEN
        DBMS_OUTPUT.PUT_LINE('Error: Division by zero is not allowed.');
END;
/
```

Expected output:

```text
Error: Division by zero is not allowed.
```

### 7.2 NO_DATA_FOUND

This example assumes the `BOOK` table has a `BOOK_ID` column and no row with ID `99999`.

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_TITLE BOOK.TITLE%TYPE;
BEGIN
    SELECT TITLE INTO V_TITLE
    FROM BOOK
    WHERE BOOK_ID = 99999;
    DBMS_OUTPUT.PUT_LINE('Book: ' || V_TITLE);
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('No book found for the given ID.');
END;
/
```

Expected output if no matching row exists: `No book found for the given ID.`

### 7.3 TOO_MANY_ROWS

This example deliberately selects all book titles into a scalar variable. It raises `TOO_MANY_ROWS` if more than one row exists.

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_TITLE BOOK.TITLE%TYPE;
BEGIN
    SELECT TITLE INTO V_TITLE FROM BOOK;
    DBMS_OUTPUT.PUT_LINE('Book: ' || V_TITLE);
EXCEPTION
    WHEN TOO_MANY_ROWS THEN
        DBMS_OUTPUT.PUT_LINE('More than one book was found.');
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('No books are available.');
END;
/
```

### 7.4 OTHERS exception

`WHEN OTHERS` handles exceptions not handled by earlier handlers. `SQLCODE` returns the error number and `SQLERRM` returns the error message.

```sql
SET SERVEROUTPUT ON;
BEGIN
    INSERT INTO BOOK (BOOK_ID, TITLE, CATEGORY_ID, AUTHOR_ID, PUBLISHER_ID, PRICE, TOTAL_COPIES, AVAILABLE_COPIES)
    VALUES (201, 'Duplicate Book', 1, 1, 1, 300, 5, 5);
EXCEPTION
    WHEN DUP_VAL_ON_INDEX THEN
        DBMS_OUTPUT.PUT_LINE('Error: Duplicate book ID or unique value.');
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error code: ' || SQLCODE);
        DBMS_OUTPUT.PUT_LINE('Error message: ' || SQLERRM);
END;
/
```

This example assumes the full Week 3 `BOOK` table and that `BOOK_ID = 201` already exists. If the key is not present, the insert may succeed; if other referenced IDs do not exist, a foreign-key exception may occur. Roll back after a successful test insert if it should not be retained.

## 8. User-defined exceptions

A user-defined exception is declared by the programmer and raised when a specific business rule is violated.

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_COPIES NUMBER := 0;
    E_NO_COPIES EXCEPTION;
BEGIN
    IF V_COPIES <= 0 THEN
        RAISE E_NO_COPIES;
    END IF;
    DBMS_OUTPUT.PUT_LINE('Book can be issued.');
EXCEPTION
    WHEN E_NO_COPIES THEN
        DBMS_OUTPUT.PUT_LINE('Cannot issue the book: no copies available.');
END;
/
```

Expected output: `Cannot issue the book: no copies available.`

## 9. Transaction control statements

Transaction control statements manage changes made by DML statements.

- `COMMIT`: permanently saves the current transaction.
- `ROLLBACK`: cancels uncommitted changes in the current transaction.
- `SAVEPOINT`: marks a point within a transaction to which a partial rollback can return.

### 9.1 COMMIT

```sql
INSERT INTO MEMBER (MEMBER_ID, MEMBER_NAME, EMAIL, PHONE, JOIN_DATE, MEMBER_TYPE)
VALUES (107, 'Nila', 'nila@example.com', '9876500000', SYSDATE, 'STUDENT');
COMMIT;
```

### 9.2 ROLLBACK

```sql
UPDATE BOOK SET AVAILABLE_COPIES = AVAILABLE_COPIES - 1 WHERE BOOK_ID = 201;
ROLLBACK;
```

The uncommitted update is cancelled.

### 9.3 SAVEPOINT and partial ROLLBACK

```sql
UPDATE BOOK SET AVAILABLE_COPIES = AVAILABLE_COPIES - 1 WHERE BOOK_ID = 201;
SAVEPOINT AFTER_BOOK_UPDATE;
UPDATE BOOK SET AVAILABLE_COPIES = AVAILABLE_COPIES - 1 WHERE BOOK_ID = 202;
ROLLBACK TO AFTER_BOOK_UPDATE;
COMMIT;
```

The update to book 202 is undone; the earlier update to book 201 is committed. Check the rows before and after execution to observe the result. Use this example only with known existing book IDs and suitable available-copy values.

## 10. Week 9 practical exercises

1. Write a PL/SQL program using a simple CASE statement to display a library menu option.
2. Write a searched CASE program to display a grade based on marks.
3. Display numbers from 1 to 10 using a basic LOOP.
4. Display even numbers from 2 to 20 using a WHILE LOOP.
5. Display the multiplication table of a number using a FOR LOOP.
6. Use nested FOR loops to print a 3-by-3 number pattern.
7. Handle division by zero using `ZERO_DIVIDE`.
8. Handle a missing book using `NO_DATA_FOUND`.
9. Write a user-defined exception for an unavailable book.
10. Demonstrate `COMMIT`, `ROLLBACK`, and `SAVEPOINT` using a test record.

---

# Common SQL*Plus commands for Weeks 7–9

| Command | Purpose |
|---|---|
| `SHOW USER;` | Displays the connected user |
| `SHOW CON_NAME;` | Displays the current container |
| `DESC BOOK;` | Displays the structure of BOOK |
| `SELECT * FROM BOOK;` | Displays BOOK records |
| `SET SERVEROUTPUT ON;` | Displays output from `DBMS_OUTPUT` |
| `/` | Executes the PL/SQL block in the SQL*Plus buffer |
| `COMMIT;` | Saves the current transaction |
| `ROLLBACK;` | Cancels uncommitted changes |
| `HOST CLS` | Clears the Windows SQL*Plus screen |

# Important error messages

| Error | Common reason | What to check |
|---|---|---|
| `ORA-65096` | Attempt to create a local-style user in `CDB$ROOT` | Switch to `XEPDB1` before creating application users |
| `ORA-65066` | A common-user change requires all containers | Run the intended common-user change from the root with `CONTAINER=ALL` |
| `ORA-01920` | User or role name already exists | Check `DBA_USERS` and `DBA_ROLES` |
| `ORA-01917` | Referenced user or role does not exist | Verify that user/role creation succeeded |
| `ORA-01017` | Invalid username/password | Check credentials and connection service |
| `ORA-00942` | Table or view does not exist or is not accessible | Check schema, object name, and privileges |
| `ORA-01403` / `NO_DATA_FOUND` | `SELECT INTO` returned no rows | Check the search condition or handle the exception |
| `ORA-01422` / `TOO_MANY_ROWS` | `SELECT INTO` returned multiple rows | Make the condition unique or use a cursor/collection |

# Final practical checklist

- [ ] Verify the current user and container before database administration tasks.
- [ ] Create users only in the intended PDB.
- [ ] Assign appropriate tablespaces and privileges.
- [ ] Test roles and privileges by connecting as the intended users.
- [ ] Enable `SERVEROUTPUT` before running output-based PL/SQL examples.
- [ ] Execute each PL/SQL block using `/` on a new SQL*Plus line.
- [ ] Test exception handlers with controlled examples.
- [ ] Use `COMMIT` and `ROLLBACK` carefully and understand which changes are saved.
