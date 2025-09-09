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

We add the value to get the cumulative values in order to see if the business in growing or declining.

11. We calculate the total sales per month and the running total sales over time with a window function using a subquery:
```sql
SELECT
ORDER_DATE,
TOTAL_SALES,
SUM(TOTAL_SALES) OVER (ORDER BY ORDER_DATE) AS RUNNING_TOTAL_SALES
FROM
(
SELECT
DATETRUNC(MONTH, ORDER_DATE) AS ORDER_DATE,
SUM(SALES_AMOUNT) AS TOTAL_SALES
FROM GOLD.FACT_SALES
WHERE ORDER_DATE IS NOT NULL
GROUP BY DATETRUNC(MONTH, ORDER_DATE)
)T
```
12. We can also calculate the moving average of the price:
```sql
SELECT
ORDER_DATE,
TOTAL_SALES,
SUM(TOTAL_SALES) OVER (ORDER BY ORDER_DATE) AS RUNNING_TOTAL_SALES,
AVG(AVG_PRICE) OVER (ORDER BY ORDER_DATE) AS MOVING_AVG_PRICE
FROM
(
SELECT
DATETRUNC(MONTH, ORDER_DATE) AS ORDER_DATE,
SUM(SALES_AMOUNT) AS TOTAL_SALES,
AVG(PRICE) AS AVG_PRICE
FROM GOLD.FACT_SALES
WHERE ORDER_DATE IS NOT NULL
GROUP BY DATETRUNC(MONTH, ORDER_DATE)
)T
```

### PERFORMANCE ANALYSIS

We usually use performance analysis to compare a current measure vs a target measure like current sales vs average sales, current year sales vs previous year sales (YOY) or current sales vs lowest sales etc. We usually use window functions.

13. Let's try to analize yearly performance of products by comparing their sales to both the avegare sales performance and previous year sales. This is a really common practice and notice that we will also need to join our tables:
```sql
WITH YEARLY_PRODUCT_SALES AS(
SELECT
YEAR(F.ORDER_DATE) AS ORDER_YEAR,
P.PRODUCT_NAME,
SUM(F.SALES_AMOUNT) AS CURRENT_SALES
FROM GOLD.FACT_SALES F
LEFT JOIN GOLD.DIM_PRODUCTS P
ON F.PRODUCT_KEY = P.PRODUCT_KEY
WHERE F.ORDER_DATE IS NOT NULL
GROUP BY YEAR(F.ORDER_DATE),P.PRODUCT_NAME
)
SELECT 
ORDER_YEAR,
PRODUCT_NAME,
CURRENT_SALES,
AVG(CURRENT_SALES) OVER(PARTITION BY PRODUCT_NAME) AS AVG_SALES,
CURRENT_SALES - AVG(CURRENT_SALES) OVER(PARTITION BY PRODUCT_NAME) AS DIFF_AVG,
CASE WHEN CURRENT_SALES - AVG(CURRENT_SALES) OVER(PARTITION BY PRODUCT_NAME) > 0 THEN 'ABOVE AVG'
     WHEN CURRENT_SALES - AVG(CURRENT_SALES) OVER(PARTITION BY PRODUCT_NAME) < 0 THEN 'BELOW AVG'
	 ELSE 'AVG'
END AVG_CHANGE,
-- YOY (YEAR OVER YEAR ANALYSIS). IF WE MODIFY IN THE WITH STATEMENT YEAR BY MONTH WE CALCULATE MOM (MONTH OVER MONTH):
LAG(CURRENT_SALES) OVER (PARTITION BY PRODUCT_NAME ORDER BY ORDER_YEAR) AS PREV_YEAR_SALES,
CURRENT_SALES - LAG(CURRENT_SALES) OVER (PARTITION BY PRODUCT_NAME ORDER BY ORDER_YEAR) AS DIFF_PREV_YEAR,
CASE WHEN CURRENT_SALES - LAG(CURRENT_SALES) OVER (PARTITION BY PRODUCT_NAME ORDER BY ORDER_YEAR) > 0 THEN 'INCREASE'
     WHEN CURRENT_SALES - LAG(CURRENT_SALES) OVER (PARTITION BY PRODUCT_NAME ORDER BY ORDER_YEAR) < 0 THEN 'DECREASE'
	 ELSE 'NO CHANGE'
END PREV_YEAR_CHANGE
FROM YEARLY_PRODUCT_SALES
ORDER BY PRODUCT_NAME, ORDER_YEAR
```

### PART-TO-WHOLE ANALYSIS

We use this kind of analysis, for example, to check how one product contributes to the total sales. Like in the previous exercise we can try the same analysis with our customer or number of orders.

14. In this case we are going to try to see what categories contribute the most to the total sales. We can see the top performing category are, by far, bikes:
```sql
WITH CATEGORY_SALES AS(
SELECT
CATEGORY,
SUM(SALES_AMOUNT) TOTAL_SALES
FROM GOLD.FACT_SALES F
LEFT JOIN GOLD.DIM_PRODUCTS P
ON P.PRODUCT_KEY = F.PRODUCT_KEY
GROUP BY CATEGORY)

SELECT
CATEGORY,
TOTAL_SALES,
SUM(TOTAL_SALES) OVER() OVERALL_SALES,
CONCAT(ROUND((CAST (TOTAL_SALES AS FLOAT)/SUM(TOTAL_SALES) OVER())*100, 2), '%') AS PERCENTAGE_OF_TOTAL
FROM CATEGORY_SALES
ORDER BY TOTAL_SALES DESC
```

### DATA SEGMENTATION

We group the data based on a specific range. For example we can analyse the total products by sale range, total customers by age group etc usin Case When.

15. We are going to segment products into cost ranges and count how many products fall into each segment:
```sql
WITH PRODUCT_SEGMENTS AS(
SELECT 
PRODUCT_KEY,
PRODUCT_NAME,
COST,
CASE WHEN COST < 100 THEN 'BELOW 100'
	 WHEN COST BETWEEN 100 AND 500 THEN '100-500'
	 WHEN COST BETWEEN 500 AND 1000 THEN '500-1000'
	 ELSE 'ABOVE 1000'
END COST_RANGE
FROM GOLD.DIM_PRODUCTS)

SELECT 
COST_RANGE,
COUNT(PRODUCT_KEY) AS TOTAL_PRODUCTS
FROM PRODUCT_SEGMENTS
GROUP BY COST_RANGE
ORDER BY TOTAL_PRODUCTS DESC
```
