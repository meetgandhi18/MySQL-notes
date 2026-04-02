# MySQL Learning Notes (Enhanced + Examples)

## SQL Execution Order

1.  FROM → Load data
2.  JOIN → Combine tables
3.  WHERE → Filter rows
4.  GROUP BY → Group data
5.  HAVING → Filter groups
6.  WINDOW FUNCTIONS
7.  SELECT → Choose columns
8.  DISTINCT → Remove duplicates
9.  ORDER BY → Sort
10. LIMIT → Restrict output

------------------------------------------------------------------------

## JOINS

### INNER JOIN

Returns matching rows

``` sql
SELECT e.name, d.name 
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;
```

### LEFT JOIN

All left + matched right

``` sql
SELECT e.name, d.name 
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;
```

### RIGHT JOIN

All right + matched left

``` sql
SELECT e.name, d.name 
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

### CROSS JOIN

m × n rows

```sql
SELECT e.name, d.name 
FROM employees e
CROSS JOIN departments d ON e.department_id = d.id;
```

------------------------------------------------------------------------

## WHERE

Filters rows before grouping

``` sql
SELECT * FROM employees WHERE salary > 50000;
```

------------------------------------------------------------------------

## GROUP BY + HAVING

``` sql
SELECT department_id, COUNT(*) 
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 2;
```

------------------------------------------------------------------------

## Window Functions (PARTITION BY)

👉 Window Functions perform calculations across a set of rows (window) but do NOT collapse rows like GROUP BY.

💡 Think:
- “GROUP BY → merges rows”
- “WINDOW FUNCTION → keeps rows, adds extra info”

🔹 What is a Window?

👉 A window is a subset of rows defined by:
- PARTITION BY → grouping
- ORDER BY → ordering within group

🔹 PARTITION BY (Core Concept)

👉 PARTITION BY divides data into groups (like GROUP BY),

BUT instead of merging rows, it keeps all rows intact.

Example:
``` sql
SELECT name, department_id,
COUNT(*) OVER(PARTITION BY department_id) AS total
FROM employees;
```

OUTPUT:-
| name | department_id | total |
| ---- | ------------- | ----- |
| A    | 1             | 3     |
| B    | 1             | 3     |
| C    | 1             | 3     |
| D    | 2             | 2     |
| E    | 2             | 2     |

👉 Each row still exists, but gets extra calculated value

🔹 Syntax of Window Function
```sql
FUNCTION_NAME() OVER (
    PARTITION BY column
    ORDER BY column
)
```


### 🔥 Types of Window Functions

1. Aggregate Window Functions

👉 Same as aggregate, but don’t collapse rows

```sql
SUM(salary) OVER(PARTITION BY department_id)
AVG(salary) OVER(PARTITION BY department_id)
MAX(salary) OVER(PARTITION BY department_id)
MIN(salary) OVER(PARTITION BY department_id)
COUNT(*) OVER(PARTITION BY department_id)
```

2. Ranking Functions

- ROW_NUMBER()

👉 Unique row number (no ties)
```sql
SELECT name, salary,
ROW_NUMBER() OVER(ORDER BY salary DESC) AS rn
FROM employees;
```

- RANK()

👉 Same rank for ties, gaps exist
```sql
RANK() OVER(ORDER BY salary DESC)
```

- DENSE_RANK()

👉 Same rank for ties, NO gaps
```sql
DENSE_RANK() OVER(ORDER BY salary DESC)
```

3. Value Functions

- LAG()

👉 Access previous row
```sql
SELECT name, salary,
LAG(salary) OVER(ORDER BY salary) AS prev_salary
FROM employees;
```

- LEAD()

👉 Access next row

```sql
LEAD(salary) OVER(ORDER BY salary)
```

- FIRST_VALUE()

👉 First value in partition

```sql
FIRST_VALUE(salary) OVER(PARTITION BY department_id ORDER BY salary DESC)
```

- LAST_VALUE()

👉 Last value in partition
```sql
LAST_VALUE(salary) OVER(PARTITION BY department_id ORDER BY salary DESC)
```

🔹 PARTITION BY vs GROUP BY
| Feature     | GROUP BY          | PARTITION BY        |
| ----------- | ----------------- | ------------------- |
| Rows        | Collapses rows    | Keeps rows          |
| Output      | One row per group | Same number of rows |
| Use case    | Aggregation       | Analytics           |
| Flexibility | Limited           | Very powerful       |


🔹 Example Comparison

GROUP BY
```sql
SELECT department_id, COUNT(*)
FROM employees
GROUP BY department_id;
```

👉 Output:
| department_id | count |
| ------------- | ----- |
| 1             | 3     |
| 2             | 2     |


PARTITION BY
```sql
SELECT name, department_id,
COUNT(*) OVER(PARTITION BY department_id)
FROM employees;
```

👉 Output:
All rows + count column 

🔹 Advantages of Window Functions
- ✅ Keeps original rows intact
- ✅ Powerful analytics (ranking, running totals)
- ✅ No need for complex subqueries
- ✅ Better readability

🔹 Disadvantages
- ❌ Slightly complex to understand
- ❌ Can be slower on large datasets without indexing
- ❌ Requires MySQL 8+

### Where we can write window function(Partition by)

#### ✅ 1. SELECT Clause
```sql
SELECT name,
       department_id,
       COUNT(*) OVER (PARTITION BY department_id) AS total_emp
FROM employees;
```

------------------------------------------------------------------------

## Subqueries

👉 A Subquery is a query written inside another query.

💡 Think:
- "Query inside a query to get intermediate result"   

🔹 Why We Use Subqueries?   
- To break complex queries into smaller parts
- To filter data based on another query
- To compare values dynamically
- To avoid hardcoding values

### 🔹 Types of Subqueries (Based on Placement)

#### 1. Subquery in WHERE Clause ✅ (Most Common)

👉 Used for filtering data

```sql
SELECT name 
FROM employees 
WHERE department_id IN (SELECT id FROM departments);
```

👉 Inner query runs first → outer query uses result

#### 2. Subquery in SELECT Clause

👉 Used to return calculated values per row

```sql
SELECT name,
(SELECT AVG(salary) FROM employees) AS avg_salary
FROM employees;
```

👉 Adds extra column using subquery

#### 3. Subquery in FROM Clause (Derived Table)

👉 Subquery acts like a temporary table

```sql
SELECT dept_id, avg_sal
FROM (
    SELECT department_id AS dept_id, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department_id
) AS temp;
```

### 🔹 Types of Subqueries (Based on Behavior)

#### 1. Scalar Subquery

👉 Returns single value

```sql
SELECT name 
FROM employees 
WHERE salary > (SELECT AVG(salary) FROM employees);
```

#### 2. Multi-row Subquery

👉 Returns multiple values

```sql
WHERE department_id IN (SELECT id FROM departments);
```

#### 3. Correlated Subquery

👉 Runs once for each row of outer query

```sql
SELECT name 
FROM employees e
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE department_id = e.department_id
);
```

💡 Slower but powerful


👉 Useful for complex transformations

### IN

👉 Checks if value matches any value in list

``` sql
SELECT name FROM employees 
WHERE department_id IN (SELECT id FROM departments);
```

✔️ Use When:
- Subquery returns small dataset
- Simple matching required

### NOT IN

👉 Opposite of IN

``` sql
SELECT name FROM employees 
WHERE department_id NOT IN (SELECT id FROM departments);
```

⚠️ Important Problem:

👉 If subquery contains NULL → result becomes EMPTY

💡 Reason:

SQL cannot compare NULL properly

### EXISTS

👉 Checks if at least one row exists

``` sql
SELECT name FROM employees e
WHERE EXISTS (
    SELECT 1 FROM departments d 
    WHERE d.id = e.department_id
);
```
✔️ Use When:
- Subquery is large
- Performance matters
- Correlated queries

💡 Stops execution as soon as match found (fast)

### NOT EXISTS

👉 Returns rows where no match exists

``` sql
SELECT name FROM employees e
WHERE NOT EXISTS (
    SELECT 1 FROM departments d 
    WHERE d.id = e.department_id
);
```
✔️ Advantage:
- Safe with NULL
- More reliable than NOT IN

🔹IN vs EXISTS
| Feature     | IN                  | EXISTS           |
| ----------- | ------------------- | ---------------- |
| Execution   | Compares values     | Checks existence |
| Performance | Slow for large data | Faster           |
| NULL issue  | Yes                 | No               |
| Best for    | Small datasets      | Large datasets   |

🔹 NOT IN vs NOT EXISTS
| Feature       | NOT IN    | NOT EXISTS |
| ------------- | --------- | ---------- |
| NULL Handling | ❌ Problem | ✅ Safe     |
| Reliability   | Low       | High       |
| Recommended   | Avoid     | Use        |

🔹 Execution Flow

👉 Subquery executes first:

```sql
SELECT name 
FROM employees 
WHERE department_id IN (SELECT id FROM departments);
```

Execution:
- Run subquery → get department ids
- Outer query uses result

------------------------------------------------------------------------

## ANY

Condition true for ANY value

``` sql
SELECT name FROM employees
WHERE salary > ANY (SELECT salary FROM employees WHERE department_id = 1);
```

👉 Greater than at least one value

## ALL

Condition must satisfy ALL values

``` sql
SELECT name FROM employees
WHERE salary > ALL (SELECT salary FROM employees WHERE department_id = 1);
```

👉 Greater than every value

------------------------------------------------------------------------

## Aggregate Functions

Aggregate Functions are used to perform calculations on a set of rows and return a single value.

"Many Roes" -> "One Result"

Why We use it?

- To summarize data
- To generate insights
- To perform calculations on groups

Important Rule
- Aggregate functions ignore NULL values (except COUNT(*))

Types of Aggregatin Function :-

### COUNT() => Counts number of rows 

Types:
- COUNT(*) -> counts all rows (including NULL)
- COUNT(column) → ignores NULL values

Example:-
```sql
SELECT COUNT(*) FROM employees;
```

👉 Total number of employees

```sql
SELECT COUNT(salary) FROM employees;
```

👉 Counts only rows where salary is NOT NULL


### SUM() => Adds all values of a column

Example:-
```sql
SELECT SUM(salary) FROM employees;
```

- Total salary of all employees
- NULL values are ignored

### AVG() => Calculates average (mean)

Example:-
```sql
SELECT AVG(salary) FROM employees;
```

### MAX() => Returns highest value

Example:-
```sql
SELECT MAX(salary) FROM employees;
```

👉 Highest salary

💡 Works on:
- Numbers
- Dates
- Strings (lexicographically)

### MIN() => Returns smallest value

Example:-
```sql
SELECT MIN(salary) FROM employees;
```

👉 Lowest salary

### Aggregate functions are mostly used with GROUP BY

Example:
```sql
SELECT department_id, AVG(salary)
FROM employees
GROUP BY department_id;
```

👉 Gives average salary per department

- 🔹 Using Multiple Aggregates
```sql
SELECT 
    COUNT(*) AS total_employees,
    SUM(salary) AS total_salary,
    AVG(salary) AS avg_salary,
    MAX(salary) AS highest_salary,
    MIN(salary) AS lowest_salary
FROM employees;
```

### ⚠️ Important Interview Points

1. Cannot use in WHERE ❌
```sql
SELECT * FROM employees WHERE AVG(salary) > 50000; ❌
```

👉 Use HAVING instead:
```sql
SELECT department_id
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 50000;
```

2. GROUP BY Rule

👉 Every non-aggregated column must be in GROUP BY

3. NULL Handling

- Ignored in SUM, AVG, COUNT(column)
- Included in COUNT(*)

------------------------------------------------------------------------

##  Indexes

👉 Indexes are special data structures used to speed up data retrieval in MySQL.

💡 Think:
- “Index = shortcut to find data faster instead of scanning entire table”

🔹 Why Indexes are Used?
- To improve query performance
- To avoid full table scan
- To speed up: WHERE, JOIN, ORDER BY, GROUP BY

🔹 How Index Works Internally

👉 MySQL uses B-Tree (Balanced Tree)
- Data is stored in sorted structure
- Search becomes O(log n) instead of O(n)

💡 Without index:
```sql
SELECT * FROM employees WHERE salary = 50000;
```
👉 Full table scan (slow)

💡 With index:
```sql
CREATE INDEX idx_salary ON employees(salary);
SELECT * FROM employees WHERE salary = 50000;
```
👉 Direct lookup (fast)

### 🔥 Types of Indexes

#### 1. Primary Index Or Clustered Index 

👉 Created automatically on PRIMARY KEY

Theory:
- Unique + NOT NULL
- Only one per table
- Stores data in sorted order (clustered in InnoDB)

Example:

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);
```

#### 2. Unique Index

👉 Ensures all values are unique

Theory:
- No duplicate values allowed
- Can allow NULL (MySQL allows multiple NULLs)

Example:
```SQL
CREATE UNIQUE INDEX idx_email ON users(email);
```

#### 3. Normal (Non-Unique) Index Or Secondary Index

👉 Used for performance optimization

Theory:
- Allows duplicate values
- Speeds up search/filter operations

```sql
CREATE INDEX idx_salary ON employees(salary);
```

#### 4. Composite Index (Multi-column Index)

👉 Index on multiple columns

Theory:
- Follows Leftmost Prefix Rule
- Works only when first column is used

Example:
```sql
CREATE INDEX idx_name_dept ON employees(name, department_id);
```

Works:
```sql
WHERE name = 'John'
WHERE name = 'John' AND department_id = 2
```

Not efficient:
```sql
WHERE department_id = 2
```

#### 5. Full-Text Index

👉 Used for searching large text data

Theory:
- Works with MATCH() AGAINST()
- Faster than LIKE for large text

Example:    
```sql
CREATE FULLTEXT INDEX idx_desc ON articles(content);

SELECT * FROM articles
WHERE MATCH(content) AGAINST('mysql');
```

#### 🔥 When to Use Index

✅ Use on:

- WHERE conditions
- JOIN columns
- ORDER BY / GROUP BY
- High cardinality columns

#### 🔥 When NOT to Use Index

❌ Avoid:

- Small tables
- Low unique values (gender, status)
- Heavy write operations

#### 🔥 Advantages

- Faster SELECT queries
- Efficient joins
- Better performance on large data

#### 🔥 Disadvantages

- Slower INSERT/UPDATE/DELETE
- Extra storage required
- Maintenance overhead
------------------------------------------------------------------------
##  Normalization

👉 Normalization is the process of organizing data in a database to:
- Reduce data redundancy
- Improve data integrity
- Avoid anomalies (insert/update/delete issues)

💡 Think:
- “Break big messy table → into small structured tables”

#### 🔥 Why Normalization?

Without normalization:
- Duplicate data ❌
- Update issues ❌
- Inconsistent data ❌

With normalization:
- Clean structure ✅
- Less redundancy ✅
- Easy maintenance ✅

#### 🔥 1NF (First Normal Form)

👉 Rule:
- No repeating groups
- Each column should have atomic (single) values

❌ Before (Not 1NF)
| id | name | skills    |
| -- | ---- | --------- |
| 1  | John | Java, PHP |

👉 Multiple values in one column ❌

✅ After (1NF)
| id | name | skill |
| -- | ---- | ----- |
| 1  | John | Java  |
| 1  | John | PHP   |

💡 Each cell contains single value

#### 🔥 2NF (Second Normal Form)

👉 Rule:
- Must be in 1NF
- Remove partial dependency

💡 Partial dependency = column depends on part of composite key

❌ Before (Not 2NF)
| student_id | course_id | student_name |
| ---------- | --------- | ------------ |

👉 student_name depends only on student_id ❌

✅ After (2NF)

- Students Table

| student_id | student_name |
| ---------- | --------- |

- Courses Table

| course_id |
| ---------- |

- Enrollment Table

| student_id | course_id |
| ---------- | --------- |

💡 Now every column depends on full primary key

#### 🔥 3NF (Third Normal Form)

👉 Rule:
- Must be in 2NF
- Remove transitive dependency

💡 Transitive dependency:
- A → B → C (indirect dependency)

❌ Before (Not 3NF)

| emp_id | dept_id | dept_name |

👉 dept_name depends on dept_id (not directly on emp_id) ❌

✅ After (3NF)

Employees Table

| emp_id | dept_id |

Departments Table

| dept_id | dept_name |

💡 Removed indirect dependency

#### 🔥 BCNF (Boyce-Codd Normal Form)

👉 Advanced version of 3NF

Rule:
- Every determinant must be a candidate key

❌ Problem Case (3NF but not BCNF)
| teacher | subject | room |
| ------- | ------- | ---- |

👉 Suppose:
- teacher → subject
- subject → room

👉 Here:
- subject is NOT a candidate key ❌

✅ After BCNF

Split into:

Teacher Table

| teacher | subject |

Subject Table

| subject | room |

💡 Removes complex dependency issues

| Normal Form | Removes                    |
| ----------- | -------------------------- |
| 1NF         | Repeating groups           |
| 2NF         | Partial dependency         |
| 3NF         | Transitive dependency      |
| BCNF        | Advanced dependency issues |


------------------------------------------------------------------------

##  Prepared Statements

👉 Prepared Statements are SQL queries that are precompiled and stored by the database, and later executed multiple times with different values.

💡 Think:
- “Write query once → reuse it with different inputs”

🔹 Why Use Prepared Statements?
- Improve performance (query compiled once)
- Prevent SQL Injection (safe parameter binding)
- Reuse queries efficiently

🔹 How It Works (Step-by-Step)

- Prepare → SQL query is parsed and compiled
- Bind Parameters → Values are attached safely
- Execute → Query runs with given values

🔹 Syntax in MySQL
```sql
PREPARE stmt FROM 'SELECT * FROM employees WHERE id = ?';

SET @id = 1;

EXECUTE stmt USING @id;

DEALLOCATE PREPARE stmt;
```

🔹 Example (Multiple Executions)
```sql
PREPARE stmt FROM 'SELECT * FROM employees WHERE salary > ?';

SET @salary = 30000;
EXECUTE stmt USING @salary;

SET @salary = 50000;
EXECUTE stmt USING @salary;
```
👉 Same query reused with different values ✅

🔹 Key Concept (Parameter Placeholder)

👉 ? is a placeholder for values
- Values are bound later
- Prevents direct string injection

🔹 Security Advantage (SQL Injection Prevention)

❌ Without Prepared Statement (Danger)  
```sql
SELECT * FROM users WHERE username = 'admin' AND password = '123';
```

👉 User can inject malicious SQL

✅ With Prepared Statement (Safe)
```sql
SELECT * FROM users WHERE username = ? AND password = ?
```

👉 Values treated as data, not SQL code

🔹 Where It Is Used
- Backend development (PHP, Node.js, Java, etc.)
- APIs handling user input
- Repeated queries (loops, batch processing)

🔹 Advantages
- ✅ Faster execution (compiled once)
- ✅ Prevents SQL Injection
- ✅ Cleaner and reusable code
- ✅ Efficient for repeated queries

🔹 Disadvantages
- ❌ Slight overhead for single execution
- ❌ More complex syntax
- ❌ Not useful for very simple one-time queries

------------------------------------------------------------------------

##  Functions vs Procedures

### 🔹 What is a Function?

👉 A Function is a stored program that:
- Takes input
- Performs some logic
- Returns a single value

💡 Think:
- “Function = calculation → gives one result”

✅ Example of Function
```sql
DELIMITER //

CREATE FUNCTION get_bonus(salary INT)
RETURNS INT
DETERMINISTIC
BEGIN
    RETURN salary * 0.10;
END //

DELIMITER ;
```
🔹 Usage:
```sql
SELECT name, salary, get_bonus(salary) AS bonus
FROM employees;
```
👉 Function can be used inside SELECT, WHERE, etc.

🔹 Key Points of Function
- Must return one value
- Can be used inside SQL queries
- Used for calculations / derived values

### 🔹 What is a Procedure?

👉 A Stored Procedure is a program that:
- Performs operations
- Can return multiple values or result sets
- Can contain complex logic

💡 Think:
“Procedure = task/process (like mini program)”

✅ Example of Procedure
```sql
DELIMITER //

CREATE PROCEDURE get_employee(IN emp_id INT)
BEGIN
    SELECT * FROM employees WHERE id = emp_id;
END //

DELIMITER ;
```

🔹 Usage:
```sql
CALL get_employee(1);
```
👉 Returns full row (multiple columns)

🔹 Example with Multiple Outputs
```sql
DELIMITER //

CREATE PROCEDURE get_salary_stats(
    OUT max_sal INT,
    OUT min_sal INT
)
BEGIN
    SELECT MAX(salary), MIN(salary)
    INTO max_sal, min_sal
    FROM employees;
END //

DELIMITER ;
```

🔹 Key Points of Procedure
- Can return multiple values
- Supports IN, OUT, INOUT parameters
- Used for business logic / operations

### 🔥 Function vs Procedure
| Feature      | Function             | Procedure                    |
| ------------ | -------------------- | ---------------------------- |
| Return Value | Single value         | Multiple values / result set |
| Usage        | Inside SELECT, WHERE | Called separately using CALL |
| Parameters   | Only IN              | IN, OUT, INOUT               |
| Purpose      | Calculation          | Full operation / logic       |
| Complexity   | Simple               | Complex                      |
| SQL Usage    | Can be embedded      | Cannot be embedded           |

### 🔹 When to Use What?

Use Function ✅
- Calculations (tax, bonus, discount)
- Returning single value
- Inside queries

Use Procedure ✅
- Complex operations
- Multiple outputs
- Business logic (insert/update workflows)

------------------------------------------------------------------------

##  Scalling (Horizontal V/s Vertical)

### 🔹 What is Database Scaling?    
- Database scaling = handling more data + more users without performance degradation

When your DB becomes slow due to:
- Too many users
- Too many queries
- Too much data
👉 You scale it.

### 🔹 Types of Scaling

There are 2 main types:
1. Vertical Scaling (Scale Up)

2. Horizontal Scaling (Scale Out)

### 🔹 1. Vertical Scaling (Scale Up)

👉 Definition

Increase the power of a single machine
- More RAM
- Faster CPU
- SSD instead of HDD

📌 Example

Your MySQL server:
- Before → 4GB RAM
- After → 32GB RAM

👍 Advantages
- Simple (no architecture change)
- No code change required
- Strong consistency (single DB)

👎 Disadvantages
- Limited (hardware limit)
- Expensive 💸
- Single point of failure

🧠 When to Use?
- Small to medium apps
- Early-stage startups
- Low traffic systems

### 🔹 2. Horizontal Scaling (Scale Out)

👉 Definition

Add multiple machines (servers) instead of upgrading one

📌 Example

Instead of:
- 1 DB Server
You use:
- 5 DB Server

👍 Advantages
- Highly scalable 🚀
- Fault tolerant
- Handles massive traffic

👎 Disadvantages
- Complex
- Requires architecture changes
- Data consistency challenges

🧠 When to Use?
- High traffic apps (Instagram, Amazon)
- Distributed systems

Horizontal scaling has multiple strategies:

### 1. Read Replicas (Master-Slave)

👉 Idea

Split READ and WRITE operations

📌 Architecture

```diagram
        App
       /   \
   Master   Replica1   Replica2
   (Write)   (Read)     (Read)
```

📌 How it works
- Master → handles INSERT, UPDATE, DELETE
- Replicas → handle SELECT queries

👍 Benefits

- Read performance increases ⚡
- Easy to implement

👎 Problems
- Replication lag (data delay)
- Slight inconsistency

### 🔹 2. Database Sharding

👉 Idea

Split data across multiple databases

📌 Example
| User ID   | DB  |
| --------- | --- |
| 1–1000    | DB1 |
| 1001–2000 | DB2 |

📌 Types of Sharding

1. Range-based
```
0–1000 → DB1
1001–2000 → DB2
```

2. Hash-based
```
user_id % 3
```

3. Geo-based
```
India users → DB1
US users → DB2
```

👍 Advantages
- Massive scalability
- Distributes load evenly

👎 Problems
- Complex joins across shards ❌
- Rebalancing data is hard
- Application logic needed

### 🔹 3. Partitioning (Inside One DB)

👉 Difference from Sharding
| Feature    | Partitioning | Sharding     |
| ---------- | ------------ | ------------ |
| Scope      | Single DB    | Multiple DBs |
| Managed by | DB engine    | Application  |

📌 Example
```
Orders table split by year:
2023 → Partition1
2024 → Partition2
```

👍 Benefits
- Faster queries
- Easier maintenance

👎 Limitation
- Still one DB server

### 🔹 4. Caching (Very Important 🔥)

👉 Idea

Store frequently used data in memory

📌 Tools
- Redis
- Memcached

📌 Example

Instead of hitting DB:
```
DB → 1000 queries/sec ❌
```

Use cache:
```
Cache → 900
DB → 100
```

👍 Benefits
- Ultra fast ⚡
- Reduces DB load

👎 Problems
- Cache invalidation is hard
- Data can become stale

### 🔹 5. Connection Pooling

👉 Problem

Opening DB connections is expensive

👉 Solution

Reuse connections

📌 Example

Instead of:
```
1000 users → 1000 connections ❌
```
Use pool:
```
100 connections reused ✅
```

### 🔹 6. Indexing (Micro Scaling)

👉 Idea

Speed up queries using indexes

📌 Example
```sql
CREATE INDEX idx_salary ON employees(salary);
```

👍 Benefit
- Faster SELECT queries

👎 Tradeoff
- Slower INSERT/UPDATE

### 🔹 Real World Scaling Strategy (Step-by-Step)   

When your app grows:

Step 1

👉 Optimize queries + add indexes

Step 2

👉 Add caching (Redis)

Step 3

👉 Read replicas

Step 4

👉 Partition tables

Step 5

👉 Sharding

------------------------------------------------------------------------

## View,CTE,with check view

### 🔹 1. What is a VIEW in MySQL?

A VIEW is a virtual table based on a SQL query.

- 👉 It does NOT store data
- 👉 It stores only the query

🔹 Basic Syntax

```sql
CREATE VIEW view_name AS
SELECT column1, column2
FROM table_name
WHERE condition;
```

🔹 Practical Example

🎯 Tables
```
employees(id, name, salary, department_id)
```

🔹 Create View
```sql
CREATE VIEW high_salary_employees AS
SELECT id, name, salary
FROM employees
WHERE salary > 50000;
```

🔹 Use View
```sql
SELECT * FROM high_salary_employees;
```

👉 Acts like a table!

🔹 Why We Use Views

✅ 1. Simplify Complex Queries

Instead of writing joins every time

✅ 2. Security

Hide sensitive columns

### 🔹 2. View with CHECK OPTION

👉 This ensures that INSERT/UPDATE must follow the view condition

🔹 Syntax
```sql
CREATE VIEW view_name AS
SELECT ...
FROM ...
WHERE condition
WITH CHECK OPTION;
```

🔹 Practical Example 🔥
```sql
CREATE VIEW high_salary_employees AS
SELECT id, name, salary
FROM employees
WHERE salary > 50000
WITH CHECK OPTION;
```

🔹 Valid Insert ✅
```sql
INSERT INTO high_salary_employees (id, name, salary)
VALUES (101, 'Meet', 60000);
```

✔ Works because salary > 50000

🔹 Invalid Insert ❌
```sql
INSERT INTO high_salary_employees (id, name, salary)
VALUES (102, 'Raj', 30000);
```

❌ ERROR — violates condition

### 🔹 3. Types of CHECK OPTION

1️⃣ LOCAL
- Checks only current view
2️⃣ CASCADED (default)
- Checks all underlying views

Example:
```sql
WITH CASCADED CHECK OPTION
```

### 🔹 4. Limitations of Views ⚠️

- ❌ Cannot always update (depends on query)
- ❌ No indexes
- ❌ Complex views = slower performance

### 🔹 5. What is CTE (Common Table Expression)?

- 👉 A CTE is a temporary result set
- 👉 Defined using WITH
- 👉 Exists only during query execution

🔹 Syntax
```sql
WITH cte_name AS (
    SELECT ...
)
SELECT * FROM cte_name;
```

### 🔹 6. Basic CTE Example
```sql
WITH high_salary AS (
    SELECT id, name, salary
    FROM employees
    WHERE salary > 50000
)
SELECT * FROM high_salary;
```

🔥 Why CTE is Powerful

✅ 1. Readability
- Cleaner than subqueries

✅ 2. Reusability
- Use same result multiple times

✅ 3. Recursive Queries support

🔹 7. CTE vs Subquery 🧠

| Feature     | CTE  | Subquery |
| ----------- | ---- | -------- |
| Readability | High | Low      |
| Reuse       | Yes  | No       |
| Recursive   | Yes  | No       |

### 🔹 8. Advanced CTE Example 🔥

🎯 Find employees with above average salary

```sql
WITH avg_salary AS (
    SELECT AVG(salary) AS avg_sal FROM employees
)
SELECT e.name, e.salary
FROM employees e, avg_salary a
WHERE e.salary > a.avg_sal;
```

### 🔹 9. Recursive CTE 🔥🔥

👉 Used for hierarchical data

Example: Employee Hierarchy

```sql
WITH RECURSIVE emp_hierarchy AS (
    -- Base case
    SELECT id, name, manager_id
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive case
    SELECT e.id, e.name, e.manager_id
    FROM employees e
    JOIN emp_hierarchy eh ON e.manager_id = eh.id
)
SELECT * FROM emp_hierarchy;
```

### 🔹 12. Interview Tricky Questions 💡

❓ Can we update a view?
- 👉 YES (only simple views)

❓ Difference between View & Table?
- 👉 View = virtual
- 👉 Table = physical

❓ CTE vs View?
- 👉 CTE = temporary
- 👉 View = permanent

### 🔥 Final Understanding
- View = saved query
- View + Check = controlled data modification
- CTE = temporary + powerful query structuring
------------------------------------------------------------------------

##  DCL (Data control Language)

### 🔐 1. What is DCL?

DCL (Data Control Language) is used to control access to data in the database.

👉 In simple terms:
- DCL decides WHO can do WHAT on WHICH data

### 🎯 Why DCL is Important?

Imagine:

- You have a production database 💰
- Developers should only read data 👀
- Admins can modify everything ⚙️
- Interns should not delete anything ❌

👉 DCL helps you enforce this.

### 🧠 Core Concepts Behind DCL

Before commands, understand these:

1. Users 👤

Database accounts (e.g., meet_user, admin)

2. Privileges 🔑

Permissions like:

- SELECT
- INSERT
- UPDATE
- DELETE
- ALL

3. Objects 📦

Things on which permissions apply:

- Tables
- Views
- Databases
- Procedures

### Types Of Permissions

### Data Permissions (most common)

Used on tables/views:
- SELECT → Read data
- INSERT → Add data
- UPDATE → Modify data
- DELETE → Remove data

Control data access

#### Execution Permissions

EXECUTE → Run stored code

✔ Allows:
- CALL procedure
- Use stored functions

#### Structure Permissions

CREATE → Create new objects

✔ Allows:
- CREATE TABLE
- CREATE VIEW
- CREATE DATABASE
- CREATE PROCEDURE/FUNCTION

#### ALTER → modify existing tables (Broad permission)

```sql
GRANT ALTER ON mydb.customers TO 'dev_user';
```

Allows:
- Add column
- Drop column
- Modify column type
- Add/remove constraints  (FK, PK, etc.)
- Rename table

You cannot restrict ALTER to only one action

This is all-or-nothing

#### DROP → delete objects

✔ Allows:
- DROP TABLE
- DROP DATABASE
- DROP VIEW

#### INDEX → Manage indexes

✔ Allows:
- CREATE INDEX
- DROP INDEX

#### TRIGGER → Manage triggers
✔ Allows:
- CREATE TRIGGER
- DROP TRIGGER

#### How To Create a user 

Creating a User:-

Syntax:-
```sql
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
```

What is 'host'?
| Host value       | Meaning                |
| ---------------- | ---------------------- |
| `'localhost'`    | Only from same machine |
| `'%'`            | From anywhere          |
| `'192.168.1.10'` | Specific IP            |

### Removing (Deleting) a User
```sql
DROP USER 'username'@'host';
```
Dropping user removes everything
- Permissions gone
- Access gone
- No recovery (unless recreated)

User = username + host

These are different users:
- 'usr'@'localhost'
- 'usr'@'%'

### Change password

```sql
ALTER USER 'usr' IDENTIFIED BY 'newpass';
```

### Lock user (disable login)
```sql
ALTER USER 'usr'@'localhost' ACCOUNT LOCK;
```

### Unlock user :
```sql
ALTER USER 'usr'@'localhost' ACCOUNT UNLOCK;
```

### ⚡ 2. Main DCL Commands

There are only 2 main commands (but very powerful):

### ✅ 1. GRANT

🔹 Purpose:

Give permissions to a user

📌 Syntax:

```sql
GRANT privilege_name
ON object_name
TO user;
```

### 💡 Example 1: Give SELECT permission

```sql
GRANT SELECT ON employees TO 'meet'@'localhost';
```

👉 Meaning:

- User meet can only read data
- Cannot insert/update/delete

### 💡 Example 2: Multiple permissions

```sql
GRANT SELECT, INSERT ON employees TO 'meet'@'localhost';
```

👉 Now user can:

- Read data
- Insert data

### 💡 Example 3: Full access

```sql
GRANT ALL PRIVILEGES ON employees TO 'meet'@'localhost';
```

### 💡 Example 4: Entire database access

```sql
GRANT ALL PRIVILEGES ON company_db.* TO 'meet'@'localhost';
```
👉 * means all tables

### 💡 Example 5: Grant with ability to further grant

```sql
GRANT SELECT ON employees TO 'meet'@'localhost' WITH GRANT OPTION;
```

👉 Now meet can:
- Give SELECT permission to others 😎

### ❌ 2. REVOKE

🔹 Purpose:

Remove permissions

📌 Syntax:
```sql
REVOKE privilege_name
ON object_name
FROM user;
```

### 💡 Example 1:
```sql
REVOKE INSERT ON employees FROM 'meet'@'localhost';
```
👉 User can no longer insert

### 💡 Example 2: Remove all permissions
```sql
REVOKE ALL PRIVILEGES ON employees FROM 'meet'@'localhost';
```

### 🔍 3. Types of Privileges (Important)

📊 Table-level privileges
- SELECT
- INSERT
- UPDATE
- DELETE

🛠️ Administrative privileges
- CREATE
- DROP
- ALTER

🔥 Special privileges
- ALL PRIVILEGES
- GRANT OPTION

------------------------------------------------------------------------

##  Transaction Control Language (TCL)

TCL is used to control transactions in a database

👉 A transaction = a group of SQL operations treated as one unit of work

Example:

- Transfer money from Account A → Account B

This involves:
- Deduct from A
- Add to B

✔ Either both happen
❌ Or none happen (to maintain consistency)

### 🔹 Why TCL is Important?

Because of ACID properties:

- Atomicity → All or nothing
- Consistency → Data remains valid
- Isolation → Transactions don’t interfere
- Durability → Once committed, changes are permanent

### 🔹 TCL Commands (Main 4)

- START TRANSACTION / BEGIN
- COMMIT
- ROLLBACK
- SAVEPOINT

### 🔥 Let’s Learn One by One with Practical Examples

### 1️⃣ START TRANSACTION / BEGIN

👉 Starts a transaction

```sql
START TRANSACTION;
```

or

```sql
BEGIN;
```

💡 Example
```sql
START TRANSACTION;

UPDATE accounts 
SET balance = balance - 1000 
WHERE id = 1;

UPDATE accounts 
SET balance = balance + 1000 
WHERE id = 2;
```

👉 At this point:

Changes are NOT permanent yet

### 2️⃣ COMMIT

👉 Saves changes permanently

```sql
COMMIT;
```

💡 Example

```sql
START TRANSACTION;

UPDATE accounts 
SET balance = balance - 1000 
WHERE id = 1;

UPDATE accounts 
SET balance = balance + 1000 
WHERE id = 2;

COMMIT;
```

✔ Now changes are permanent

### 3️⃣ ROLLBACK

👉 Undo all changes in the transaction

```sql
ROLLBACK;
```

💡 Example (Error Scenario)

```sql
START TRANSACTION;

UPDATE accounts 
SET balance = balance - 1000 
WHERE id = 1;

-- ERROR occurs here ❌

ROLLBACK;
```

✔ Everything goes back to original state

### 4️⃣ SAVEPOINT

👉 Create a checkpoint inside a transaction

💡 Example

```sql
START TRANSACTION;

UPDATE accounts SET balance = balance - 1000 WHERE id = 1;

SAVEPOINT sp1;

UPDATE accounts SET balance = balance + 1000 WHERE id = 2;

-- Suppose something goes wrong

ROLLBACK TO sp1;

COMMIT;
```

👉 Result:

- First update remains
- Second update is undone

### 🔥 Full Practical Example

🏦 Bank Transfer System

```sql
START TRANSACTION;

-- Step 1: Deduct money
UPDATE accounts 
SET balance = balance - 500 
WHERE id = 1;

-- Step 2: Add money
UPDATE accounts 
SET balance = balance + 500 
WHERE id = 2;

-- Check condition
-- If something wrong → rollback
-- Otherwise → commit

COMMIT;
```

### ⚠️ Important Notes

1. Auto Commit Mode

MySQL by default:

```
SET autocommit = 1;
```

👉 Every query is committed automatically

To disable:
```
SET autocommit = 0;
```

2. When ROLLBACK Doesn’t Work ❌

Rollback works only when:

- Transaction is not committed
- DDL Commands (Craete,Alter,Truncate,Drop) Can't be Rollback
- Table uses InnoDB engine

------------------------------------------------------------------------

## Locking

### 🔹 What is Locking in MySQL?

👉 Locking = restricting access to data so multiple users don’t corrupt it

When multiple queries run at the same time:
- Without locks → ❌ wrong data (race conditions)
- With locks → ✅ safe & consistent data

### 🔥 Simple Real-Life Example

Two users trying to withdraw money:
```
Balance = 1000
```

- User A → withdraw 800
- User B → withdraw 500

Without locking:
- 👉 Both read 1000 → total deducted = 1300 ❌

With locking:
- 👉 One waits → correct result ✅

### 🔹 Types of Locks in MySQL

### 1️⃣ Table-Level Lock

👉 Entire table is locked

Example:

```sql
LOCK TABLES employees WRITE;

SELECT * FROM employees; -- others cannot read/write
```
Unlock:

```sql
UNLOCK TABLES;
```

#### 🔥 Behavior

| Lock Type | Effect                     |
| --------- | -------------------------- |
| READ      | Others can read, not write |
| WRITE     | No one else can read/write |

#### ⚠️ Problem

- 👉 Slow in high-traffic apps
- 👉 Not used much with InnoDB

### 2️⃣ Row-Level Lock (Most Important)

- 👉 Only specific rows are locked (InnoDB)

Example
```sql
START TRANSACTION;

SELECT * FROM accounts 
WHERE id = 1 
FOR UPDATE;
```

👉 This locks only that row

🔥 Now:

Another query:

UPDATE accounts SET balance = 500 WHERE id = 1;

👉 ⏳ It will WAIT until first transaction finishes

### 3️⃣ Shared Lock (S Lock)

👉 Read lock

```sql
SELECT * FROM accounts WHERE id = 1 LOCK IN SHARE MODE;
```

Behavior:
- ✅ Others can read
- ❌ Others cannot write

### 4️⃣ Exclusive Lock (X Lock)

👉 Write lock

```sql
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
```

Behavior:
- ❌ No read (in some cases)
- ❌ No write
- Full control

🔥 Difference (Important)

| Lock           | Read      | Write     |
| -------------- | --------- | --------- |
| Shared Lock    | ✅ Allowed | ❌ Blocked |
| Exclusive Lock | ❌ Blocked | ❌ Blocked |

### 5️⃣ Intent Locks

👉 Used internally by MySQL

Types:
- Intent Shared (IS)
- Intent Exclusive (IX)

👉 Helps MySQL manage row + table locks together

You don’t write these manually

### 6️⃣ Gap Lock

👉 Locks a range of rows (even non-existing rows)

```sql
SELECT * FROM accounts 
WHERE balance BETWEEN 1000 AND 2000 
FOR UPDATE;
```

👉 Locks:

- Existing rows
- AND gaps between them

Why?

👉 Prevent phantom reads

### 🔥 Types Summary

| Lock Type      | Level    | Use             |
| -------------- | -------- | --------------- |
| Table Lock     | Table    | Old / MyISAM    |
| Row Lock       | Row      | InnoDB          |
| Shared Lock    | Row      | Read            |
| Exclusive Lock | Row      | Write           |
| Gap Lock       | Range    | Prevent phantom |
| Intent Lock    | Internal | Optimization    |

### 🔥 Practical Scenario

🏦 Money Transfer

```sql
START TRANSACTION;

SELECT balance FROM accounts 
WHERE id = 1 FOR UPDATE;

UPDATE accounts 
SET balance = balance - 500 
WHERE id = 1;

UPDATE accounts 
SET balance = balance + 500 
WHERE id = 2;

COMMIT;
```

------------------------------------------------------------------------

## 🚨 Deadlock

👉 A deadlock happens when:

Two transactions are waiting for each other to release locks — and neither can proceed

#### 🏦 Bank Accounts

We have:
```
Account A → id = 1  
Account B → id = 2
```

#### 🔥 Step-by-Step Deadlock Scenario

👉 Transaction 1 (T1)
```sql
START TRANSACTION;

UPDATE accounts 
SET balance = balance - 100 
WHERE id = 1;
```
👉 T1 locks Account A (id=1) 🔒

👉 Transaction 2 (T2)
```sql
START TRANSACTION;

UPDATE accounts 
SET balance = balance - 200 
WHERE id = 2;
```
👉 T2 locks Account B (id=2) 🔒

👉 Now the Problem Starts

T1 tries:
```sql
UPDATE accounts 
SET balance = balance + 100 
WHERE id = 2;
```

- 👉 ❌ Cannot proceed
- 👉 Because T2 already locked id=2
- 👉 T1 is now WAITING ⏳

T2 tries:
```sql
UPDATE accounts 
SET balance = balance + 200 
WHERE id = 1;
```

- 👉 ❌ Cannot proceed
- 👉 Because T1 already locked id=1
- 👉 T2 is also WAITING ⏳

#### 💥 DEADLOCK CREATED

| Transaction | Holding Lock | Waiting For |
| ----------- | ------------ | ----------- |
| T1          | id=1         | id=2        |
| T2          | id=2         | id=1        |

- 👉 Circular dependency 🔁
- 👉 No one can move forward

#### 🔥 What MySQL Does

- 👉 MySQL detects deadlock automatically
- 👉 It kills one transaction

Example error:
```
ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction
```

#### 🔄 What Happens Next?

Suppose MySQL kills T2

- T2 → ❌ ROLLBACK
- T1 → ✅ Continues and commits

#### 🔥 How to See Deadlock Info (Very Important)

```
SHOW ENGINE INNODB STATUS;
```

👉 Shows:

- Which queries caused deadlock
- Which transaction was killed

### 🚨 Why Deadlocks Happen?

Main reasons:

1. ❌ Different order of locking

T1:
```
locks id=1 → then id=2
```

T2:
```
locks id=2 → then id=1
```

👉 💥 Boom → deadlock

------------------------------------------------------------------------

## Deadlocks Prevention Techniques

✅ 1. Always lock rows in same order

✔ Correct approach:

```sql
-- Both transactions follow same order

UPDATE accounts WHERE id = 1;
UPDATE accounts WHERE id = 2;
```

👉 No circular wait → no deadlock

✅ 2. Use SELECT ... FOR UPDATE properly

```sql
START TRANSACTION;

SELECT * FROM accounts 
WHERE id IN (1,2) 
ORDER BY id 
FOR UPDATE;
```

👉 Locks rows in consistent order

✅ 3. Keep transactions SHORT

❌ Bad:

```sql
START TRANSACTION;
-- do API call
-- do logic
UPDATE ...
```

👉 Locks held too long → deadlocks likely

✅ 4. Proper indexing

👉 Without index:

- MySQL locks many rows → increases chances

------------------------------------------------------------------------

## Transaction Anomalies:-

Transaction anomalies are problems or inconsistencies that occur when multiple transactions run concurrently in a database
without proper isolation.

👉 In simple terms:

When two or more users access/modify data at the same time, wrong or unexpected results can happen.

### 🔹 Why Do Transaction Anomalies Occur?

Because:

- Databases allow concurrent execution
- But without proper control → data becomes inconsistent

👉 That’s why MySQL provides Isolation Levels to prevent them.

### 🔹 Types of Transaction Anomalies

### 1️⃣ Dirty Read

👉 One transaction reads uncommitted data from another transaction

✅ Example

```sql
-- Transaction A
START TRANSACTION;
UPDATE accounts SET balance = 500 WHERE id = 1;
-- NOT COMMITTED

-- Transaction B
SELECT balance FROM accounts WHERE id = 1;
```

👉 Transaction B sees 500

BUT…
```sql
-- Transaction A
ROLLBACK;
```

👉 Actual value is still 1000

❌ Problem:

B read data that never actually existed

### 2️⃣ Non-Repeatable Read

👉 Same query gives different results within same transaction

✅ Example

```sql
-- Transaction A
START TRANSACTION;
SELECT salary FROM employees WHERE id = 1;  -- 50000

-- Transaction B
UPDATE employees SET salary = 60000 WHERE id = 1;
COMMIT;

-- Transaction A
SELECT salary FROM employees WHERE id = 1;  -- 60000
```

❌ Problem:

Data changed during transaction

### 3️⃣ Phantom Read

👉 New rows appear/disappear in repeated queries

✅ Example

```sql
-- Transaction A
SELECT * FROM employees WHERE salary > 50000;

-- Transaction B
INSERT INTO employees VALUES (5, 'New', 70000);
COMMIT;

-- Transaction A
SELECT * FROM employees WHERE salary > 50000;
```

👉 Now extra row appears

❌ Problem:

Result set changed unexpectedly

### 4️⃣ Lost Update

👉 Two transactions update same data → one update is lost

✅ Example

```sql
-- Initial balance = 1000

-- Transaction A
SELECT balance = 1000
UPDATE balance = 900

-- Transaction B
SELECT balance = 1000
UPDATE balance = 800
```

👉 Final value = 800

❌ Problem:

A’s update is lost

### 🔹 How MySQL Solves These?

| Isolation Level           | Prevents               |
| ------------------------- | ---------------------- |
| READ UNCOMMITTED          | Nothing                |
| READ COMMITTED            | Dirty Read             |
| REPEATABLE READ (default) | Dirty + Non-repeatable |
| SERIALIZABLE              | All anomalies          |

### 🔹 Simple Summary Table

| Anomaly             | Problem               |
| ------------------- | --------------------- |
| Dirty Read          | Read uncommitted data |
| Non-repeatable Read | Same row changes      |
| Phantom Read        | New rows appear       |
| Lost Update         | Update overwritten    |

------------------------------------------------------------------------

## Isolation Levels

👉 Isolation levels define:

“How much one transaction can see another transaction’s data”

### 🔹 1. READ UNCOMMITTED

👉 Lowest level (almost no protection)

❌ Allows:
- Dirty Read ✅
- Non-repeatable Read ✅
- Phantom Read ✅

✅ Example
```sql
-- Transaction A
START TRANSACTION;
UPDATE accounts SET balance = 500 WHERE id = 1;

-- Transaction B
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SELECT balance FROM accounts WHERE id = 1;
```

👉 B can see uncommitted data (500) ❌

🧠 Use case

- Almost NEVER used in real systems

### 🔹 2. READ COMMITTED

👉 Only sees committed data

❌ Prevents:
- Dirty Read ❌

⚠️ Still allows:
- Non-repeatable Read ✅
- Phantom Read ✅

✅ Example

```sql
-- Transaction A
START TRANSACTION;
SELECT salary FROM employees WHERE id = 1; -- 50000

-- Transaction B
UPDATE employees SET salary = 60000 WHERE id = 1;
COMMIT;

-- Transaction A again
SELECT salary FROM employees WHERE id = 1; -- 60000 ❌ changed
```

🧠 Use case
- Used in systems like PostgreSQL default

### 🔹 3. REPEATABLE READ (🔥 MySQL Default)

👉 Same data remains consistent within transaction

❌ Prevents:
- Dirty Read ❌
- Non-repeatable Read ❌

⚠️ Phantom Read?

- 👉 MySQL prevents it using gap locks

✅ Example

```sql
-- Transaction A
START TRANSACTION;
SELECT salary FROM employees WHERE id = 1; -- 50000

-- Transaction B
UPDATE employees SET salary = 60000 WHERE id = 1;
COMMIT;

-- Transaction A again
SELECT salary FROM employees WHERE id = 1; -- STILL 50000 ✅
```

👉 MySQL uses MVCC (snapshot)

🧠 Key concept:

You see a consistent snapshot of data

### 🔹 4. SERIALIZABLE

👉 Highest level (strictest)

❌ Prevents:
- All anomalies ❌

✅ Behavior
```sql
SELECT * FROM employees WHERE salary > 50000;
```

👉 MySQL locks range → no insert allowed

⚠️ Drawback:
- Slow performance
- High locking

------------------------------------------------------------------------

## Generated Column or Calculated Column

A Generated Column is a column whose value is automatically computed from other columns using an expression.

👉 You don’t insert/update it manually — MySQL calculates it.

### 🔹 Simple Idea
- total_price = quantity * price

Instead of calculating this in your application every time, you let MySQL handle it.

### 🔹 2. Types of Generated Columns

MySQL supports 2 types:

### 1️⃣ Virtual Column:-
- Not stored physically
- Calculated on the fly
- Uses less storage
- Slightly slower when reading

### 2️⃣ Stored Column:-
- Stored physically in table
- Takes storage
- Faster reads

### 🔥 Syntax

```sql
column_name data_type 
GENERATED ALWAYS AS (expression)
[VIRTUAL | STORED]
```

### 🔹 3. First Practical Example

🎯 Scenario: Orders table

```sql
CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    price DECIMAL(10,2),
    quantity INT,

    total_price DECIMAL(10,2) 
    GENERATED ALWAYS AS (price * quantity) STORED
);
```

🔹 Insert Data

```sql
INSERT INTO orders (price, quantity)
VALUES (100, 2), (50, 5);
```

🔹 Output
```sql
SELECT * FROM orders;
```

| id | price | quantity | total_price |
| -- | ----- | -------- | ----------- |
| 1  | 100   | 2        | 200         |
| 2  | 50    | 5        | 250         |

👉 You never inserted total_price, MySQL calculated it.

### 🔹 4. Virtual vs Stored

| Feature       | Virtual           | Stored            |
| ------------- | ----------------- | ----------------- |
| Storage       | ❌ No              | ✅ Yes             |
| Performance   | Slower read       | Faster read       |
| Index allowed | Limited           | Yes               |
| Use case      | Lightweight logic | Heavy computation |

#### Calculated Column is same as Generated Column In SQL Server It is call as Calculated Column 

Instead of stored we have to write persist when creating table to store data physically

------------------------------------------------------------------------

## Insert On Duplicate Key

INSERT ... ON DUPLICATE KEY UPDATE in MySQL is a very powerful feature used to handle situations where:

- 👉 You try to insert a row
- 👉 But a duplicate key conflict happens (PRIMARY KEY or UNIQUE KEY)
- 👉 Instead of throwing an error, MySQL updates the existing row

### 🔹 1. Basic Idea

👉 Normally:
```sql
INSERT INTO users (id, name) VALUES (1, 'Meet');
```

If id = 1 already exists ❌ → Error

👉 With duplicate handling:

```sql
INSERT INTO users (id, name)
VALUES (1, 'Meet')
ON DUPLICATE KEY UPDATE name = 'Meet';
```

✔ If no duplicate → INSERT
✔ If duplicate → UPDATE

### 🔹 2. Practical Example

Create table
```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    salary INT
);
```

Insert first time
```sql
INSERT INTO users VALUES (1, 'Raj', 50000);
```

Insert again with same PK
```sql
INSERT INTO users (id, name, salary)
VALUES (1, 'Meet', 60000)
ON DUPLICATE KEY UPDATE
    name = VALUES(name),
    salary = VALUES(salary);
```

🔍 What happens?
- id = 1 already exists
- Instead of error ❌

👉 It updates row:
```
Before: (1, Raj, 50000)
After : (1, Meet, 60000)
```

### 🔹 3. Important Syntax
```sql
INSERT INTO table_name (col1, col2)
VALUES (val1, val2)
ON DUPLICATE KEY UPDATE
    col1 = new_value,
    col2 = new_value;
```

### 🔹 4. What triggers "duplicate key"?

This works when conflict happens on:

- ✔ PRIMARY KEY
- ✔ UNIQUE KEY

Example:
```
UNIQUE(email)
```

If same email inserted → triggers update

### 🔹 5. Using VALUES() (Important)

```sql
ON DUPLICATE KEY UPDATE
name = VALUES(name)
```

👉 Means:

Use the value from INSERT statement

⚠️ Note (MySQL 8+):

- VALUES() is deprecated
- Use alias instead:

```sql
INSERT INTO users (id, name, salary)
VALUES (1, 'Meet', 60000) AS new
ON DUPLICATE KEY UPDATE
    name = new.name,
    salary = new.salary;
```
------------------------------------------------------------------------

## Dump in MySQL

### 🔹 1. What is a Dump?

👉 A dump is:

A file containing SQL statements (like CREATE, INSERT) that recreate your database

📌 Simple meaning:
- It’s a backup of your database
- Stored as a .sql file

### 🔹 2. Why Do We Use Dump?

✅ Backup
- Save data before risky changes
✅ Migration
- Move data from one server to another
✅ Recovery
- Restore data if something goes wrong

## 🔹 3. How Dump Looks Internally

A dump file contains SQL like:

```sql
CREATE TABLE employees (
    id INT,
    name VARCHAR(50)
);

INSERT INTO employees VALUES (1, 'Meet');
INSERT INTO employees VALUES (2, 'Raj');
```

👉 When you run this file → database is recreated

### 🔹 4. Tool Used: mysqldump

👉 MySQL provides a command-line tool called:

mysqldump

✅ Dump a single database
```bash
mysqldump -u root -p company_db > company_db.sql
```

✅ Dump specific table
```bash
mysqldump -u root -p company_db employees > employees.sql
```

✅ Dump all databases
```bash
mysqldump -u root -p --all-databases > all_db.sql
```

### 🔹 5. How to Restore Dump
```bash
mysql -u root -p company_db < company_db.sql
```

👉 This will recreate:
- Tables
- Data
- Structure

### 🔹 6. Types of Dump

🔸 Logical Dump

- SQL statements (mysqldump)
- Human-readable
- Portable

🔸 Physical Dump (Advanced)

- Copy actual data files
- Faster but complex

### 🔹 7. Important Options 🔥

- 👉 But a duplicate key conflict happens (PRIMARY KEY or UNIQUE KEY)
- 👉 Instead of throwing an error, MySQL updates the existing row

### 🔹 1. Basic Idea

```bash
mysqldump -u root -p --no-data db_name > structure.sql
```
👉 Only table structure

```bash
mysqldump -u root -p --no-create-info db_name > data.sql
```
👉 Only data

```bash
mysqldump -u root -p --where="salary > 50000" db_name employees > filtered.sql
```
👉 Partial dump

------------------------------------------------------------------------
## set v/s enum

### 🔹 1. ENUM in MySQL

👉 What it is:
- A column that can store only ONE value from a predefined list

✅ Syntax:
```sql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    status ENUM('active', 'inactive', 'banned')
);
```
📌 Example Insert:
```sql
INSERT INTO users (status) VALUES ('active');
```

🧠 Key Point:
- Only one value allowed
- Stored internally as index (number) → efficient

### 🔹 2. SET in MySQL

👉 What it is:
- A column that can store MULTIPLE values from a predefined list

✅ Syntax:
```sql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    roles SET('admin', 'editor', 'viewer')
);
```
📌 Example Insert:
```sql
INSERT INTO users (roles) VALUES ('admin,editor');
```

🧠 Key Point:
- Can store multiple values at once
- Stored as bitmask internally

### 🔥 Difference: ENUM vs SET

| Feature        | ENUM             | SET              |
| -------------- | ---------------- | ---------------- |
| Values allowed | Only ONE         | Multiple         |
| Storage        | Index (1,2,3...) | Bitmask          |
| Use case       | Status, type     | Tags, roles      |
| Example        | active/inactive  | admin, editor    |
| Complexity     | Simple           | Slightly complex |

------------------------------------------------------------------------

##  Keys

------------------------------------------------------------------------

## Partitioning

------------------------------------------------------------------------

## ACID

------------------------------------------------------------------------

## DBEngines

------------------------------------------------------------------------

##  Dataypes

------------------------------------------------------------------------

## Triggers

------------------------------------------------------------------------

## Cursor

------------------------------------------------------------------------

##  SQL v/s NoSQL

------------------------------------------------------------------------
