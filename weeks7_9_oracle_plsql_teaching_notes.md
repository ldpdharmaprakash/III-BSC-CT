# Week 8 – Developing Basic PL/SQL Programs

## 1. Learning objectives

Students will develop PL/SQL programs using:

- PL/SQL block structure.
- Variables, constants, and data types.
- Assignment statements and arithmetic operators.
- Comments and embedded SQL statements.
- Conditional statements, loops, and nested blocks.

## 2. Introduction to PL/SQL

PL/SQL (Procedural Language/SQL) is Oracle's procedural extension to SQL. SQL retrieves and manipulates data; PL/SQL adds programming features such as variables, conditions, loops, and exception handling.

A PL/SQL block has three main sections:

```sql
DECLARE
    -- Optional declaration section
BEGIN
    -- Mandatory executable section
EXCEPTION
    -- Optional exception-handling section
END;
/
```

- `DECLARE`: declares variables, constants, cursors, and other local items.
- `BEGIN`: contains executable statements.
- `EXCEPTION`: handles runtime errors.
- `/`: runs the block in SQL*Plus; it is not part of the PL/SQL block itself.

Before displaying output, enable:

```sql
SET SERVEROUTPUT ON;
```

## 3. First PL/SQL program

```sql
SET SERVEROUTPUT ON;
BEGIN
    DBMS_OUTPUT.PUT_LINE('Welcome to PL/SQL');
END;
/
```

Expected output:

```text
Welcome to PL/SQL
PL/SQL procedure successfully completed.
```

## 4. Variables and data types

A variable stores a value that can change while a PL/SQL block executes. Common types include `NUMBER`, `VARCHAR2`, `DATE`, and `BOOLEAN`.

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_NAME VARCHAR2(30) := 'Arun';
    V_MARK NUMBER := 85;
    V_RESULT VARCHAR2(10);
BEGIN
    V_RESULT := 'PASS';
    DBMS_OUTPUT.PUT_LINE('Name: ' || V_NAME);
    DBMS_OUTPUT.PUT_LINE('Mark: ' || V_MARK);
    DBMS_OUTPUT.PUT_LINE('Result: ' || V_RESULT);
END;
/
```

Expected output:

```text
Name: Arun
Mark: 85
Result: PASS
```

## 5. Constants

A constant is declared using `CONSTANT`. Its value cannot be changed after initialization.

```sql
SET SERVEROUTPUT ON;
DECLARE
    C_FINE CONSTANT NUMBER := 5;
    V_DAYS NUMBER := 4;
    V_TOTAL NUMBER;
BEGIN
    V_TOTAL := C_FINE * V_DAYS;
    DBMS_OUTPUT.PUT_LINE('Fine Amount: ' || V_TOTAL);
END;
/
```

Expected output:

```text
Fine Amount: 20
```

## 6. Assignment statements and arithmetic operators

PL/SQL uses `:=` for assignment. Common arithmetic operators are `+`, `-`, `*`, `/`, and `**` (exponentiation).

```sql
SET SERVEROUTPUT ON;
DECLARE
    A NUMBER := 20;
    B NUMBER := 5;
BEGIN
    DBMS_OUTPUT.PUT_LINE('Addition: ' || (A+B));
    DBMS_OUTPUT.PUT_LINE('Subtraction: ' || (A-B));
    DBMS_OUTPUT.PUT_LINE('Multiplication: ' || (A*B));
    DBMS_OUTPUT.PUT_LINE('Division: ' || (A/B));
    DBMS_OUTPUT.PUT_LINE('Power: ' || (A**2));
END;
/
```

Expected output:

```text
Addition: 25
Subtraction: 15
Multiplication: 100
Division: 4
Power: 400
```

## 7. Comments

Single-line comments begin with `--`. Multi-line comments are enclosed by `/*` and `*/`.

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_COUNT NUMBER := 10; -- Stores the initial count
BEGIN
    /* Display the count value */
    DBMS_OUTPUT.PUT_LINE('Count: ' || V_COUNT);
END;
/
```

## 8. Embedded SQL statements

PL/SQL can execute SQL statements such as `SELECT`, `INSERT`, `UPDATE`, and `DELETE`. A `SELECT` statement that returns one row uses `INTO` to store the result.

### 8.1 SELECT INTO

Run this example in the schema that owns the Week 3 `BOOK` table:

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_TITLE BOOK.TITLE%TYPE;
    V_PRICE BOOK.PRICE%TYPE;
BEGIN
    SELECT TITLE, PRICE INTO V_TITLE, V_PRICE
    FROM BOOK
    WHERE BOOK_ID = 201;
    DBMS_OUTPUT.PUT_LINE('Book Title: ' || V_TITLE);
    DBMS_OUTPUT.PUT_LINE('Book Price: ' || V_PRICE);
END;
/
```

Expected output depends on the row stored with `BOOK_ID = 201`. If no such row exists, the block raises `NO_DATA_FOUND`; if more than one row matches, it raises `TOO_MANY_ROWS`.

### 8.2 INSERT using PL/SQL

Use this only if the specified key does not already exist and the columns match the actual `MEMBER` table:

```sql
BEGIN
    INSERT INTO MEMBER (MEMBER_ID, MEMBER_NAME, EMAIL, PHONE, JOIN_DATE, MEMBER_TYPE)
    VALUES (106, 'Kavin', 'kavin@example.com', '9876543210', SYSDATE, 'STUDENT');
    COMMIT;
END;
/
```

Expected output:

```text
PL/SQL procedure successfully completed.
```

### 8.3 UPDATE using PL/SQL

```sql
BEGIN
    UPDATE BOOK
    SET AVAILABLE_COPIES = AVAILABLE_COPIES + 1
    WHERE BOOK_ID = 201;
    COMMIT;
END;
/
```

### 8.4 DELETE using PL/SQL

```sql
BEGIN
    DELETE FROM MEMBER WHERE MEMBER_ID = 106;
    COMMIT;
END;
/
```

## 9. Conditional statements

### 9.1 IF statement

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_MARK NUMBER := 75;
BEGIN
    IF V_MARK >= 50 THEN
        DBMS_OUTPUT.PUT_LINE('PASS');
    ELSE
        DBMS_OUTPUT.PUT_LINE('FAIL');
    END IF;
END;
/
```

Expected output: `PASS`.

### 9.2 IF–ELSIF–ELSE statement

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_MARK NUMBER := 82;
BEGIN
    IF V_MARK >= 90 THEN
        DBMS_OUTPUT.PUT_LINE('Grade A+');
    ELSIF V_MARK >= 80 THEN
        DBMS_OUTPUT.PUT_LINE('Grade A');
    ELSIF V_MARK >= 70 THEN
        DBMS_OUTPUT.PUT_LINE('Grade B');
    ELSIF V_MARK >= 50 THEN
        DBMS_OUTPUT.PUT_LINE('Grade C');
    ELSE
        DBMS_OUTPUT.PUT_LINE('Fail');
    END IF;
END;
/
```

Expected output: `Grade A`.

## 10. Nested blocks

A nested block is a PL/SQL block placed inside another block. The inner block can have its own declaration and exception sections.

```sql
SET SERVEROUTPUT ON;
DECLARE
    V_OUTER NUMBER := 100;
BEGIN
    DBMS_OUTPUT.PUT_LINE('Outer value: ' || V_OUTER);
    DECLARE
        V_INNER NUMBER := 50;
    BEGIN
        DBMS_OUTPUT.PUT_LINE('Inner value: ' || V_INNER);
    END;
END;
/
```

Expected output:

```text
Outer value: 100
Inner value: 50
```

## 11. Week 8 practical exercises

1. Write a PL/SQL block to display your name, department, and college.
2. Declare two numbers and display their sum, difference, product, and quotient.
3. Declare a constant fine rate and calculate the fine for a given number of overdue days.
4. Retrieve a book title and price using `SELECT INTO`.
5. Insert one member record using an embedded SQL statement.
6. Display PASS or FAIL using an `IF` statement.
7. Display a grade using `IF–ELSIF–ELSE`.
8. Write a nested block that displays separate outer and inner variables.

---
