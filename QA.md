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

Throughout initial data exploration and data cleaning, I checked records for missing values, inconsistent formatting, unreasonable values, and duplicate values. The results of these investigations were considered when creating the "clean" versions of each table (particularly standardizing column values and removing duplicates), and missing/unreasonable values are to be especially considered when interpreting the results of queries combining different tables.

Once cleaned, I evaluated which column (or combination of columns) contained unique information within each table. Where a unique combination relevant to the table existed, I assigned a primary key.



