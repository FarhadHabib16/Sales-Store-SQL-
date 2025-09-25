# Sales-Store-SQL
CREATE TABLE sales_store
(
transaction_id VARCHAR(15),
customer_id VARCHAR(15),
customer_name VARCHAR(30),
customer_age int,
gender VARCHAR(15),
product_id VARCHAR(15),
product_name VARCHAR(15),
product_category VARCHAR(15),
quantiy int,
prce INT,
payment_mode VARCHAR(15),
purchase_date DATE,
time_of_purchase TIME,
status VARCHAR(15)
)

SELECT * FROM sales_store
SET DATEFORMAT dmy
BULK INSERT sales_store
FROM 'C:\Users\farha\Downloads\archive (2)\sales_store.csv'
	WITH (
	FIRSTROW = 2,
	FIELDTERMINATOR=',',
	ROWTERMINATOR='\n'
	);

--- Data Cleaning
SELECT *
FROM sales_store

SELECT * 
INTO store 
FROM sales_store

--- Checking duplicate
WITH CTE AS (
    SELECT *,
        ROW_NUMBER() OVER(PARTITION BY transaction_id ORDER BY transaction_id) AS row_num
    FROM store
)
SELECT *
FROM CTE
WHERE row_num > 1;

WITH CTE AS (
    SELECT *,
        ROW_NUMBER() OVER(PARTITION BY transaction_id ORDER BY transaction_id) AS row_num
    FROM store
)
DELETE
FROM CTE
WHERE row_num > 1;

--- Find Null Values
SELECT *
FROM store
WHERE transaction_id IS NULL 
or customer_id is null

----------------------------------------------------------------------
DECLARE @SQL NVARCHAR(MAX) = '';

SELECT @SQL = STRING_AGG(
    'SELECT ''' + COLUMN_NAME + ''' AS ColumnName, 
    COUNT(*) AS NullCount 
    FROM ' + QUOTENAME(TABLE_SCHEMA) + '.store 
    WHERE ' + QUOTENAME(COLUMN_NAME) + ' IS NULL', 
    ' UNION ALL '
)
WITHIN GROUP (ORDER BY COLUMN_NAME)
FROM INFORMATION_SCHEMA.COLUMNS 
WHERE TABLE_NAME = 'store';

-- Execute the dynamic SQL
EXEC sp_executesql @SQL;

--- Treat null values
SELECT *
FROM store
WHERE transaction_id is null
or customer_id is null
or customer_name is null
or customer_age is null
or gender is null
or product_id is null
or product_name is null
or product_category is null
or quantity is null 
or price is null 
or payment_mode is null
or purchase_date is null
or time_of_purchase is null
or status is null
--- First row has many null, it is outlier so deleting 
delete from store
where transaction_id is null

Select *
from store
where customer_name = 'Ehsaan Ram'

UPDATE store
SET customer_id = 'CUST9494'
WHERE transaction_id = 'TXN977900'

Select *
from store
where customer_name = 'Damini Raju'

UPDATE store
SET customer_id = 'CUST1401'
WHERE transaction_id = 'TXN985663'

SELECT *
from store
where customer_id = 'CUST1003'

UPDATE store
set customer_name = 'Mahika Saini', customer_age='35',gender='Male'
WHERE customer_id = 'CUST1003'

SELECT *
FROM store

SELECT DISTINCT(gender)
from store

UPDATE store
SET gender = 'Female'
WHERE gender IN ('f','Female')

UPDATE store
SET gender = 'Male'
WHERE gender IN ('M','Male')

SELECT DISTINCT(payment_mode)
from store

UPDATE store
SET payment_mode = 'Credit Card'
WHERE payment_mode IN ('CC','Credit Card')



--- Solving Business Insights ---
SELECT *
FROM store

--- Top 5 most selling products by quantity?
----- Helps priortize stock and boost sales through targeted promotions
SELECT TOP 5 product_name, SUM(quantity) as total_qty_sold
from store
WHERE status='delivered'
Group by product_name
Order by total_qty_sold DESC

--- Which products are most frequently cancelled
--- Identify poor performing product to improve quality or remove from store
SELECT TOP 5 product_name, COUNT(*) as total_cancelled
FROM store 
WHERE status = 'cancelled'
Group by product_name
ORDER BY total_cancelled DESC

--- What time of the day has the highest number of purchase?
--- Optimize staffing, promotions, server loads
SELECT 
    Case
        WHEN DATEPART(HOUR,time_of_purchase) BETWEEN 0 AND 5 THEN 'NIGHT'
        WHEN DATEPART(HOUR,time_of_purchase) BETWEEN 6 AND 11 THEN 'MORNING'
        WHEN DATEPART(HOUR,time_of_purchase) BETWEEN 12 AND 17 THEN 'AFTERNOON'
        WHEN DATEPART(HOUR,time_of_purchase) BETWEEN 18 AND 23 THEN 'evening'
    END AS time_of_day,
    COUNT(*) AS total_order
FROM store
GROUP BY 
    Case 
        WHEN DATEPART(HOUR,time_of_purchase) BETWEEN 0 AND 5 THEN 'NIGHT'
        WHEN DATEPART(HOUR,time_of_purchase) BETWEEN 6 AND 11 THEN 'MORNING'
        WHEN DATEPART(HOUR,time_of_purchase) BETWEEN 12 AND 17 THEN 'AFTERNOON'
        WHEN DATEPART(HOUR,time_of_purchase) BETWEEN 18 AND 23 THEN 'evening'
    END
    ORDER BY total_order DESC

--- Who are the top 5 highest spending customers?
--- Personalized offers, loyalty rewards and retention
SELECT TOP 5 customer_name, FORMAT(SUM(price*quantity),'C0','en-IN') as total_spends
FROM store
GROUP BY customer_name
ORDER BY SUM(price*quantity) DESC 

--- Which product category generates the highest revenue

SELECT TOP 1 product_category, FORMAT(SUM(quantity*price),'C0','en-IN') as revenue
from store
GROUP BY product_category
ORDER BY SUM(quantity*price) DESC

--- What is the return / cancellation rate per category
-- Cancelled
SELECT product_category, format(COUNT(CASE WHEN status = 'cancelled' THEN 1 END)*100.0/count(*),'N3')+'%' as cancelled_product
FROM store
GROUP BY product_category
order by cancelled_product DESC 
-- Returned
SELECT product_category, format(COUNT(CASE WHEN status = 'returned' THEN 1 END)*100.0/count(*),'N3')+'%' as returned_product
FROM store
GROUP BY product_category
order by returned_product DESC 

--- Most preferred payment mode
 
SELECT payment_mode, COUNT(*) as total_count
FROM store
GROUP BY payment_mode
ORDER BY total_count DESC

--- How does age group affect purchasing behaviour

---SELECT min(customer_age),max(customer_age)
---from store
SELECT
   Case
        WHEN customer_age BETWEEN 18 AND 25 THEN '18-25'
        WHEN customer_age BETWEEN 26 AND 35 THEN '26-35'
        WHEN customer_age BETWEEN 36 AND 45 THEN '36-50'
       ELSE '51+'
   END AS customer_age,
   FORMAT(SUM(price*quantity),'C0','en-IN') AS total_purchase
FROM store
GROUP BY  
   Case
        WHEN customer_age BETWEEN 18 AND 25 THEN '18-25'
        WHEN customer_age BETWEEN 26 AND 35 THEN '26-35'
        WHEN customer_age BETWEEN 36 AND 45 THEN '36-50'
       ELSE '51+'
   END
ORDER BY SUM(price*quantity) DESC

--- Monthly Sales Trend
--Method 1
SELECT 
    FORMAT(purchase_date,'yyyy-MM') AS Month_Year,
    FORMAT(SUM(price*quantity),'C0','en-IN') as total_sales,
    SUM(quantity) as total_quantity
FROM store
GROUP BY FORMAT(purchase_date,'yyyy-MM')
--Method 2
SELECT 
    YEAR(purchase_date) AS Years,
    MONTH(purchase_date) AS Months,
    FORMAT(SUM(price*quantity),'C0','en-IN') as total_sales,
    SUM(quantity) as total_quantity
FROM store
GROUP BY YEAR(purchase_date), MONTH(purchase_date)
ORDER BY Months

--- Are certain genders buying more product categories
--Method 1
SELECT gender, product_category,
    COUNT(product_category) as total_purchase
FROM store
Group by gender, product_category
ORDER BY gender 
--Method 2
SELECT *
FROM ( 
    SELECT gender, product_category
    FROM store
    ) as source_table 
PIVOT (
    COUNT(gender)
    FOR gender IN ([Male],[Female])
    ) as pivot_table 
ORDER BY product_category
