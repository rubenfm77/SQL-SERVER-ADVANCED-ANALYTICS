# SQL SERVER ADVANCED ANALYTICS in progress

Here you'll find SQL queries very helpful for sales advanced analytics, in this case, for one of my favourite subjects/objects: bikes. Check out Baraa profile and follow him on Youtube and Github: https://github.com/DataWithBaraa

Queries are better suited for SQL server. First we create the databases:

Script Purpose:
    This script creates a new database named 'DataWarehouseAnalytics' after checking if it already exists. 
    If the database exists, it is dropped and recreated. Additionally, this script creates a schema called gold
	
WARNING:
    Running this script will drop the entire 'DataWarehouseAnalytics' database if it exists. 
    All data in the database will be permanently deleted. Proceed with caution 
    and ensure you have proper backups before running this script.
*/
```sql
USE master;
GO
```
Drop and recreate the 'DataWarehouseAnalytics' database:
```sql
IF EXISTS (SELECT 1 FROM sys.databases WHERE name = 'DataWarehouseAnalytics')
BEGIN
    ALTER DATABASE DataWarehouseAnalytics SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
    DROP DATABASE DataWarehouseAnalytics;
END;
GO
```
Create the 'DataWarehouseAnalytics' database:

```sql
CREATE DATABASE DataWarehouseAnalytics;
GO

USE DataWarehouseAnalytics;
GO
```
Create Schemas:

```sql
CREATE SCHEMA gold;
GO
```

```sql
CREATE TABLE gold.dim_customers(
	customer_key int,
	customer_id int,
	customer_number nvarchar(50),
	first_name nvarchar(50),
	last_name nvarchar(50),
	country nvarchar(50),
	marital_status nvarchar(50),
	gender nvarchar(50),
	birthdate date,
	create_date date
);
GO
```
```sql
CREATE TABLE gold.dim_products(
	product_key int ,
	product_id int ,
	product_number nvarchar(50) ,
	product_name nvarchar(50) ,
	category_id nvarchar(50) ,
	category nvarchar(50) ,
	subcategory nvarchar(50) ,
	maintenance nvarchar(50) ,
	cost int,
	product_line nvarchar(50),
	start_date date 
);
GO
```
```sql
CREATE TABLE gold.fact_sales(
	order_number nvarchar(50),
	product_key int,
	customer_key int,
	order_date date,
	shipping_date date,
	due_date date,
	sales_amount int,
	quantity tinyint,
	price int 
);
GO
```
```sql
TRUNCATE TABLE gold.dim_customers;
GO
```

```sql
BULK INSERT gold.dim_customers
FROM 'C:\sql\sql-data-analytics-project\datasets\csv-files\gold.dim_customers.csv'
WITH (
	FIRSTROW = 2,
	FIELDTERMINATOR = ',',
	TABLOCK
);
GO
```
```sql
TRUNCATE TABLE gold.dim_products;
GO
```
```sql
BULK INSERT gold.dim_products
FROM 'C:\sql\sql-data-analytics-project\datasets\csv-files\gold.dim_products.csv'
WITH (
	FIRSTROW = 2,
	FIELDTERMINATOR = ',',
	TABLOCK
);
GO
```
```sql
TRUNCATE TABLE gold.fact_sales;
GO
```
```sql
BULK INSERT gold.fact_sales
FROM 'C:\sql\sql-data-analytics-project\datasets\csv-files\gold.fact_sales.csv'
WITH (
	FIRSTROW = 2,
	FIELDTERMINATOR = ',',
	TABLOCK
);
GO
```
### EXPLORATORY ANALYSIS OF TRENDS OVER TIME

1. We can analyse sales by date and sales amount, but we see that there are several order_date with null values:
```sql
SELECT 
ORDER_DATE,
SALES_AMOUNT
FROM GOLD.FACT_SALES
ORDER BY ORDER_DATE
```
2. We repeat the query avoiding null values:
```sql
SELECT 
ORDER_DATE,
SALES_AMOUNT
FROM GOLD.FACT_SALES
WHERE ORDER_DATE IS NOT NULL
ORDER BY ORDER_DATE
```
3. We can aggregate the sales and group them by date:
```sql
SELECT 
ORDER_DATE,
SUM(SALES_AMOUNT) AS TOTAL_SALES
FROM GOLD.FACT_SALES
WHERE ORDER_DATE IS NOT NULL
GROUP BY ORDER_DATE
ORDER BY ORDER_DATE
```
4. Some companies want to monitor daily sales for budgeting, but in general we aggregate at month or year level with date functions. Note that we need to use the date function in group and order by to get the correct results. We can see that sales in 2014 are really low compared with previous years:
```sql
SELECT 
YEAR(ORDER_DATE) AS ORDER_YEAR,
SUM(SALES_AMOUNT) AS TOTAL_SALES
FROM GOLD.FACT_SALES
WHERE ORDER_DATE IS NOT NULL
GROUP BY YEAR(ORDER_DATE)
ORDER BY YEAR(ORDER_DATE)
```
5. We'll try to dig deeper into our data and see if there's any change in the number of customers. We can see a huge decrease in sales during 2014:
```sql
SELECT 
YEAR(ORDER_DATE) AS ORDER_YEAR,
SUM(SALES_AMOUNT) AS TOTAL_SALES,
COUNT(DISTINCT CUSTOMER_KEY) AS TOTAL_CUSTOMERS
FROM GOLD.FACT_SALES
WHERE ORDER_DATE IS NOT NULL
GROUP BY YEAR(ORDER_DATE)
ORDER BY YEAR(ORDER_DATE)
```
6. We can also check the quantity:
```sql
SELECT 
YEAR(ORDER_DATE) AS ORDER_YEAR,
SUM(SALES_AMOUNT) AS TOTAL_SALES,
COUNT(DISTINCT CUSTOMER_KEY) AS TOTAL_CUSTOMERS,
SUM(QUANTITY) AS TOTAL_QUANTITY
FROM GOLD.FACT_SALES
WHERE ORDER_DATE IS NOT NULL
GROUP BY YEAR(ORDER_DATE)
```
7. We can also check the performance by month changing year for month, where we can see the seasonality with the most sales in december and least sales in February:
```sql
MONTH(ORDER_DATE) AS ORDER_MONTH,
SUM(SALES_AMOUNT) AS TOTAL_SALES,
COUNT(DISTINCT CUSTOMER_KEY) AS TOTAL_CUSTOMERS,
SUM(QUANTITY) AS TOTAL_QUANTITY
FROM GOLD.FACT_SALES
WHERE ORDER_DATE IS NOT NULL
GROUP BY MONTH(ORDER_DATE)
ORDER BY MONTH(ORDER_DATE)
```
8. Seems a good idea to check the previous variables, not only by month but also by year:
```sql
SELECT
YEAR(ORDER_DATE) AS ORDER_YEAR,
MONTH(ORDER_DATE) AS ORDER_MONTH,
SUM(SALES_AMOUNT) AS TOTAL_SALES,
COUNT(DISTINCT CUSTOMER_KEY) AS TOTAL_CUSTOMERS,
SUM(QUANTITY) AS TOTAL_QUANTITY
FROM GOLD.FACT_SALES
WHERE ORDER_DATE IS NOT NULL
GROUP BY YEAR(ORDER_DATE), MONTH(ORDER_DATE)
ORDER BY YEAR(ORDER_DATE), MONTH(ORDER_DATE)
```
9. Seems a good idea to check the previous variables, not only by month but also by year:
```sql
SELECT
YEAR(ORDER_DATE) AS ORDER_YEAR,
MONTH(ORDER_DATE) AS ORDER_MONTH,
SUM(SALES_AMOUNT) AS TOTAL_SALES,
COUNT(DISTINCT CUSTOMER_KEY) AS TOTAL_CUSTOMERS,
SUM(QUANTITY) AS TOTAL_QUANTITY
FROM GOLD.FACT_SALES
WHERE ORDER_DATE IS NOT NULL
GROUP BY YEAR(ORDER_DATE), MONTH(ORDER_DATE)
ORDER BY YEAR(ORDER_DATE), MONTH(ORDER_DATE)
```
10. We can truncate month and year with Datetrunc function:
```sql
SELECT
DATETRUNC(MONTH, ORDER_DATE) AS ORDER_YEAR,
SUM(SALES_AMOUNT) AS TOTAL_SALES,
COUNT(DISTINCT CUSTOMER_KEY) AS TOTAL_CUSTOMERS,
SUM(QUANTITY) AS TOTAL_QUANTITY
FROM GOLD.FACT_SALES
WHERE ORDER_DATE IS NOT NULL
GROUP BY DATETRUNC(MONTH, ORDER_DATE)
ORDER BY DATETRUNC(MONTH, ORDER_DATE)
```

### CUMULATIVE ANALYSIS
