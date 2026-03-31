# SQL Execution Order.

```
1. FROM
2. JOIN
3. WHERE
4. GROUP BY
5. HAVING
6. Partition by(Window Function)
7. SELECT
8. DISTINCT
9. ORDER BY
10. LIMIT + Offset
```

### 1.FROM (Data Source Selection)

👉 This is the first step
- MySQL decides which table(s) to read
- Loads raw data into memory (logical stage)

💡 Example:

```bash
SELECT * FROM users;
```

👉 MySQL:

```
Step 1: Load all rows from users table
```

### 2.Joins (Combine Tables)

👉 If multiple tables are used, MySQL combines them here

💡 Example:

```bash
SELECT u.name, a.city
FROM users u
JOIN addresses a ON u.id = a.user_id;
```

👉 MySQL:

```
Step 1: Load users
Step 2: Load addresses
Step 3: Combine rows based on condition
```

### 3. WHERE (Row Filtering)

👉 Filters individual rows

- Runs before grouping
- Cannot use aggregate functions ❌

💡 Example:

```bash
SELECT * 
FROM users 
WHERE salary > 50000;
```
👉 MySQL removes rows where salary ≤ 50000

### 4. GROUP BY (Grouping Rows)

👉 Groups rows based on column(s)

💡 Example:

```bash
SELECT city, COUNT(*) 
FROM users 
GROUP BY city;
```

👉 MySQL:

```
Ahmedabad → group
Surat → group
Rajkot → group
```

### 5. HAVING

👉 Filters grouped data

- works after aggregation
- Can use COUNT, SUM, etc.

💡 Example:

```bash
SELECT city, COUNT(*) 
FROM users 
GROUP BY city 
HAVING COUNT(*) > 2;
```

👉 Only cities with more than 2 users

### 6. SELECT

👉 Now MySQL selects what to show

💡 Example:

```bash
SELECT name, salary 
FROM users;
```

👉 Only these columns appear in output

#### ⚠️ Important

Aliases are created here:

```bash
SELECT salary AS s
```

👉 That’s why WHERE s > 50000 ❌ fails

### 7. DISTINCT

👉 Removes duplicate rows

💡 Example:

```bash
SELECT DISTINCT city FROM users;
```

👉 Only unique cities shown

### 8. ORDER BY

👉 Sorts final result

💡 Example:

```bash
SELECT * 
FROM users 
ORDER BY salary DESC;
```

👉 Highest salary first


### 9. LIMIT

👉 Limits number of rows

💡 Example:

```bash
SELECT * 
FROM users 
LIMIT 5;
```

👉 Only first 5 rows


## 🔥 Full Example

```bash
SELECT city, COUNT(*) AS total
FROM users
WHERE salary > 30000
GROUP BY city
HAVING COUNT(*) > 2
ORDER BY total DESC
LIMIT 3;
```

## 🧠 Internal Execution

```
1. FROM users
2. WHERE salary > 30000
3. GROUP BY city
4. COUNT(*) calculated
5. HAVING COUNT(*) > 2
6. SELECT city, total
7. ORDER BY total DESC
8. LIMIT 3
```

### 🚀 Real-Life Analogy

Think like a restaurant kitchen:

```
FROM → bring ingredients  
WHERE → remove bad ingredients  
GROUP BY → group dishes  
HAVING → remove unwanted groups  
SELECT → prepare final dish  
ORDER BY → arrange dishes  
LIMIT → serve only few  
```

## Inner Join

Gives All Data That Matching from both Tabels

## Left Join

Give All data From Left hand Side if data is not present in Righthand side then it gives null

## Right Join

Give All data from Right hand side if data is not present in lefthand side it gives null

## Croos Join

Give m * n


