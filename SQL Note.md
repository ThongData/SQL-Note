# 📝 SQL 101 Guide

## 📚 Table of Content

<details>
<summary>
Click here to expand!
</summary>

1. Order of Execution
2. CRUD Operations
3. Relational Database
    - Create, Update, Alter Table
    - Insert Values
    - Table Constraints
    - Create Index
4. Data Types	
5. Base query
6. Filtering Techniques
7. Grouping Records
8. Joining Tables
9. Built-in Functions
10. Aggregations
    - Summary Statistics
11. Window Functions
12. Condition Statement
13. Create Temporary Table
14. Derived Tables
  - Common Table Expressions (CTE)  
  - Subquery
15. PIVOT
16. Declare Variables
 
</details>

***

## 📌 Section 1: Order of Execution

Sequence of how SQL runs the query:

1. FROM, JOINS - SQL fetches the data from the tables.
2. WHERE - Filters based on specified condition
3. GROUP BY - Aggregates / Groups data into groupings and remove duplicates.
4. HAVING - Filters grouped data based on specified condition.
5. SELECT - Selects the columns and expressions to display.
6. DISTINCT - Remove duplicates.
7. UNION, INTERSECT, EXCEPT - Can remove duplicates. UNION ALL include duplicates.
8. ORDER BY - Orders the columns.
9. OFFSET, LIMIT - Limits the results.

## 📌Section 2: CRUD Operations

- **CREATE** - Create databases, tables or views
- **READ** - Read using SELECT clause
- **UPDATE** - Amend existing database records
- **DELETE** - Delete records

### 2.1: Create Table

````sql
CREATE TABLE rooms (
  room_id INT IDENTITY(1,1) NOT NULL, -- IDENTITY(1,1) refers to an identity key which auto-increments by 1
  room_no CHAR(3) NOT NULL,
  bed_type VARCHAR(15) NOT NULL,
  rate SMALLMONEY NOT NULL);
````

````sql
CRREATE TABLE students (
  ssn INTEGER PRIMARY KEY NOT NULL, -- Set as primary key and not null
  name VARCHAR(64),
  dob DATE,
  average_grade NUMERIC (3,2), -- Precision of 3 and scale of 2, eg. 100.00
  tuition_paid BOOLEAN);
````

### 2.2: Update Records

Before running `UPDATE` clause, run a `SELECT` query to identify the exact row(s) to update to ensure that we update correct rows.

````sql
UPDATE guests
SET checkin_date = '2021-05-10'
WHERE reservation_id = 1001; - Specify condition to identify specific row(s) to update
````

````sql
UPDATE guests
SET checkin_date = '2021-05-10',
	checkout_date = '2021-05-15'
WHERE reservation_id = 1001;
````

````sql
UPDATE movies
SET title = "Toy Story 3", 
	director = "Lee Unkrich"
WHERE id = 11;
````

### 2.3: Delete Records

Before running `DELETE` clause, run a `SELECT` query to identify the exact row(s) for deletion to ensure that we delete only the unwanted rows.

````sql
DELETE FROM guests
WHERE customer_id = 1005;
````

***

## 📌 Section 3: Relational Database

### 3.1: Alter Table

````sql
-- ADD new column
ALTER TABLE guests
ADD last_name VARCHAR(15);
````

````sql
-- Add new column and set text default
ALTER TABLE movies
ADD COLUMN Language TEXT DEFAULT "English"; -- Set 'English' as default values in all rows of Language column
````

````sql
-- Alter column's data type
ALTER TABLE name
ALTER COLUMN firstname TYPE VARCHAR(128);
````

````sql
-- Alter column's data type
ALTER TABLE student
ALTER COLUMN average_grade -- eg. original output is 98.68
TYPE INTEGER -- Alters to integer output, eg. 98 (normally, will round down)
USING ROUND(average_grade); -- Round up integer instead to, eg. 99
````

````sql
-- Set / Drop NOT NULL constraint
ALTER TABLE students
ALTER COLUMN home_phone
SET NOT NULL; -- Set not-null constraint, OR
DROP NOT NULL; -- Drop not-null constraint
````

````sql
-- Drop column
ALTER TABLE table_name
DROP COLUMN old_column;
````

````sql
-- Rename column
ALTER TABLE table_name
RENAME COLUMN old_column TO new_column;
````

### 3.2: Remove Table

Use `TRUNCATE TABLE` to remove data from table with table structure still existing in the database.

````sql
TRUNCATE TABLE rooms;
````

Use `DROP TABLE` to delete the entire table including data from the database.

````sql
DROP TABLE rooms;
````

### 3.3: Insert Records

````sql
- Using results from query to insert into table
INSERT INTO rooms (room_no, bed_type, rate)
SELECT
  col_1,
  col_2,
  col_3
FROM other_table
WHERE --conditions apply
````

````sql
INSERT INTO rooms (room_no, bed_type, rate)
	VALUES ('101', 'King', 120),
		('102', 'Queen', 100),
		('103', 'Deluxe', 80),
		('104', 'King', 120),
		('105', 'Queen', 100);
````

***

## 📌 Section 4: Date Types

### ADD_MONTHS

https://stackoverflow.com/questions/28635226/adding-months-to-date-postgresql-vs-oracle

***
````sql
-- Credit to http://www.dwhworld.com/2010/08/date-dimension-sql-scripts-mysql/
-- Small-numbers table
DROP TABLE IF EXISTS numbers_small;
CREATE TABLE numbers_small (number INT);
INSERT INTO numbers_small VALUES (0),(1),(2),(3),(4),(5),(6),(7),(8),(9);

-- Main-numbers table
DROP TABLE IF EXISTS numbers;
CREATE TABLE numbers (number BIGINT);
INSERT INTO numbers
SELECT thousands.number * 1000 + hundreds.number * 100 + tens.number * 10 + ones.number
FROM numbers_small thousands, numbers_small hundreds, numbers_small tens, numbers_small ones
LIMIT 1000000;

-- Create Date Dimension table
DROP TABLE IF EXISTS date_d;
CREATE TABLE date_d (
date_id          BIGINT PRIMARY KEY,
date             DATE NOT NULL,
year             INT,
month            CHAR(10),
month_of_year    CHAR(2),
day_of_month     INT,
day              CHAR(10),
day_of_week      INT,
weekend          CHAR(10) NOT NULL DEFAULT "Weekday",
day_of_year      INT,
week_of_year     CHAR(2),
quarter  INT,
previous_day     date NOT NULL default '0000-00-00',
next_day         date NOT NULL default '0000-00-00',
UNIQUE KEY `date` (`date`));

-- First populate with ids and Date
-- Change year start and end to match your needs. The above sql creates records for year 2010.
INSERT INTO date_d (date_id, date)
SELECT number, DATE_ADD( '2014-01-01', INTERVAL number DAY )
FROM numbers
WHERE DATE_ADD( '2014-01-01', INTERVAL number DAY ) BETWEEN '2014-01-01' AND '2024-12-31'
ORDER BY number;

-- Update other columns based on the date.
UPDATE date_d SET
year            = DATE_FORMAT( date, "%Y" ),
month           = DATE_FORMAT( date, "%M"),
month_of_year   = DATE_FORMAT( date, "%m"),
day_of_month    = DATE_FORMAT( date, "%d" ),
day             = DATE_FORMAT( date, "%W" ),
day_of_week     = DAYOFWEEK(date),
weekend         = IF( DATE_FORMAT( date, "%W" ) IN ('Saturday','Sunday'), 'Weekend', 'Weekday'),
day_of_year     = DATE_FORMAT( date, "%j" ),
week_of_year    = DATE_FORMAT( date, "%V" ),
quarter         = QUARTER(date),
previous_day    = DATE_ADD(date, INTERVAL -1 DAY),
next_day        = DATE_ADD(date, INTERVAL 1 DAY);
````
## 📌 Section 5 : Table Constraints

Constraints give the data structure and help with consistency and data quality.

**Integrity Constraints**
1. Attribute constraints - Eg. data types on columns
2. Key constraints - **Primary keys**
3. Referential integrity constraints - Enforced through **foreign keys**

- Auto-increment
- Unique
- NOT NULL

````SQL
ALTER TABLE professors 
ADD COLUMN id serial -- Add auto-incremental function to id column
````

### 5.1: Primary Key

````sql
-- Set primary key using CONCAT
UPDATE cars
SET id = CONCAT(make, model); -- Update id with make + model

ALTER TABLE cars
ADD CONSTRAINT id_pk PRIMARY KEY(id); -- Make id a primary key
````

````sq;
-- Rename the organization column to id
ALTER TABLE organizations
RENAME organization TO id;

-- Make id a primary key
ALTER TABLE organizations
ADD CONSTRAINT organization_pk PRIMARY KEY (id);
````

````sql
ALTER TABLE guests
ADD CONSTRAINT first_name_unq UNIQUE (first_name); -- 'first_name-unq' represents name of the constraint
````

### 5.2: Foreign Key

````sql
ALTER TABLE a
ADD CONSTRAINT a_fkey FOREIGN KEY (b_id) REFERENCES table_b (id)
````

**Reference a Table with FOREIGN KEY**

For example, you want the `professors` table to reference the `universities` table using `university_id`.

````sql
-- Rename column to xxx_id
ALTER TABLE professors
RENAME COLUMN university_shortname TO university_id;

-- Add a foreign key on professors referencing universities
ALTER TABLE professors
ADD CONSTRAINT professors_fkey FOREIGN KEY (university_id) REFERENCES universities (id);
````

**How to Populate Foreign Key from Another Table**

````sql
UPDATE table_a
SET column_to_update = table_b.column_to_update_from
FROM table_b
WHERE condition1 
  AND condition2 
  AND ...;
````

For example,
````sql
UPDATE affiliations
SET professor_id = professors.id
FROM professors
WHERE affiliations.firstname = professors.firstname 
	AND affiliations.lastname = professors.lastname;
````

### 5.3: NULL Values

Take note that we cannot use `!=` or `<>` on NULL values.

````sql
SELECT 
  WorkOrderID, 
  ScrappedQty, 
  ScrapReasonID
FROM Production.WorkOrder
WHERE ScrapReasonID IS NOT NULL;
````

**ISNULL**

````sql
SELECT 
  GDP, Year,
  ISNULL(Country, 'Unknown') AS NewCountry
FROM EconomicIndicator;
````

For example, replace NULL values with '99'.
````sql
SELECT 
  WorkOrderID, 
  ScrappedQty, 
  ISNULL(ScrapReasonID, 99) AS ScrapReason
FROM Production.WorkOrder;
````

**To Find for Blank Values**

````sql
SELECT 
  Country, GDP, Year 
FROM EconomicIndicator
WHERE LEN(GDP) > 0; -- To search for blank values
````

**COALESCE**

Returns the first non-missing value and use to replace NULL values.

````sql
SELECT 
    Country, 
    COALESCE(Country, IncidentState, City, 'Unknown') AS Location
    -- If Country is NULL, then look for replacement in IncidentState, then followed by City values. Otherwise, replace with
FROM Incidents
WHERE Country IS NULL
````

In the above example, 
- We filter to results where Country is NULL, then use the COALESCE function to replace a value from one of the columns (Country, IncidentState, or City).
- If Country value is NULL, then look for the next replacement in IncidentState, then followed by City values. If values in all 3 columns are NULL, then replace with 'Unknown' value.

````sql
-- Replace the nulls in the columns with meaningful text
SELECT
  COALESCE(Country, 'All countries') AS Country,
  COALESCE(Gender, 'All genders') AS Gender,
  COUNT(*) AS Awards
FROM Summer_Medals
WHERE
  Year = 2004
  AND Medal = 'Gold'
  AND Country IN ('DEN', 'NOR', 'SWE')
GROUP BY ROLLUP(Country, Gender)
ORDER BY Country ASC, Gender ASC;
````

***

## 📌 Section 6: Index

Index improves the speed of looking through the table's data. Without an index, SQL performs a table scan by searching for every record in the table. 

It acts as 'Table of Content' in a book - it's (usually) much faster to look up something in a book by looking at its index than by flipping every page until we find what we want.

### Create Index

### Drop Index

***

## 📌 Section 7 : Data Types

- text - character strings of any length
- varchar(x) - a maximum of 'x' characters
- char(x) - a fixed-length string of 'x' characters
- boolean - True(1) , False(0) or NULL
- date, time, timestamps - date, datetime, timezone
- numeric - float, decimal
- integer - whole numbers

### 7.1: Use CAST

````sql
SELECT 
	CAST(some_column AS integer)
FROM table;
````

````sql
SELECT 
  transaction_date, 
  (amount + CAST(fee AS integer)) AS net_amount 
FROM transactions;
````

### 7.2: Use CONVERT 

````sql
DECLARE
	@CubsWinWorldSeries DATETIME2(3) = '2016-11-03 00:30:29.245';

SELECT
	CONVERT(DATE, @CubsWinWorldSeries) AS CubsWinDateForm,
	CONVERT(NVARCHAR(30),@CubsWinWorldSeries) AS CubsWinStringForm;
````

````sql
DECLARE
	@CubsWinWorldSeries DATETIME2(3) = '2016-11-03 00:30:29.245';

SELECT
	CONVERT(NVARCHAR(30), @CubsWinWorldSeries, 0) AS DefaultForm,
	CONVERT(NVARCHAR(30), @CubsWinWorldSeries, 3) AS UK_dmy,
	CONVERT(NVARCHAR(30), @CubsWinWorldSeries, 1) AS US_mdy,
	CONVERT(NVARCHAR(30), @CubsWinWorldSeries, 103) AS UK_dmyyyy,
	CONVERT(NVARCHAR(30), @CubsWinWorldSeries, 101) AS US_mdyyyy;
````

### 7.3: Use FORMAT

````sql
-- To retrieve short dates using 'd' and long dates using 'D'
DECLARE
	@Python3ReleaseDate DATETIME2(3) = '2008-12-03 19:45:00.033';

SELECT
	-- Fill in the format parameter
	FORMAT(@Python3ReleaseDate, 'd', 'en-US') AS US_D,
	FORMAT(@Python3ReleaseDate, 'd', 'de-DE') AS DE_D,
	-- Fill in the locale for Indonesia
	FORMAT(@Python3ReleaseDate, 'D', 'id-ID') AS ID_D,
	FORMAT(@Python3ReleaseDate, 'D', 'zh-cn') AS CN_D;
````
***

## 📌 Section 8 : Filtering Techniques

### 8.1: Using WHERE

### Limit Results with TOP

**Limit results with TOP**
````sql
SELECT 
  TOP 3 TaxRate
FROM Sales.SalesTaxRate
ORDER BY TaxRate DESC;
````

**Limit results with TOP PERCENT**

Specify the number of percentage of results to generate. For example, to generate the first 50% of the results, use `TOP 50 PERCENT`. 

````sql
SELECT 
  TOP 50 PERCENT TaxRate, 
  Name
FROM Sales.SalesTaxRate
ORDER BY TaxRate DESC;
````

**Limit results with TOP X WITH TIES**

Results include multiple records of the same values from the last record. 

For example, we are interested to know the Top 5 students in the classroom. Since there are 2 students who have received the same 5th highest score in the classroom, hence there will be a total of 6 rows of records in the results table - 1, 2, 3, 4, 5, 5.

````sql
SELECT 
  TOP 5 WITH TIES student_name, 
  score
FROM classroom
ORDER BY score;
````

### 8.2: Remove duplicates with DISTINCT

To remove duplicates and retrieve unique values only.

````sql
SELECT 
  DISTINCT City, 
  StateProvinceID
FROM Person.Address
ORDER BY City;
````
### 8.3: Using ISNUMERIC()

To return values that are numeric only.

````sql
SELECT 
	ISNUMERIC(student_score)
FROM School.Scores;
````

### 8.4: Comparison operators

| Comparison Operator | Description |
| ------------------- | ----------- |
| =                   | Equal to    |
| !=                  | Not equal to    |
| <>                  | Not equal to   |
| >                   | Greater than    |
| >=                  | Greater than or equal to    |
| <                   | Less than    |
| <=                  | Less than or equal to    |
| BETWEEN ... AND ... |     |
| NOT BETWEEN ... AND ... |     |
| IN (..., ..., ...)    |    |
| NOT IN (..., ..., ...)  | |

### 8.5: Match texts using LIKE and Wildcards**

````sql
WHERE first_name LIKE 'a%' -- Finds any values that starts with "a"
WHERE first_name LIKE '%a' -- Finds any values that ends with "a"
WHERE first_name LIKE '%ae%' -- Finds any values that have "ae" in the middle
WHERE first_name LIKE '_b%' -- Finds any values with "b" in the second position
WHERE first_name LIKE 'a_%_%' -- Finds any values that starts with "a" and are at least 3 characters in length
WHERE first_name LIKE 'a%o' -- Finds any values that starts with "a" and ends with "o"
WHERE first_name LIKE 'a___' -- Finds any value that starts with "a" and has 3 characters
WHERE first_name LIKE '[abc]%' -- Finds any values with "a", "b" or "c"
WHERE first_name LIKE '[a-f]%' -- Finds any values with "a" to "f"
WHERE first_name LIKE 'a[l-n]' -- Finds any values that starts with "a" and has "l", "m" or "n"
WHERE first_name LIKE 'a[c-e]__' -- Finds any values that starts with "a" and has "c", "d" or "e" in the middle and ends with 2 characters
````

***

## 📌 Section 9 : JOINS

- `INNER JOIN`: Only returns matching rows.
- `LEFT JOIN` (or `RIGHT JOIN`): All rows from main table + matches from joining table.
- `NULL`: Displayed if no match is found in `LEFT JOIN` / `RIGHT JOIN`.

### 9.1: Inner Joins

````sql
SELECT 
  p.BusinessEntityID, 
  p.FirstName, 
  p.LastName, 
  pp.PhoneNumber
FROM Person.Person AS p
JOIN Person.PersonPhone AS pp
	ON p.BusinessEntityID = pp.BusinessEntityID;
````  

### 9.2: Left Joins
![Left join](https://user-images.githubusercontent.com/116416338/197342828-61315cb4-a910-421e-ab1e-e4efbe26fb4b.png)


````sql
SELECT 
  p.BusinessEntityID, 
  p.PersonType, 
  p.FirstName, 
  p.LastName, 
  e.JobTitle
FROM Person.Person AS p
LEFT JOIN HumanResources.Employee AS e
	ON p.BusinessEntityID = e.BusinessEntityID;
````
![Left outer join](https://user-images.githubusercontent.com/116416338/197391934-9f060fca-daf8-4622-adc2-9df8d5eb50b6.png)
![Left Join special](https://user-images.githubusercontent.com/116416338/197391994-82bf806b-92ec-4cfa-8959-50cd1964e8f1.png)

### 9.3: Right Joins

````sql
SELECT 
  p.BusinessEntityID, 
  p.PersonType, 
  p.FirstName, 
  p.LastName, 
  e.JobTitle
FROM Person.Person AS p
RIGHT JOIN HumanResources.Employee AS e
	ON p.BusinessEntityID = e.BusinessEntityID;
````

### 9.4: Cross Joins

For example, table_1 has 10 rows and table_2 has 5 rows. A `CROSS JOIN` would result in 10 rows x 5 rows = 50 rows table.

````sql
SELECT 
  d.Name AS DepartmentName, 
  a.Name AS AddressName
FROM HumanResources.Department AS d
CROSS JOIN Person.AddressType AS a;
````

### 9.5: Combine results with UNION

`UNION` excludes duplicate rows in both set of queries whereas, `UNION ALL` includes any duplicate rows in the sets of queries.

To use `UNION` or `UNION ALL`, when combining data from different tables
- Select same number of columns in same order
- Columns should have same data type

````sql
SELECT ProductCategoryID, NULL AS ProductSubCategoryID, Name
-- Create an empty column to represent ProductSubCategoryID column from 2nd table. Both table data types must be same.
FROM Production.ProductCategory
UNION
SELECT ProductCategoryID, ProductSubCategoryID, Name
FROM Production.ProductSubcategory;
````

### 9.6: Return distinct rows with EXCEPT

````sql
SELECT BusinessEntityID
FROM Person.Person
WHERE PersonType <> 'EM' -- Select everyone who is not an employee
EXCEPT
SELECT BusinessEntityID
FROM Sales.PersonCreditCard; -- Everyone with credit card
````

You can achieve the same results as above using a `LEFT JOIN` below.

````sql
SELECT p.BusinessEntityID
FROM Person.Person AS p
LEFT JOIN Sales.PersonCreditCard AS c --Same results using LEFT JOIN
	ON p.BusinessEntityID = c.BusinessEntityID
WHERE p.PersonType <> 'EM' AND c.CreditCardID IS NULL;
````

### 9.7: Return common rows with INTERSECT

````sql
SELECT ProductID
FROM Production.ProductProductPhoto
INTERSECT
SELECT ProductID
FROM Production.ProductReview;
````

You can achieve the same results as above using `JOIN` below.

````sql
SELECT DISTINCT(p.ProductID) -- Same results using JOIN
FROM Production.ProductProductPhoto AS p
JOIN Production.ProductReview AS r
	ON p.ProductID = r.ProductID;
````

***

## 📌 Section 10: Create Temp Table

````sql
DROP TABLE IF EXISTS clean_weight_logs;
CREATE TEMP TABLE clean_weight_logs AS (
SELECT *
FROM health.user_logs
WHERE measure = 'weight' 
	AND measure_value > 0
	AND measure_value < 201);
````

***

## 📌 Section 11: Derived Tables & Complex Queries

Derived Tables are temporary tables that are specified in the FROM clause. Types of derived tables are:
- Common Table Expressions (CTE)
- Subquery

### 11.1: Common Table Expressions (CTE)

To create a CTE, use WITH keyword followed by the CTE name and query. The CTE will also include the definition of the table enclosed within the AS().

![CTE Complex query](https://user-images.githubusercontent.com/116416338/197349846-bcf48034-517e-4641-a003-94af212ef2f5.png)

**Solution 1**

````sql
WITH product_category_spend AS (
SELECT 
  category, 
  product, 
  SUM(spend) AS total_spend 
FROM product_spend 
WHERE transaction_date >= '2022-01-01' 
  AND transaction_date <= '2022-12-31' 
GROUP BY category, product
),
top_spend AS (
SELECT *, 
  RANK() OVER (
    PARTITION BY category 
    ORDER BY total_spend DESC) AS ranking 
FROM product_category_spend)

SELECT category, product, total_spend 
FROM top_spend 
WHERE ranking <= 2 
ORDER BY category, ranking;
````

**Solution 2**

````sql
SELECT 
  category, 
  product, 
  total_spend 
FROM (
    SELECT 
      *, 
      RANK() OVER (
        PARTITION BY category 
        ORDER BY total_spend DESC) AS ranking 
    FROM (
        SELECT 
          category, 
          product, 
          SUM(spend) AS total_spend 
        FROM product_spend 
        WHERE transaction_date >= '2022-01-01' 
          AND transaction_date <= '2022-12-31' 
        GROUP BY category, product) AS total_spend
  ) AS top_spend 
WHERE ranking <= 2 
ORDER BY category, ranking;
````
**Solution 3**

````sql
Select 
category,
product,
total_spend
from
(SELECT category, 
product, 
sum(spend) as total_spend,
Rank() over (Partition BY category ORDER BY sum(spend) desc) AS Rank
from product_spend
where date_part('year',transaction_date) = 2022
group by category, product) as a
where rank<=2
````

### 11.2: Subquery

**Subquery in SELECT**

````sql
SELECT 
  BusinessEntityID, 
  SalesYTD, 
	  (SELECT MAX(SalesYTD) 
	  FROM Sales.SalesPerson) AS HighestSalesYTD,
	  (SELECT MAX(SalesYTD) 
	  FROM Sales.SalesPerson) - SalesYTD AS SalesGap
FROM Sales.SalesPerson
ORDER BY SalesYTD DESC
````

**Subquery in HAVING**

````sql
SELECT 
  SalesOrderID, 
  SUM(LineTotal) AS OrderTotal
FROM Sales.SalesOrderDetail
GROUP BY SalesOrderID
HAVING SUM(LineTotal) > 
	(
	SELECT 
    AVG(ResultTable.MyValues) AS AvgValue -- subsquery no. 2. Cannot combine both subqueries as cannot do AVG(SUM(LineTotal))
	FROM
		(SELECT SUM(LineTotal) AS MyValues --subquery no. 1
		FROM Sales.SalesOrderDetail
		GROUP BY SalesOrderID) AS ResultTable
	);
````

**Correlated Subquery**

````sql
SELECT 
  BusinessEntityID, 
  FirstName, 
  LastName,
	  (SELECT 
      JobTitle
	  FROM HumanResources.Employee
	  WHERE BusinessEntityID = MyPeople.BusinessEntityID) AS JobTitle --Better to use LEFT JOIN as give same results
FROM Person.Person AS MyPeople
WHERE EXISTS 
  (SELECT JobTitle
		FROM HumanResources.Employee
		WHERE BusinessEntityID = MyPeople.BusinessEntityID); -- Better to use JOIN or WHERE EXISTS

````

**Examples of Subquery**

A subquery is a select query within another query.
  - **Question: Find (SQL MOSH Databases) the products where the unit price is greater than the product Lettuce with id = 3**
    ```sql
    SELECT *
    FROM products
    WHERE unit_price > (SELECT unit_price
                        FROM products
                        WHERE product_id = 3)
    ```
   
  - **Question: In sql_hr database, find employees who earn more than average**
    ```sql
    SELECT *
    FROM employees
    WHERE salary > (SELECT AVG(salary)
                    FROM employees)
    ORDER BY employee_id
    ```
  - **Question: In sql_hr database, find employees who earn the highest salary in each department- Multiple column, multiple row subquery - **
    ```sql
    SELECT 
    *
    FROM employees
    WHERE (department, salary) in (select dept_name, max(salary) 
    				from employee 
				group by dept_name);
    ```   
  - **Question: In sql_hr database, find department who do not have any employees - Single Column , Multiple row subquery-**
    ```sql
    SELECT 
    *
    FROM department
    WHERE dept_name not in (select distinct dept_name 
    				from employee);
    ```   
**Using IN operator to write subqueries** 

  - **Ques: Find the products that have never been ordered**

    ```sql
    SELECT *
    FROM products
    WHERE product_id NOT IN (SELECT DISTINCT product_id
                            FROM order_items)
    ```

  - **Ques: Find clients without invoices**

    ```sql
    SELECT *
    FROM clients
    WHERE client_id NOT IN (SELECT DISTINCT client_id
                            FROM invoices)
    ```
    
 **SUBQUERIES vs JOINS** 

  - Quite often, we can write subqueries using JOIN clauses.
  - **Ques: same as above**

    ```sql
    SELECT *
    FROM clients c
    LEFT JOIN invoices i USING (client_id)
    WHERE i.invoice_id IS NULL
    ```

  - **Ques: Find customers who have ordered lettuce (id = 3), select customer_id, first_name, last_name**

    ```sql
    SELECT DISTINCT customer_id,
                    first_name,
                    last_name
    FROM customers c
            JOIN orders o USING (customer_id)
            JOIN order_items oi USING (order_id)
    WHERE oi.product_id = 3
    ```

 **ALL Keyword** 
 
  - **Ques: Select invoices larger than all invoices of client 3**

    ```sql
    SELECT *
    FROM invoices
    WHERE invoice_total > (SELECT MAX(invoice_total)
                          FROM invoices
                          WHERE client_id = 3)

    -- Above is equivalent to

    SELECT *
    FROM invoices
    WHERE invoice_total > ALL (SELECT invoice_total
                              FROM invoices
                              WHERE client_id = 3)

    ```
 **ANY/SOME Keyword** 

  - **Ques: Select clients with at least two invoices**

    ```sql
    SELECT c.client_id,
           c.name
    FROM clients c
    WHERE c.client_id IN (SELECT client_id
                        FROM invoices
                        GROUP BY client_id
                        HAVING COUNT(*) >= 2)

    -- Equivalent to

    SELECT c.client_id,
           c.name
    FROM clients c
    WHERE c.client_id = (ANY/SOME) (SELECT client_id
                             FROM invoices
                             GROUP BY client_id
                             HAVING COUNT(*) >= 2)

    -- Equivalent to

    SELECT c.client_id,
       c.name
    FROM invoices i
    JOIN clients c USING (client_id)
    GROUP BY client_id
    HAVING COUNT(*) >= 2
    ```
**CORRELATED Queries**

  - **Ques: Select employees whose salary is above the average in their office**

    ```sql
    SELECT *
    FROM employees e
    WHERE salary > (SELECT AVG(salary)
                    FROM employees
                    WHERE office_id = e.office_id)

    -- In the first run, the first employee is selected and then the subquery condition runs for it. The subquery condition is dependent upon the outer query using the WHERE clause. As such, it calculates the average salary of all employees where their office_id = outer employee's office_id. In short, it averages the salary office wise in each iteration, rather than just once that happens when there is no correlation.

    -- This constant iterative execution causes the query to run slow sometimes depending upon the number of data records.

    -- Still quite powerfull having lot of application in real world
    ```
    
  - **Ques: find the employees in each department who earn more than the average salary in that department**
    ```sql
    SELECT *
    FROM employee e1 e2
    WHERE salary > (SELECT AVG(salary)
                          FROM employee
                          WHERE e2.dept_name = e1.dept_name)
    ```
**The EXISTS Operator**

  - When the IN operator subquery returns a large result set of values, it's better to use the EXISTS operator which doesn't return the result set.
  - EXISTS works better in above case as its fast in comparison to IN Operator.

    ```sql
    SELECT *
    FROM clients c
    WHERE EXISTS(SELECT client_id
                FROM invoices
                WHERE c.client_id = client_id)

    --

    SELECT *
    FROM products
    WHERE NOT EXISTS(
            SELECT product_id
            FROM order_items
            WHERE products.product_id = order_items.product_id
        )
    ```
***

## 📌 Section 12: Window Functions (First_Value, Last_value, Lead(), Lag(),Ranking, Rows between, Moving Average, Rollup, cube)

Window functions are used by
- Creating window with `OVER` clause
- `PARTITION BY` to create the frame. If do not include PARTITION BY, the frame is created for entire table.
- ORDER BY to arrange the results

### 12.1: FIRST_VALUE() and LAST_VALUE() 

- `FIRST_VALUE()` - returns first value in the window
- `LAST_VALUE()` - returns last value in the window
- `ORDER BY` is compulsory as it has to determine the first or last value.

````sql
SELECT 
  TerritoryName, OrderDate, 
  FIRST_VALUE(OrderDate) -- Select the first value in each partition
    OVER(PARTITION BY TerritoryName ORDER BY OrderDate) AS FirstOrder  -- Create the partitions and arrange the rows
FROM Orders;
````

### 12.2: LEAD() and LAG()

- `LEAD()` - returns value in the next row
- `LAG()` - returns value in the preceding row
- `ORDER BY` is compulsory as it has to determine the next or preceding value.

````sql
SELECT 
  TerritoryName, OrderDate, 
  LAG(OrderDate) -- Specify the previous OrderDate in the window
    OVER(PARTITION BY TerritoryName ORDER BY OrderDate) AS PreviousOrder, -- Over the window, partition by territory & order by order date
  LEAD(OrderDate) -- Specify the next OrderDate in the window
    OVER(PARTITION BY TerritoryName ORDER BY OrderDate) AS NextOrder -- Create the partitions and arrange the rows
FROM Orders;
````

### 12.3: Ranking

- `ROW_NUMBER()` - Assigns unique numbers. Sequence is incremented by n + 1. For example, 1, 2, 3, 4, 5
- `RANK()` - Assigns same number to rows with identical sequence, skipping over the next sequence. For example, 1, 2, 2, 4, 5
- `DENSE_RANK()` - Assigns same number to rows with identical sequence, but does not skip over the next sequence. For example, 1, 2, 2, 3, 4

````sql
SELECT 
  TerritoryName, OrderDate, 
  ROW_NUMBER() 
    OVER(PARTITION BY TerritoryName ORDER BY OrderDate) AS OrderCount
FROM Orders;
````

### 12.4: FIRST_VALUE() and LAST_VALUE() 

- `FIRST_VALUE()` - returns first value in the window
- `LAST_VALUE()` - returns last value in the window

Uses RANGE BETWEEN 

- PRECEDING
- FOLLOWIING
- UNBOUNDED PRECEDING:  All rows before the current row
- UNBOUNDED FOLLOWING
- CURRENT ROW: All rows after the current row

![Range between](https://user-images.githubusercontent.com/116416338/197387388-2667fad1-bd5e-4079-b9bf-db8c2aee2a30.png)

````sql
SELECT
  Year,
  City,
  -- Get the last city in which the Olympic games were held
  LAST_VALUE(city) OVER (ORDER BY year ASC
   RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS Last_City
FROM Hosts
ORDER BY Year ASC;
````

### 12.5: ROWS BETWEEN

Eg
- ROWS BETWEEN 3 PRECEDING AND CURRENT ROW - Frame starts 3 rows before current row and ends at current row
- ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING - Frame starts 1 row before current row and ends 1 row after current row
- ROWS BETWEEN 5 PRECEDING AND 1 FOLLOWING - Frame starts 5 row before current row and ends 1 row after current row

**Difference between ROWS BETWEEN and RANGE BETWEEN**
- RANGE treats duplicates as single entity, similar to RANK() where it treats duplicate values in rows as single entity.
- Always use ROWS BETWEEN.

### 12.6: Paging

Split data into equal chunks.

- NTILE(n) OVER (ORDER BY col_name) - Splits data into x approximately equal pages, or quartiles (similar to Bin in Tableau)

````sql
SELECT
  --- Split up the distinct events into 111 unique groups
  event,
  NTILE(111) OVER (ORDER BY event ASC) AS Page
FROM Events
ORDER BY Event ASC;
````

### 12.7: Mathematical Aggregations

Can use mathematical aggregations such as `MIN()`, `MAX()`, `SUM()`, `AVG()` and `COUNT()` in Windows function.

````sql
SELECT
  SalesPerson, SalesYear, CurrentQuota,
  SUM(CurrentQuota) 
    OVER (PARTITION BY SalesYear) AS YearlyTotal,
FROM SaleGoal;
````

````sql
-- How to create a Running Total?
SELECT 
  TerritoryName, OrderDate, 
  SUM(OrderPrice)
       OVER(PARTITION BY TerritoryName ORDER BY OrderDate) AS TerritoryTotal -- MUST HAVE ORDER BY clause  
FROM Orders;
````

````sql
SELECT
  year,
  medals,
  MIN(medals) OVER (ORDER BY year ASC) AS Min_Medals
FROM France_Medals
ORDER BY Year ASC;
````

### 12.8:  Moving Average and Totals

Moving average (MA) - Average of last n periods. Use to indicate momentum/trends and eliminate seasonality.
Moving total - Sum of last n periods. Use to indicate performance.

````sql
--- Calculate the 3-year moving average of medals earned
SELECT
  Year, Medals,
  AVG(medals) OVER
    (ORDER BY Year ASC
     ROWS BETWEEN
     2 preceding AND current row) AS Medals_MA
FROM Russian_Medals
ORDER BY Year ASC;
````

````sql
-- Calculate each country's 3-game moving total
SELECT
  Year, Country, Medals,
  sum(Medals) OVER
    (PARTITION BY country
     ORDER BY Year ASC
     ROWS BETWEEN
     2 preceding AND current row) AS Medals_MA
FROM Country_Medals
ORDER BY Country ASC, Year ASC;
````

### 12.9: ROLLUP and CUBE

To calculate group-level and grand totals.
ROLLUP is a GROUP BY subclause that includes extra rows for group-level aggregations by hierachical.

CUBE works similarly to ROLLUP, but generates all possible group-level aggregations by non-hierachical.

The grand total rows are placed with 'null' values.

Country-level subtotals
````sql
-- Count the gold medals per country and gender
SELECT
  country,
  gender,
  COUNT(*) AS Gold_Awards
FROM Summer_Medals
WHERE
  Year = 2004
  AND Medal = 'Gold'
  AND Country IN ('DEN', 'NOR', 'SWE')
-- Generate Country-level subtotals
GROUP BY country, ROLLUP(gender)
ORDER BY Country ASC, Gender ASC;
````

All group-level subtotals
````sql
-- Count the medals per country and medal type
SELECT
  gender,
  medal,
  COUNT(*) AS Awards
FROM Summer_Medals
WHERE
  Year = 2012
  AND Country = 'RUS'
GROUP BY CUBE(gender, medal) -- Get all possible group-level subtotals
ORDER BY Gender ASC, Medal ASC;
````

### 12.10: Statistics

- `STDEV()` - Calculate standard deviation to understand statistical distribution of numeric columns
- `MODE` - Values that appear the most in table

````sql
SELECT
  SalesPerson, SalesYear, CurrentQuota,
  STDEV(CurrentQuota) 
    OVER (PARTITION BY SalesYear ORDER BY SalesYear) AS StdDev,
FROM SaleGoal;
````
  
````sql
-- To find mode of CurrentQuota
WITH quota_count AS (
  SELECT
    SalesPerson, SalesYear, CurrentQuota,
    ROW_NUMBER() -- To find the no. of times the same CurrentQuotavalue appears in the table
      OVER (PARTITION BY CurrentQuota ORDER BY CurrentQuota) AS QuotaList
  FROM SaleGoal)
  
SELECT 
  CurrentQuota, QuotaList as Mode
FROM quota_count
WHERE QuotaList IN (SELECT MAX(QuotaList) -- Filter to find the maximum value only
                    FROM quota_count);
````
***

## 📌Section 13: Grouping records

### 13.1: GROUP BY and COUNT

Every column in `GROUP BY` clause needs to be in `SELECT` clause.

````sql
SELECT 
  City, 
  StateProvinceID, 
  COUNT(*) AS AddressCount
FROM Person.Address
GROUP BY City, StateProvinceID
ORDER BY AddressCount DESC;
````

### 13.2: GROUP BY and HAVING

`HAVING` must be used in conjuction with `GROUP BY`.

````sql
SELECT 
  City, 
  StateProvinceID, 
  COUNT(*) AS AddressCount
FROM Person.Address
GROUP BY City, StateProvinceID
HAVING City = 'New York'
````

***

## 📌 Section 14: Functions ( Aggregate, String, Date)

### 14.1: Aggregate Functions

| Aggregate Functions     | Description                 |
| ----------------------- | --------------------------- |
| COUNT(*)                | Count total number of rows  |
| COUNT(DISTINCT column)  | Count unique values only    |
| COUNT_BIG()             |                             |
| SUM, AVG, MIN, MAX()    | Self-explanatory            |
| STDEV, VAR, VARP()      | Self-explanatory            |


### 14.2: String/ Character Functions

| String Functions  | Description                 | Syntax | 
| ----------------- | --------------------------- | ------- |
| UPPER()           | Convert 'NaMe' into UPPERCAESE  | SELECT UPPER(col_1) |
| LOWER()           | Convert 'NaMe' into lowercase   | SELECT LOWER(col_1) |
| LEN()             | Count number of characters in a value   | SELECT LEN(col_1) |
| TRIM()            | Trim blank spaces in a value            | SELECT TRIM(col_1) | 
| LEFT(,3)         | Return 1st 3 characters from the left    | SELECT LEFT(col_1,3) |
| RIGHT(,3)        | Returns 1st 3 characters from the right  | SELECT RIGHT(col_1,3) |
| INITCAP()       | Converts 1st letter of each word to uppercase  | SELECT INITCAP(film_title) |
| SUBSTRING(col,3,3) | To retrieve midsection of string        | SELECT SUBSTRING(col_1,10,10) |
| CONCAT()          | Combine >= 2 values with ' ',-,/          | SELECT CONCAT(col_1, ' ', col_2, ' ', col_3) |
| CONCAT_WS()       | To insert ' ' (blank space) between all values     | SELECT CONCAT_WS(' ', col_1, col_2, col_3) |
| REPLACE()       | Replace char in a string     | SELECT REPLACE(col, 'wrong_value', 'correct_value) |
| STRING_AGG(col, separator | Takes all values in col and concatenates with separator btw each value | SELECT STRING_AGG(country,',') |
| CHARINDEX(' ',col) | To find location of character in string        | SELECT CHARINDEX('-', col_1) |
| TOP(5) REPLACE(col,' ', ' ') | TOP specifies the rows being replaced and REPLACE replaces 1st '' with 2nd ' ' | SELECT TOP(5) REPLACE(col_1, '_', '-') |

Refer to examples below.

````sql
SELECT 
  FirstName, 
  LastName, 
	UPPER(FirstName) AS FName, LOWER(LastName) AS LName, 
	LEN(FirstName) AS LenFName,
	LEFT(LastName, 3) AS First3, RIGHT(LastName,3) AS Last3,
	TRIM(LastName) AS TrimmedName
FROM Person.Person;
````

````sql
SELECT 
  FirstName, 
  LastName,
	CONCAT(FirstName, ' ', MiddleName, ' ', LastName) AS FullName,
	CONCAT_WS(' ', FirstName, MiddleName, LastName) AS FullNameWS -- 'WS' stands for 'With Separator'
FROM Person.Person;
````

````sql
SELECT 
	TOP 10 description, 
  CHARINDEX('Weather', description) AS start_of_string, -- Find index of 'Weather' which is 8
  LEN('Weather') AS length_of_string, -- Find length of 'Weather'
  SUBSTRING(description, 15, LEN(description)) AS additional_description -- Get substring between index 15 (after 'Weather') to end of description column
FROM grid
WHERE description LIKE '%Weather%';
````

````sql
SELECT
    Country,
    RANK() OVER (ORDER BY Medals DESC) AS Rank
  FROM Country_Medals
  ORDER BY Rank ASC)

-- Compress the countries column
SELECT STRING_AGG(Country, ', ')
FROM Country_Ranks
-- Select only the top three ranks
WHERE Rank <= 3;
````

```sql
SELECT 
  SUBSTRING(address 
  	FROM POSITION(' ' IN address)+1 
	FOR LENGTH(address))   -- Select only the street name from the address table
FROM address;
```

```sql
SELECT
  LEFT(email, POSITION('@' IN email)-1) AS username, -- Extract the characters to the left of the '@'
  SUBSTRING(email -- Extract the characters to the right of the '@'
    FROM POSITION('@' IN email)+1 
    FOR LENGTH(email)) AS domain
FROM customer;
```


### 14.3: Mathematical Functions

| Mathematical Functions  | Description        | Syntax |
| ----------------- | ------------------------ | ------ |
| ROUND(,2)           | Round with 2 decimals  | ROUND(col_1, 2) |
| ROUND(,-2)           | Round up to nearest hundreds   | ROUND(col_1, -2) |
| ROUND(,0,1)           | Truncate to nearest integer   | ROUND(col_1, 0, 1) |
| CEILING()           | Round up to nearest integer     | CEILING(col_1) |
| FLOOR()            | Round down to nearest integer    | FLOOR(col_1)
| ABS()            | Returns non-negative values    | ABS(col_1)
| SQRT()            | Returns square root of a value    | SQRT(9) = 3
| SQUARE()            | Returns square to the power of 2 of value    | SQUARE(9) = 81
| LOG()            | Returns natual logarithm    | LOG(col_1, 10)

When data appears skewed, use LOG() to return to natural logarith and move skew to a normalized bell curve.
Can only apply to non-zero values in column.

````sql
SELECT 
  BusinessEntityID, 
  SalesYTD,
	ROUND(SalesYTD, 2) AS Round2, -- 2 decimals
	ROUND(SalesYTD, -2) AS Round100, -- Round up to nearest hundreds
	CEILING(SalesYTD) AS RoundCeiling, -- Round up to nearest integer
	FLOOR(SalesYTD) AS RoundFloor -- Round down to nearest integer
FROM Sales.SalesPerson;
````

````sql
SELECT 
	profit, 
	ROUND(duration_seconds, 0) AS rounded_values, 
	ROUND(duration_seconds, 0, 1) AS truncated_values
FROM incidents;
````

````sql
SELECT 
	WeightValue, 
	ABS(WeightValue) AS AbsoluteValue
        SQUARE(WeightValue) AS WeightSquare, 
        SQRT(WeightValue) AS WeightSqrt
FROM Shipments;
````

### 14.4: Date Functions

### MS SQL Server

| Date Time Functions  | Description                 | Example |
| --------------- | --------------------------- | ------- |
| YEAR()          | Returns year from date  |
| MONTH()         | Returns month from date   |
| DAY()           | Returns day from date                           |
| GETDATE()       | Returns today's local date in DATETIME      |
| GETDATEUTC()    | Returns today's UTC date in DATETIME           |
| SYSDATETIME()    | Returns date in local DATETIME2           |
| SYSUTCDATETIME()  | Returns date in UTC DATETIMEs           |
| DATEPART()  | Returns numeric value of the part we want           | DATEPART(YEAR, start_date)
| DATENAME()  | Returns string value of the month           | DATENAME(MONTH, start_date)
| DATEDIFF()      | Add or subtract datetime values. Returns date.    | DATEDIFF(DD/MM/YY, start_date, end_date)
| DATEADD()       | Obtain difference between 2 datetime values. Returns days.       | DATEADD(DD/MM/YY, 10, date_col)


**Add and Subtract Date and Time**

````sql
SELECT
	DATEADD(DD, 30, '2020-06-30'), -- Add days
	DATEADD(DD, -30, '2020-06-30'), -- Subtract days
	DATEDIFF(DD, start_date, end_date);
````

````sql
SELECT
  DATEDIFF(SECOND, start_time, end_time) AS seconds_elapsed,
  DATEDIFF(MINUTE, start_time, end_time) AS minutes_elapsed,
  DATEDIFF(HOUR, start_time, end_time) AS hours_elapsed;
````

**Break Date and Time into Component Parts*

Refer [here](https://docs.microsoft.com/en-us/sql/t-sql/functions/datepart-transact-sql) for list of parts.

````sql
DECLARE
	@BerlinWallFalls DATETIME2(7) = '1989-11-09 23:49:36.2294852';

SELECT
	DATEPART(YEAR, @BerlinWallFalls) AS TheYear,
	DATEPART(MONTH, @BerlinWallFalls) AS TheMonth,
	DATEPART(DAY, @BerlinWallFalls) AS TheDay,
	DATEPART(DAYOFYEAR, @BerlinWallFalls) AS TheDayOfYear,
	DATEPART(WEEKDAY, @BerlinWallFalls) AS TheDayOfWeek,
	DATEPART(WEEK, @BerlinWallFalls) AS TheWeek,
	DATEPART(SECOND, @BerlinWallFalls) AS TheSecond,
	DATEPART(NANOSECOND, @BerlinWallFalls) AS TheNanosecond;
````

**Working on Leap Years**

````sql
DECLARE
	@LeapDay DATETIME2(7) = '2012-02-29 18:00:00';

SELECT
	DATEADD(DAY, -1, @LeapDay) AS PriorDay,
	DATEADD(DAY, 1, @LeapDay) AS NextDay,
-- For leap years, we need to move 4 years, not just 1
	DATEADD(YEAR, -4, @LeapDay) AS PriorLeapYear,
	DATEADD(YEAR, 4, @LeapDay) AS NextLeapYear,
	DATEADD(YEAR, -1, @LeapDay) AS PriorYear
-- Move 4 years forward and one day back
	DATEADD(DAY, -1, DATEADD(YEAR, 4, @PostLeapDay)) AS NextLeapDay,
  DATEADD(DAY, -2, @PostLeapDay) AS TwoDaysAgo;
````

````sql
DECLARE
	@PostLeapDay DATETIME2(7) = '2012-03-01 18:00:00',
    	@TwoDaysAgo DATETIME2(7);

SELECT
	@TwoDaysAgo = DATEADD(DAY, -2, @PostLeapDay);

SELECT
	@TwoDaysAgo AS TwoDaysAgo,
	@PostLeapDay AS SomeTime,
	DATEDIFF(DAY, @TwoDaysAgo, @PostLeapDay) AS DaysDifference,
	DATEDIFF(HOUR, @TwoDaysAgo, @PostLeapDay) AS HoursDifference,
	DATEDIFF(MINUTE, @TwoDaysAgo, @PostLeapDay) AS MinutesDifference;
````

**Rounding dates to DAY, HOUR and MINUTE**

````sql
DECLARE
	@SomeTime DATETIME2(7) = '2018-06-14 16:29:36.2248991';

SELECT
	DATEADD(DAY, DATEDIFF(DAY, 0, @SomeTime), 0) AS RoundedToDay,
	DATEADD(HOUR, DATEDIFF(HOUR, 0, @SomeTime), 0) AS RoundedToHour,
	DATEADD(MINUTE, DATEDIFF(MINUTE, 0, @SomeTime), 0) AS RoundedToMinute;
````

### PostgreSQL
| Date Time Functions  | Description                 | Example |
| --------------- | --------------------------- | ------- |
| EXTRACT     | Extracts year, month or day in numerical  | EXTRACT(MONTH FROM date_col) |
| DATE_PART()     | Extracts year, month or day only  | DATE_PART('Month', date_col) |
| DATE_TRUNC()    | Returns year, month, day in specified precision | DATE_TRUNC('Month',date_col) |
| AGE()       | Find difference between timestamped dates | SELECT AGE(timestamp_1, timestamp2) |
| TO_CHAR()       | Extracts month or day in date and cast in name  | TO_CHAR(date_col, 'Month') |
| INTERVAL ' '    | Adds date column with time period in year, month, days, hours, etc | SELECT date + INTERVAL '3 DAYS' AS expected_return_date |
| CURRENT_DATE       | Extracts current date  |  |
| CURRENT_TIMESTAMP() | Retrieve date timestamped with time zone and specify precision. Same as NOW() | SELECT CURRENT_TIMESTAMP(2) |
| NOW()       | Retrieve date timestamped with time zone and microsecond precision |  |


````sql
SELECT
  DATE_PART('month',start_date) AS month_date,
  TO_CHAR(start_date, 'Month') AS month_name,
  COUNT(*) AS trial_subscriptions
FROM foodie_fi.subscriptions s
JOIN foodie_fi.plans p
  ON s.plan_id = p.plan_id
WHERE s.plan_id = 0
GROUP BY DATE_PART('month',start_date), TO_CHAR(start_date, 'Month')
ORDER BY month_date;
````

<img width="403" alt="image" src="https://user-images.githubusercontent.com/81607668/129827317-21a487e9-a72d-4442-b140-87d510198ab3.png">

````sql
SELECT 
    c.order_id, 
    c.order_time, 
    r.pickup_time, 
    DATEDIFF(MINUTE, c.order_time, r.pickup_time) AS pickup_minutes
  FROM #customer_orders AS c
  JOIN #runner_orders AS r
    ON c.order_id = r.order_id
  WHERE r.distance != 0
  GROUP BY c.order_id, c.order_time, r.pickup_time
````

````sql
SELECT
	rental_date,
	return_date,
	rental_date + INTERVAL '3 DAYS' AS expected_return_date
FROM rental;
````

```sql
SELECT 
  EXTRACT(DAY FROM rental_date) AS dayofweek,   -- Extract the day of week date part from the rental_date
  AGE(return_date, rental_date) AS rental_days
FROM rental AS r 
WHERE rental_date BETWEEN CAST('2005-05-01' AS date)   -- Use an INTERVAL for the upper bound of the rental_date 
   AND CAST('2005-05-01' AS date) + INTERVAL '90 day';
```

```sql
SELECT 
  c.first_name || ' ' || c.last_name AS customer_name, -- Same as CONCAT(c.first_name, c.last_name, ' ')
  f.title,
  r.rental_date,
  EXTRACT(dow FROM r.rental_date) AS dayofweek,   -- Extract the day of week date part from the rental_date
  AGE(r.return_date, r.rental_date) AS rental_days,
  CASE WHEN DATE_TRUNC('day', AGE(r.return_date, r.rental_date)) >   -- Use DATE_TRUNC to get days from the AGE function
    f.rental_duration * INTERVAL '1' day   -- Calculate number of d
  THEN TRUE ELSE FALSE END AS past_due 
FROM film AS f 
INNER JOIN inventory AS i 
	ON f.film_id = i.film_id 
INNER JOIN rental AS r 
	ON i.inventory_id = r.inventory_id 
INNER JOIN customer AS c 
	ON c.customer_id = r.customer_id 
WHERE r.rental_date BETWEEN CAST('2005-05-01' AS DATE) -- Use an INTERVAL for the upper bound of the rental_date 
  AND CAST('2005-05-01' AS DATE) + INTERVAL '90 day';
```

***

## 📌Section 14: Formatting Functions (Cast, Convert)

### 14.1: CAST()
- Convert one date type to another data type, including date types.
- Used in SQL Server, PostgreSQL, MySQL.

### 14.2: CONVERT()
- Convert one date type to another data type, including date types.
- Used in SQL Server only.

````sql
CONVERT(NVARCHAR(30, date_col, 0) as default_date_time,
CONVERT(NVARCHAR(30, date_col, 1) as US_mmddyy,
CONVERT(NVARCHAR(30, date_col, 101) as US_mmddyyyy,
CONVERT(NVARCHAR(30, date_col, 120) as yyyymmdd_time
````

### 14.3: FORMAT()
- Used in SQL Server only.

````sql
FORMAT(date_col, 'yyyy-MM-dd')
````

***

## 📌 Section 15: Condition statement

### 15.1: IF 

````sql
SELECT 
  IIF (SalesYTD > 2000000, 'Met sales goal', 'Has not met goal') AS Status,
	COUNT(*)
FROM Sales.SalesPerson
GROUP BY IIF (SalesYTD > 2000000, 'Met sales goal', 'Has not met goal');
````

### 15.2: If Else

### CASE Statement

`CASE` statement is use for
- Create value groups, or bins for continuous values

````sql
SELECT 
	DurationSeconds, 
  CASE WHEN (DurationSeconds <= 120) THEN 1          
	WHEN (DurationSeconds > 120 AND DurationSeconds <= 600) THEN 2            
  WHEN (DurationSeconds > 601 AND DurationSeconds <= 1200) THEN 3              
  WHEN (DurationSeconds > 1201 AND DurationSeconds <= 5000) THEN 4     
  ELSE 5
  END AS SecondGroup   
FROM Incidents;

````

***

## 📌 PIVOT

Before Pivot
````sql
SELECT 
	country, year, 
	COUNT(*) AS awards
FROM summer_medals
WHERE country IN ('CHN', 'RUS', 'USA')
	AND year IN (2008, 2012)
	AND medal = 'Gold'
GROUP BY country, year
ORDER BY country, year;
````

After Pivot
````sql
CREATE EXTENSION IF NOT EXISTS tablefunc;

SELECT *
FROM CROSSTAB
($$
	SELECT 
		country, year, 
		COUNT(*)::INTEGER AS awards
FROM summer_medals
WHERE country IN ('CHN', 'RUS', 'USA')
	AND year IN (2008, 2012)
	AND medal = 'Gold'
GROUP BY country, year
ORDER BY country, year;
$$) AS ct
(country VARCHAR, "2008" INTEGER, "2012" INTEGER)

ORDER BY country;
````

````sql
CREATE EXTENSION IF NOT EXISTS tablefunc;

SELECT * FROM CROSSTAB($$
  WITH Country_Awards AS (
    SELECT
      Country,
      Year,
      COUNT(*) AS Awards
    FROM Summer_Medals
    WHERE
      Country IN ('FRA', 'GBR', 'GER')
      AND Year IN (2004, 2008, 2012)
      AND Medal = 'Gold'
    GROUP BY Country, Year)

  SELECT
    Country,
    Year,
    RANK() OVER
      (PARTITION BY Year
       ORDER BY Awards DESC) :: INTEGER AS rank
  FROM Country_Awards
  ORDER BY Country ASC, Year ASC;
$$) AS ct (Country VARCHAR,
           "2004" INTEGER,
           "2008" INTEGER,
           "2012" INTEGER)

Order by Country ASC;
````


````sql
SELECT 
  'AverageListPrice' AS 'ProductLine', -- To add meaning to the table
FROM
	(SELECT 
    ProductLine, 
    ListPrice
	FROM Production.Product) AS SourceData
  PIVOT (AVG(ListPrice) FOR ProductLine IN (M, R, S, T)) AS PivotTable; -- For every product line M, R, S, T we are going to find the average price
````

***

## 📌 Create and Use Variables

Use Variables to avoid repetition of creating different queries using same variable. 

For example, having to create different queries to find for specific artists. Instead, we assign/declare placeholder `=@my_artist`. All we need to do is update the variable each time we run the query.

Syntax example
````sql
-- To declare a variable
DECLARE @col_1 INT
DECLARE @my_name VARCHAR(100)

-- To assign value to variable
SET @col_1 = 5
SET @my_name = 'Katie'
````

### Using DECLARE

````sql
DECLARE @MyFirstVariable INT;

SET @MyFirstVariable = 10;

SELECT 
	@MyFirstVariable AS MyValue, 
	@MyFirstVariable * 5 AS Multiplication,
	@MyFirstVariable + 10 AS Addition;

````sql
DECLARE @VarColor VARCHAR(20) = 'Blue';

SELECT ProductID, Name, ProductNumber, Color, ListPrice
FROM Production.Product
WHERE Color = @VarColor;
````

### Create counter for WHILE loops

- WHILE evaluates a true or false condition.
- After WHILE, there should be a line with BEGIN.
- Next, include code to run until condition in WHILE loop is true.
- After code, add END. 
- BREAK will cause exit out of the loop and CONTINUE will cause the loop to continue.

````sql
DECLARE @Counter INT = 1;

WHILE @Counter <= 3 -- 1. Starts with WHILE.
BEGIN -- 2. Then, BEGIN.
	SELECT @Counter AS CurrentValue -- 3. Specify a condition to be true or false.
	SET @Counter = @Counter + 1
END; -- 4. END the loop.
SELECT @Counter -- Print the Counter column
````

````sql
DECLARE @Counter INT = 1;
DECLARE @Product INT = 710;

WHILE @Counter <= 3
BEGIN
	SELECT ProductID, Name, ProductNumber, Color, ListPrice
	FROM Production.Product
	WHERE ProductID = @Product
	SET @Counter = @Counter + 1
	SET @Product = @Product + 10
END;
````

````sql
DECLARE @ctr INT -- Declare @ctr as integer
SET @ctr = 1
WHILE @ctr < 10 -- Specify the condition
	BEGIN
	SET @ctr = @ctr + 1 - Keep incrementing until @ctr < 10
	IF @ctr = 4 -- Check if @ctr = 4
		BREAK -- When @ctr = 4, loop breaks
END;
SELECT @ctr -- Result will be 4 because of BREAK condition
````
***




