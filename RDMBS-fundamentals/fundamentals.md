# Relational Datbase Management System (RDBMS)

## Learn Standard SQL Programming Language

#### Basic syntax

- SQL Keywoards
- Data types
- Operators
- Statements: SELECT, INSERT, DELETE, UPDATE, ...

#### Data Definition language(DDL)

- Create table
- Drop table
- Alter table
- Truncate table

#### Data Manipulation Language(DML)

- SELECT - columns
- FROM - tables
- WHERE - filter rows
- JOINs - data from tow or more tables
- GROUP BY - used along aggregate fucntions
- ORDER BY - sort rows
- HAVINGS - filter rows of aggregated result
- INSERT - add new data in table
- UPDATE - column data for specific row(s)
- DELETE - delete specific row(s)

#### Aggregate functions / Queries

- SUM - calculate the total of a set of values
- COUNT - returns the number of rows that match the specified criteria
- AVG / MIN / MAX - returns the averate, minimum and highest value in a set of values
- GROUP BY - arrange identical data into groups from the aggregated result
- HAVING - filter the results of aggregate functions

#### Data Constraints

- PRIMARY KEY
- FOREIGN KEY
- UNIQUE
- NOT NULL
- CHECK
- DEFAULT
- INDEX

#### JOIN tables

- INNTER JOIN
- LEFT JOIN
- RIGHT JOIN
- OUTER JOIN
- SELF JOIN
- CROSS JOIN

#### Sub Quries(subqueries)

- Nest subquery
- Correlated subquery

Types of subquery

- Scolar
- Column
- Row
- Table

#### String functions

- CONCAT
- LENGTH
- REPLACE
- SUBSTRING
- UPPER
- LOWER

#### Numeric functions

- FLOOR
- ABS
- MOD
- ROUND
- CEILING

#### Date and Time

- DATE
- TIME
- TIMESTAMP
- DATEPART
- DATEADD

#### Conditionals

- CASE
- NULLIF
- COALESCE

#### Views

- CREATE VIEW
- UPDATE VIEW
- DELETE VIEW

#### Indexes

Indexes improve the speed of data retrieval operations on database tables by providing a quick lookup mechanism for finding rows with specific column values instead of scanning the entire table.

- Managing indexes; create / update / indexes:
  Managing indexes in SQL involves creating, modifying, and dropping indexes to optimize database performance. This process includes identifying columns that benefit from indexing (frequently queried or used in JOIN conditions), creating appropriate index types (e.g., single-column, composite, unique), and regularly analyzing index usage and effectiveness.

- Query optimization:
  Query optimization is the process of selecting the most efficient way to execute a SQL query. It involves the database management system (DBMS) analyzing different possible execution plans for a query and choosing the one that will return the results fastest, using the least amount of resources.

#### Transaction

A transaction is a logical unit of work that groups one or more SQL statements into an all-or-nothing operation.
It guarantees four properties of ACID :

- Atomicity: All or nothing – partial failures are rolled back.
- Consistency: The database moves from one valid state to another (e.g., no negative stock).
- Isolation: Concurrent transactions don’t interfere (as if they run serially).
- Durability: Committed changes survive crashes (written to disk).

Commands:
`BEGIN` (or START TRANSACTION) Start a transaction block.
`COMMIT` Permanently save all changes made inside the transaction.
`ROLLBACK` Abort the transaction, discarding all changes.
`SAVEPOINT` Create a marker within a transaction for partial rollback.

- Example of simple BEGIN - COMMIT

  ```sql
  BEGIN;  -- Start transaction

  -- Step 1: Deduct from customer balance
  UPDATE customers
  SET balance = balance - 50
  WHERE id = 1;

  -- Step 2: Reduce stock
  UPDATE books
  SET stock = stock - 1
  WHERE id = 101;

  -- Step 3: Create order record
  INSERT INTO orders (customer_id, book_id)
  VALUES (1, 101);

  -- If all went well
  COMMIT;
  ```

- Simulating a failure (rollback)

  ```sql
  BEGIN;

  UPDATE customers SET balance = balance - 50 WHERE id = 1;  -- Works
  UPDATE books SET stock = stock - 1 WHERE id = 101;        -- Works
  -- Oops, we forgot to insert the order! Or an error occurs.

  ROLLBACK;  -- Undoes both updates. Alice’s money and stock are restored.
  ```

- Example with SAVEPOINT
  Sometimes you want to roll back only part of a transaction. For example, you try to update stock, but if it fails, you still want to log an error without aborting everything.

  ```sql
  BEGIN;

  UPDATE customers SET balance = balance - 50 WHERE id = 1;

  SAVEPOINT before_stock_update;

  UPDATE books SET stock = stock - 1 WHERE id = 101;

  -- Suppose this update fails (e.g., stock becomes -1 due to a constraint)
  -- We can rollback only to the savepoint:
  ROLLBACK TO SAVEPOINT before_stock_update;

  -- Now log the failure, but keep the customer debit?
  -- Actually, we probably shouldn't keep the debit. Let's rollback entirely:
  ROLLBACK;
  ```

### Stored Procedures and Functions

Stored procedures and functions are precompiled database objects that encapsulate a set of SQL statements and logic. Stored procedures can perform complex operations and are typically used for data manipulation, while functions are designed to compute and return values. Both improve performance by reducing network traffic and allowing code reuse. They also enhance security by providing a layer of abstraction between the application and the database.

```sql
CREATE OR REPLACE FUNCTION purchase_book(
    p_customer_id INT,
    p_book_id INT
)
RETURNS TEXT AS $$
DECLARE
    book_price DECIMAL(10,2);
BEGIN
    -- Start an implicit transaction (function runs inside a transaction)

    -- Get price
    SELECT price INTO book_price FROM books WHERE id = p_book_id;

    -- Deduct from customer
    UPDATE customers SET balance = balance - book_price WHERE id = p_customer_id;

    -- Reduce stock
    UPDATE books SET stock = stock - 1 WHERE id = p_book_id;

    -- Insert order
    INSERT INTO orders (customer_id, book_id) VALUES (p_customer_id, p_book_id);

    RETURN 'Order placed successfully';

EXCEPTION
    WHEN check_violation OR foreign_key_violation THEN
        -- Automatically rolls back the transaction
        RETURN 'Failed: ' || SQLERRM;
END;
$$ LANGUAGE plpgsql;
```

### Performance Optimization

Performance optimization in SQL focuses on making your queries run faster and more efficiently. This involves techniques like using indexes to speed up data retrieval, rewriting queries for better performance, and understanding how the database engine executes your SQL code. It's about ensuring your database can handle large amounts of data and complex queries without slowing down.

### Choose one RDBMS (Go with PostgreSQL) and learn it's SQL dailect
