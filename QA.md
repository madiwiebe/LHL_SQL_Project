What are your risk areas? Identify and describe them.

The primary risk areas within this database are related to missing or incomplete data. Several cases exist where a table column contains the same type of information as column in a different table, but the records do not match. 

For example:
  - The number of distinct visitid values that exist across analytics and all_sessions is 159538. The number of distinct visitid values that are common to both analytics and all_sessions is 3660.
  - The number of distinct productsku values is different in each table containing productsku records. The most concerning disparity is between sales_by_sku and sales_report, where sales_by_sku contains 8 productsku values that do not exist in sales_report. These same 8 productsku values do not exist in the products table either. The products table contains 1092 distinct records, of which only 720 are referenced by any other table in the database. Only 278 productsku values exist in all tables containing productsku records (all_sessions, products, sales_by_sku, and sales_report). 
  - The unit_price column in the analytics table is not directly associated with a productsku value, or any other product information. The products table does not contain any unit price information. The only table containing product information as well as price information is all_sessions, which has several missing values and several columns with no records at all. Therefore, any analysis of sales is less reliable and likely an incomplete representation. Some unit_price values are zero.
  - There are five rows in the analytics table where the bounces column contains a value, but there are also records in the units_sold or the revenue column. Given that a bounce constitues navigation to a webpage followed by leaving the webpage without any interactions, no transaction information should exist where the bounce column is not NULL.
  - Product names and category names are very inconsistent. In the products table, only the item name is included. In the all_sessions table, the product name often includes the brand name as well (e.g. Google, Android, YouTube, Nest). There are some cases in the all_sessions table where two instances of the same productsku have different v2categoryname values. This may interfere with compiling data on individual product performance.
  - Columns containing units_sold, unit_price, and revenue information in the analytics table do not balance when applying the equation revenue = units_sold x unit_price. The reason for the discrepancy is unknown.
  - The all_sessions table and the analytics table do not have good candidates for primary keys. Both tables contain multiple records for columns that should contain distinct information, such as visitid or fullvisitorid. These tables should likely be broken up into smaller and more simplified tables.

QA Process:
Describe your QA process and include the SQL queries used to execute it.

When importing the raw data, VARCHAR was used as a universal data type for all columns in all tables to preserve the contents of the original records. 

Once the tables were successfully imported into pgAdmin4, I reviewed the contents of each column and determined a suitable data type based on the type of information the column contains (e.g. a column containing 'units_sold' information should contain only integer values). 

Throughout initial data exploration and data cleaning, I checked records for missing values, inconsistent formatting, unreasonable values, and duplicate values. The results of these investigations were considered when creating the "clean" versions of each table (particularly standardizing column values and removing duplicates), and missing/unreasonable values are to be especially considered when interpreting the results of queries combining different tables. Some QA concerns remain unresolved.

- Example 1: productsku investigation
  ```
  	--Investigate number of SKUs present in tables containing SKU values 
		--(checked for duplicates by comparing SELECT DISTINCT() query to general SELECT() query)
	
		SELECT DISTINCT(sku)
		FROM products
		--1092 rows returned (no duplicates)
		
		
		SELECT DISTINCT(productsku)
		FROM sales_by_sku
		--462 rows returned (no duplicates)
		
		
		SELECT DISTINCT(productsku)
		FROM sales_report
		--454 rows returned (no duplicates)
		
		
		SELECT DISTINCT(productsku)
		FROM all_sessions;
		--536 rows returned (but table contains duplicates)
  ```
  
- Example 2: total_ordered investigation
  ```
  --Check for whether productsku and total_ordered records are consistent across sales_by_sku and sales_report tables, ignoring productsku values from sales_by_sku that do not have an associate productsku value in products or in sales_report
	
		SELECT	ss.productsku,
				ss.total_ordered AS total_ordered_sales,
				sr.total_ordered AS total_ordered_report
		FROM sales_by_sku ss
		JOIN sales_report sr
		ON ss.productsku = sr.productsku
		WHERE ss.total_ordered <> sr.total_ordered
		;
		
		--0 rows returned.
  ```
  
- Example 3: analytics time data investigation
  ```
  --Check whether analytics date and visitstarttime records are valid.
	
    SELECT "date",
  		DATE(visitstarttime)
    FROM analytics_clean
    WHERE "date" <> DATE(visitstarttime);
  --0 rows returned.
  ```

- Example 4: Check for invalid records between products.orderedquantity and products.stocklevel (stocklevel should always be higher than orderedquantity)
  ```
  SELECT *
  FROM products_clean
  WHERE stocklevel < orderedquantity;
  --15 rows returned.
  ```

- Example 5: Investigate max and min sentimentscore values. If score represents a fraction or percentile of satisfaction, max should be 1.0 and min should be 0.0.
  ```
  SELECT	MAX(sentimentscore) AS maximum_score,
		MIN(sentimentscore) AS minimum_score
  FROM products_clean;
  --maximum score is 1.0, minimum score is -0.6.
  ```
  

Once cleaned, I evaluated which column (or combination of columns) contained unique information within each table. Where a unique combination relevant to the table existed, I assigned a primary key.

- all_sessions_clean: PK = (visitid, time, productsku)
- analytics_clean: No PK.
- analytics_clean_sales_info: PK = rowid
- analytics_clean_visitor_info: PK = (fullvisitorid, visitid, date)
- product_categories_clean: no PK
- products_clean: PK = productsku
- sales_by_sku: PK = productsku
- sales_report_clean: PK = productsku

