---
layout: post
title: "Oracle SQL notes #1"
date: 2026-02-10 10:00:00 +0200
categories: technology, notes
---


## The Building Blocks: Data Definition Language (DDL)

Data Definition Language (DDL) is the foundation of any SQL database. It allows you to create and manage the structure of your database objects.

### Creating Tables

The most fundamental object in a database is the `TABLE`, a basic unit of storage composed of rows and columns. You can create a table using the `CREATE TABLE` statement:

```sql
CREATE TABLE employees (
    id NUMBER(7) PRIMARY KEY,
    last_name VARCHAR2(25) NOT NULL,
    email VARCHAR2(25) UNIQUE,
    salary NUMBER(8,2) CHECK (salary > 0),
    department_id NUMBER(7) REFERENCES departments(id)
);
```

### Constraints: Enforcing Data Integrity

Constraints enforce rules on the data in a table, ensuring the consistency and integrity of your database.

-   **PRIMARY KEY**: Uniquely identifies each record in a table.
-   **NOT NULL**: Ensures that a column cannot have a NULL value.
-   **UNIQUE**: Ensures that all values in a column are different.
-   **CHECK**: Ensures that all values in a column satisfy a specific condition.
-   **FOREIGN KEY**: Establishes a link between two tables to enforce referential integrity.

You can define what happens when a referenced row in a parent table is deleted or updated using `ON DELETE CASCADE` (deletes dependent rows) or `ON DELETE SET NULL` (sets dependent foreign key values to null).

### Modifying and Removing Tables

Over time, you may need to modify your table structures. The `ALTER TABLE` statement is a powerful tool for this:

```sql
-- Add a new column
ALTER TABLE employees ADD (job_id VARCHAR2(20));

-- Modify an existing column
ALTER TABLE employees MODIFY (last_name VARCHAR2(30));

-- Drop a column
ALTER TABLE employees DROP COLUMN job_id;
```

If you no longer need a table, you can remove it with the `DROP TABLE` statement. To permanently delete the table and free up space immediately, use the `PURGE` option.

```sql
DROP TABLE employees PURGE;
```

## Manipulating Data: Data Manipulation Language (DML)

Data Manipulation Language (DML) is used to manage data within your schema objects.

-   **INSERT**: Adds new rows of data to a table.
    ```sql
    INSERT INTO employees (id, last_name, email, salary, department_id)
    VALUES (101, 'King', 'king@example.com', 24000, 90);
    ```
-   **UPDATE**: Modifies existing data in a table.
    ```sql
    UPDATE employees
    SET salary = 25000
    WHERE id = 101;
    ```
-   **DELETE**: Removes existing rows from a table.
    ```sql
    DELETE FROM employees
    WHERE id = 101;
    ```
Unlike DDL statements, DML statements are not auto-committed, meaning you can roll back your changes if you make a mistake.

## Advanced Schema Objects

Beyond tables, SQL offers several other objects that help you manage and access your data more effectively.

### Views: A Window into Your Data

A **View** is a stored query that you can treat as a table. Views are useful for:
-   Restricting data access by showing only specific rows or columns.
-   Simplifying complex queries.
-   Providing a consistent structure even if the underlying tables change.

```sql
CREATE OR REPLACE VIEW high_salary_employees AS
SELECT id, last_name, salary
FROM employees
WHERE salary > 15000;
```

The `WITH CHECK OPTION` clause can be added to a view to ensure that any `INSERT` or `UPDATE` operations performed through the view adhere to the conditions in the `WHERE` clause.

### Sequences: Generating Unique Numbers

A **Sequence** is a shareable object used to generate unique numbers automatically, commonly used for primary key values.

```sql
CREATE SEQUENCE employee_seq
START WITH 200
INCREMENT BY 1;

INSERT INTO employees (id, last_name)
VALUES (employee_seq.nextval, 'Smith');
```

### Indexes: Speeding Up Your Queries

An **Index** is a performance-tuning method for allowing faster retrieval of records. Indexes are automatically created on `PRIMARY KEY` and `UNIQUE` columns, but you can also create them manually.

-   **B-Tree Indexes**: Ideal for columns with high cardinality (many distinct values) and for retrieving small subsets of data. They are common in Online Transaction Processing (OLTP) systems.
-   **Bitmap Indexes**: Best for columns with low cardinality (few distinct values) and are often used in data warehousing applications for complex queries on large datasets.

```sql
CREATE INDEX emp_last_name_idx ON employees(last_name);
```

### Synonyms: Creating Aliases for Objects

A **Synonym** is an alternative name for a database object, which can simplify access and hide the underlying schema and object names.

```sql
CREATE SYNONYM emps FOR employees;
```

## Advanced Querying Techniques

As your data grows, you'll need more advanced techniques to retrieve the information you need.

### Subqueries: Queries Within Queries

A subquery is a `SELECT` statement nested inside another statement. They can be very powerful for performing multi-step queries.

A **scalar subquery** returns a single value and can be used in various clauses of a `SELECT` statement.

### Common Table Expressions (CTEs) with the `WITH` Clause

The `WITH` clause, also known as a Common Table Expression (CTE), allows you to define a temporary, named result set that you can reference within your main query. CTEs improve readability and can enhance performance by storing the results of a query block.

```sql
WITH department_summary AS (
    SELECT
        d.department_name,
        SUM(e.salary) AS total_salary
    FROM employees e
    JOIN departments d ON e.department_id = d.id
    GROUP BY d.department_name
)
SELECT department_name, total_salary
FROM department_summary
WHERE total_salary > 50000;
```

## Managing User Access

Controlling who can do what in your database is crucial for security.

-   **System Privileges**: These grant the ability to perform actions at the database level, such as `CREATE TABLE` or `CREATE SESSION`.
-   **Object Privileges**: These grant permissions to manipulate the content of specific database objects, such as `SELECT` or `UPDATE` on a table.

You can grant privileges to users and also allow them to pass those privileges on to others using the `WITH GRANT OPTION`.

```sql
GRANT UPDATE ON employees TO another_user WITH GRANT OPTION;
```

Privileges can be removed using the `REVOKE` statement.

## Handling Time Zones

SQL provides several functions to manage date and time information, which is especially important when dealing with data from different time zones.

-   `CURRENT_DATE`: Returns the current date from the user session.
-   `CURRENT_TIMESTAMP`: Returns the current date and time with time zone information.
-   `LOCALTIMESTAMP`: Returns the current date and time without time zone information.
-   `SYSDATE`: Returns the date and time of the database server.

You can set the time zone for a session using `ALTER SESSION SET TIME_ZONE`.

By understanding and utilizing these SQL features, you can build robust, secure, and efficient databases to power your applications.
