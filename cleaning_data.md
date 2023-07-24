What issues will you address by cleaning the data?

1. Review column data types and ensure that they are appropriate for the data values.

2. Trim extra white space in text columns, standardize column names, reformat time data, adjust dollar value records

3. Create new tables from analytics table that contain more specific and simplified information. 
       - Table containing sales information
       - Table containing visitor information
    Row_number column added to preserve data integrity.

4. Check for duplicates.
  



Queries:
Below, provide the SQL queries you used to clean your data.

	CREATE TABLE analytics_clean AS 
     SELECT  (LPAD(fullvisitorid::text, 19, '0')) AS fullvisitorid,
             visitid,
             visitnumber,
             userid,
             "date",
             to_timestamp(visitstarttime) AS visitstarttime,
             to_char((timeonsite || ' second')::interval, 'HH24:MI:SS') AS timeonsite,
             pageviews,
             bounces,
             channelgrouping,
             socialengagementtype,
             units_sold,
             unit_price/1000000 AS unit_price,
             revenue/1000000 AS revenue 
     FROM analytics
     ORDER BY fullvisitorid, visitnumber;

    ALTER TABLE IF EXISTS public.analytics_clean
      ADD COLUMN row_id serial NOT NULL;


    CREATE TABLE analytics_clean_sales_info AS
      SELECT  row_id,
              visitid
              units_sold,
              unit_price,
              revenue
      FROM analytics_clean;
	
	  CREATE TABLE analytics_clean_visitor_info AS
	    SELECT  DISTINCT(fullvisitorid),
              visitid,
              visitnumber,
              userid,
              "date",
              visitstarttime,
              timeonsite,
              pageviews,
              bounces,
              channelgrouping,
              socialengagementtype,
              COUNT(row_id) AS num_interactions
      FROM analytics_clean
      GROUP BY  fullvisitorid, 
                visitid,
                visitnumber,
                userid,
                "date",
                visitstarttime,
                timeonsite,
                pageviews,
                bounces,
                channelgrouping,
                socialengagementtype;


	CREATE TABLE all_sessions_clean AS
     SELECT  (LPAD(fullvisitorid::text, 19, '0')) AS fullvisitorid,
             visitid,
             "date",
             to_char(("time" || 'second')::interval, 'HH24:MI:SS') AS "time",
             to_char((timeonsite || ' second')::interval, 'HH24:MI:SS') AS timeonsite,
             pageviews,
             channelgrouping,
             "type",			
             country,
             city,
             REPLACE(productsku, ' ', '') AS productsku,
             TRIM(regexp_replace(v2productname, '\s+', ' ', 'g')) AS product_name,
             v2productcategory,
             productvariant,
             productquantity,
             productprice/1000000 AS productprice,
             productrevenue/1000000 AS productrevenue,
             productrefundamount,
             currencycode,
             itemquantity,
             itemrevenue,
             transactions,
             transactionid,
             transactionrevenue/1000000 AS transactionrevenue,
             totaltransactionrevenue/1000000 AS totaltransactionrevenue,
             pagetitle,
             searchkeyword,
             pagepathlevel1,
             sessionqualitydim,
             ecommerceaction_type,
             ecommerceaction_step,
             ecommerceaction_option
		FROM all_sessions
	  ORDER BY fullvisitorid, visitid, "time";
	
	ALTER TABLE IF EXISTS public.all_sessions_clean
     ADD PRIMARY KEY (visitid, "time", productsku);


	CREATE TABLE products_clean AS
	  SELECT  REPLACE(sku, ' ', '') AS productsku,
          TRIM(regexp_replace("name", '\s+', ' ', 'g')) AS product_name,
          orderedquantity,
          stocklevel,
          restockingleadtime,
          sentimentscore,
          sentimentmagnitude
    FROM products;


	CREATE TABLE sales_report_clean AS
	  SELECT  productsku,
            total_ordered,
            TRIM(regexp_replace("name", '\s+', ' ', 'g')) AS product_name,
            stocklevel,
            restockingleadtime,
            sentimentscore,
            sentimentmagnitude,
            ratio
    FROM sales_report;


