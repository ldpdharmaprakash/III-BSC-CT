
# Week 7 – Managing Oracle Database Users and Security

## 1. Learning objectives

Students will learn to:

- Create and manage Oracle database users.
- Understand tablespaces and assign default and temporary tablespaces.
- Create and manage roles.
- Grant and revoke system and object privileges.
- Apply basic database security practices.

## 2. Database user

A database user is an account used to connect to Oracle and own database objects such as tables, views, sequences, and procedures. A user needs the `CREATE SESSION` system privilege to log in.

### 2.1 Create users

Run these commands as an administrator in `XEPDB1`:

```sql
CREATE USER LIB_ADMIN IDENTIFIED BY LibAdmin123 DEFAULT TABLESPACE USERS TEMPORARY TABLESPACE TEMP;
CREATE USER LIB_STAFF IDENTIFIED BY LibStaff123 DEFAULT TABLESPACE USERS TEMPORARY TABLESPACE TEMP;
CREATE USER LIB_VIEWER IDENTIFIED BY LibView123 DEFAULT TABLESPACE USERS TEMPORARY TABLESPACE TEMP;
```

Expected result for each statement:

```text
User created.
```

If a user already exists, Oracle reports `ORA-01920`. Check first:

```sql
SELECT USERNAME FROM DBA_USERS WHERE USERNAME IN ('LIB_ADMIN','LIB_STAFF','LIB_VIEWER');
```

### 2.2 Grant login and system privileges

```sql
GRANT CREATE SESSION TO LIB_ADMIN;
GRANT CREATE SESSION TO LIB_STAFF;
GRANT CREATE SESSION TO LIB_VIEWER;
GRANT CREATE TABLE, CREATE VIEW, CREATE SEQUENCE, CREATE PROCEDURE, CREATE ROLE TO LIB_ADMIN;
```

`CREATE SESSION` permits login. The other privileges allow `LIB_ADMIN` to create the specified objects and roles in its own schema.

### 2.3 Connect as a user

Use SQL*Plus connection syntax. Replace the service name if your PDB service differs:

```sql
CONNECT LIB_ADMIN/LibAdmin123@localhost:1521/XEPDB1
SHOW USER;
SHOW CON_NAME;
```

Expected user and container:

```text
USER is "LIB_ADMIN"
XEPDB1
```

### 2.4 Alter a user

As an administrator:

```sql
ALTER USER LIB_STAFF IDENTIFIED BY StaffNew123;
ALTER USER LIB_VIEWER ACCOUNT LOCK;
ALTER USER LIB_VIEWER ACCOUNT UNLOCK;
```

Expected result: `User altered.`

### 2.5 Drop a user

Only drop a user when its account and objects are no longer required. `CASCADE` removes the user's schema objects as well.

```sql
DROP USER LIB_VIEWER;
```

If the user owns objects, Oracle may require:

```sql
DROP USER LIB_VIEWER CASCADE;
```

Do not execute this in a shared or production database without authorization and a backup.

## 3. Tablespaces

A tablespace is a logical storage unit in an Oracle database. It contains segments for database objects. A datafile provides the physical storage for a permanent tablespace.

### 3.1 View existing tablespaces

As a user with dictionary access:

```sql
SELECT TABLESPACE_NAME, STATUS, CONTENTS FROM DBA_TABLESPACES ORDER BY TABLESPACE_NAME;
```

### 3.2 Create a tablespace

The following example is for a typical Windows Oracle XE installation. Run as SYSDBA or an administrator with the required privilege, and confirm the Oracle datafile directory on your computer before using the path.

```sql
CREATE TABLESPACE LIBRARY_TS DATAFILE 'C:\app\oracle\oradata\XE\XEPDB1\library_ts01.dbf' SIZE 50M AUTOEXTEND ON NEXT 10M MAXSIZE 500M;
```

If that path does not exist on your computer, use the actual datafile directory from your Oracle installation. Do not copy a guessed path blindly.

### 3.3 Assign a tablespace to a user

```sql
ALTER USER LIB_ADMIN DEFAULT TABLESPACE LIBRARY_TS;
```

### 3.4 Verify tablespace details

```sql
SELECT TABLESPACE_NAME, FILE_NAME, BYTES/1024/1024 AS SIZE_MB FROM DBA_DATA_FILES ORDER BY TABLESPACE_NAME;
```

### 3.5 Drop a tablespace

Only use this for a tablespace created specifically for practice and after confirming that its contents are no longer needed:

```sql
DROP TABLESPACE LIBRARY_TS INCLUDING CONTENTS AND DATAFILES;
```

## 4. System privileges and object privileges

**System privileges** allow a user to perform database-level actions, such as creating a table or connecting to the database.

**Object privileges** allow a user to perform actions on a particular object, such as selecting from or inserting into a table.

### 4.1 System privilege examples

```sql
GRANT CREATE SESSION TO LIB_STAFF;
GRANT CREATE SESSION TO LIB_VIEWER;
GRANT CREATE TABLE TO LIB_ADMIN;
```

Revoke a system privilege:

```sql
REVOKE CREATE TABLE FROM LIB_ADMIN;
```

### 4.2 Prepare sample library objects as LIB_ADMIN

Connect as `LIB_ADMIN` and create these small teaching tables if they do not already exist. They are simplified security-practical tables and are not a replacement for the full Week 3 schema.

```sql
CREATE TABLE BOOK (BOOK_ID NUMBER PRIMARY KEY, TITLE VARCHAR2(100), AVAILABLE_COPIES NUMBER);
CREATE TABLE MEMBER (MEMBER_ID NUMBER PRIMARY KEY, MEMBER_NAME VARCHAR2(100), EMAIL VARCHAR2(100));
INSERT INTO BOOK VALUES (201,'Database Systems',5);
INSERT INTO BOOK VALUES (202,'Learning Oracle SQL',3);
INSERT INTO MEMBER VALUES (101,'Arun','arun@example.com');
INSERT INTO MEMBER VALUES (102,'Meena','meena@example.com');
COMMIT;
```

If these tables already exist, do not repeat the `CREATE TABLE` statements. Use `DESC BOOK;` and `SELECT * FROM BOOK;` to inspect them.

### 4.3 Grant object privileges

Run as `LIB_ADMIN`, the owner of the tables:

```sql
GRANT SELECT ON BOOK TO LIB_STAFF;
GRANT SELECT, INSERT, UPDATE ON BOOK TO LIB_STAFF;
GRANT SELECT ON MEMBER TO LIB_VIEWER;
```

### 4.4 Revoke object privileges

Run as the object owner or an appropriately authorized administrator:

```sql
REVOKE UPDATE ON BOOK FROM LIB_STAFF;
REVOKE INSERT ON BOOK FROM LIB_STAFF;
```

## 5. Roles

A role is a named collection of privileges. Roles simplify privilege management when several users need similar access.

### 5.1 Create roles

Run as `LIB_ADMIN` after it has received `CREATE ROLE`:

```sql
CREATE ROLE LIB_STAFF_ROLE;
CREATE ROLE LIB_VIEWER_ROLE;
```

### 5.2 Grant privileges to roles

Run as `LIB_ADMIN`, the owner of the tables:

```sql
GRANT SELECT, INSERT, UPDATE ON BOOK TO LIB_STAFF_ROLE;
GRANT SELECT ON MEMBER TO LIB_VIEWER_ROLE;
```

### 5.3 Grant roles to users

Run as an administrator with authority to grant these roles:

```sql
GRANT LIB_STAFF_ROLE TO LIB_STAFF;
GRANT LIB_VIEWER_ROLE TO LIB_VIEWER;
```

### 5.4 Revoke a role

```sql
REVOKE LIB_STAFF_ROLE FROM LIB_STAFF;
```

### 5.5 View roles and privileges

```sql
SELECT ROLE FROM DBA_ROLES ORDER BY ROLE;
SELECT GRANTEE, GRANTED_ROLE FROM DBA_ROLE_PRIVS WHERE GRANTEE LIKE 'LIB_%' ORDER BY GRANTEE;
SELECT GRANTEE, PRIVILEGE FROM DBA_SYS_PRIVS WHERE GRANTEE LIKE 'LIB_%' ORDER BY GRANTEE, PRIVILEGE;
SELECT GRANTEE, OWNER, TABLE_NAME, PRIVILEGE FROM DBA_TAB_PRIVS WHERE GRANTEE LIKE 'LIB_%' ORDER BY GRANTEE, TABLE_NAME;
```

## 6. Test the permissions

Connect as `LIB_STAFF`:

```sql
CONNECT LIB_STAFF/LibStaff123@localhost:1521/XEPDB1
SELECT * FROM LIB_ADMIN.BOOK;
```

If `SELECT` has been granted, the book rows should be displayed. If the role or object privilege has been revoked, Oracle may return `ORA-00942` or a privilege-related error depending on the statement and object visibility.

Connect as `LIB_VIEWER`:

```sql
CONNECT LIB_VIEWER/LibView123@localhost:1521/XEPDB1
SELECT * FROM LIB_ADMIN.MEMBER;
```

A viewer with only `SELECT` permission should be able to read the member table but should not be able to insert or update its rows.

## 7. Database security practices

- Give each person or application a separate database account.
- Grant only the privileges required for the user's work.
- Prefer roles for managing groups of privileges.
- Do not share administrator passwords.
- Use strong passwords and change default or classroom passwords before real deployment.
- Lock accounts that should not be used.
- Avoid granting broad privileges such as `DBA` to ordinary users.
- Back up important data before destructive operations.

## 8. Week 7 practical exercises

1. Create three users named `LIB_ADMIN`, `LIB_STAFF`, and `LIB_VIEWER`.
2. Grant `CREATE SESSION` to all three users.
3. Grant table and view creation privileges to `LIB_ADMIN`.
4. Create a tablespace named `LIBRARY_TS` using a valid local datafile path.
5. Create a role named `LIB_READ_ROLE` and grant `SELECT` on `LIB_ADMIN.BOOK` to it.
6. Grant the role to `LIB_VIEWER` and test the permission.
7. Revoke the role and test the permission again.
8. Lock and unlock `LIB_VIEWER`.

---
