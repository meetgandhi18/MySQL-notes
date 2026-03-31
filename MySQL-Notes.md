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

##  Dataypes :-

------------------------------------------------------------------------

##  SQL v/s NoSQL

------------------------------------------------------------------------

## View,CTE,with check view


------------------------------------------------------------------------

##  Keys

------------------------------------------------------------------------

## Partitioning

------------------------------------------------------------------------

## ACID

------------------------------------------------------------------------

## DBEngines

------------------------------------------------------------------------

##  DCL (Data control Language)

------------------------------------------------------------------------

##  Transactionc (TCL)

------------------------------------------------------------------------


## Generated Column

------------------------------------------------------------------------

## 🚀 Final Summary

You covered: - Execution flow - Joins - Filtering - Grouping - Window
functions - Subqueries (IN, EXISTS, ANY, ALL) - Optimization

Practice writing queries daily 🔥
