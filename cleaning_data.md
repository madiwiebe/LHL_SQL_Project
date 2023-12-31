What issues will you address by cleaning the data?

1. Review column data types and ensure that they are appropriate for the data values.

2. Trim extra white space in text columns, standardize column names, reformat time data, adjust dollar value records

3. Create new tables from analytics table that contain more specific and simplified information. 
       - Table containing sales information
       - Table containing visitor information
    Row_number column added to preserve data integrity across analytics_clean, analytics_clean_sales_info, and analytics_clean_visitor_info.

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


    WITH cleaned_categories AS(
	SELECT	productsku,
			product_name,
			v2productcategory,
			CASE
				WHEN product_name LIKE '%Bottle%' THEN 'Drinkware'
				WHEN product_name LIKE '%Mug%' THEN 'Drinkware'
				WHEN product_name LIKE '%Tumbler%' THEN 'Drinkware'
				WHEN product_name LIKE '%Women%' THEN 'Apparel'
				WHEN product_name LIKE '%Men%' THEN 'Apparel'
				WHEN product_name LIKE '%Kid%' THEN 'Apparel'
				WHEN product_name LIKE '%Bag%' THEN 'Bags'
				WHEN product_name LIKE '%Backpack%' THEN 'Bags'
				WHEN product_name LIKE '%Rucksack%' THEN 'Bags'
				WHEN product_name LIKE '%Pen%' THEN 'Office'
				WHEN product_name LIKE '%Journal%' THEN 'Office'
				WHEN product_name LIKE '%Gift%Card%' THEN 'Gift Cards'
				WHEN product_name LIKE '%Android%' THEN 'Shop by Brand' 
				WHEN product_name LIKE '%You%Tube%' THEN 'Shop by Brand' 
				WHEN product_name LIKE '%Nest%' THEN 'Shop by Brand' 
				WHEN product_name LIKE '%Waze%' THEN 'Shop by Brand'
				WHEN product_name LIKE '%Google%' THEN 'Shop by Brand'
				WHEN product_name LIKE '%Flashlight%' THEN 'Electronics'
				WHEN product_name LIKE '%Charger%' THEN 'Electronics'
				WHEN product_name LIKE '%Yoga%' THEN 'Accessories'
				ELSE CASE
					WHEN v2productcategory LIKE '%Apparel%' THEN 'Apparel'
					WHEN v2productcategory LIKE '%Electronics%' THEN 'Electronics'
					WHEN v2productcategory LIKE '%Housewares%' THEN 'Housewares'
					WHEN v2productcategory LIKE '%Drinkware%' THEN 'Drinkware'
					WHEN v2productcategory LIKE '%Accessories%' THEN 'Accessories'
					WHEN v2productcategory LIKE '%Bags%' THEN 'Bags'
					WHEN v2productcategory LIKE '%Office%' THEN 'Office'
					WHEN v2productcategory LIKE '%Lifestyle%' THEN 'Lifestyle'
					WHEN v2productcategory LIKE '%Sale%' THEN 'Sale'
					WHEN v2productcategory LIKE '%Headgear%' THEN 'Apparel'
					WHEN v2productcategory LIKE '%Wearables%' THEN 'Apparel'
					WHEN v2productcategory LIKE '%Limited Supply%' THEN 'Limited Supply'
					WHEN v2productcategory LIKE '%Gift Cards%' THEN 'Gift Cards'
					WHEN v2productcategory LIKE '%Fun%' THEN 'Lifestyle'
					WHEN v2productcategory LIKE '%Bottle%' THEN 'Drinkware'
					WHEN v2productcategory LIKE '%Kids%' THEN 'Apparel'
					WHEN v2productcategory LIKE '%Android%' THEN 'Shop by Brand' 
					WHEN v2productcategory LIKE '%You%Tube%' THEN 'Shop by Brand' 
					WHEN v2productcategory LIKE '%Nest%' THEN 'Shop by Brand' 
					WHEN v2productcategory LIKE '%Waze%' THEN 'Shop by Brand'
					WHEN v2productcategory LIKE '%Google%' THEN 'Shop by Brand'
					WHEN v2productcategory LIKE '%Brand%' THEN 'Shop by Brand'
					WHEN v2productcategory LIKE '%Fruit Games%' THEN 'Shop by Brand'
					ELSE 'Miscellaneous'
					END
			END category
			,
			CASE 
				WHEN product_name LIKE '%Bottle%' THEN 'Water Bottles & Tumblers'
				WHEN product_name LIKE '%Mug%' THEN 'Mugs & Cups'
				WHEN product_name LIKE '%Tumbler%' THEN 'Water Bottles & Tumblers'
				WHEN product_name LIKE '%Women%' THEN 'Women''s'
				WHEN product_name LIKE '%Men%' THEN 'Men''s'
				WHEN product_name LIKE '%Kid%' THEN 'Kid''s'
				WHEN product_name LIKE '%Pen%' THEN 'Writing Instruments'
				WHEN product_name LIKE '%Note%' THEN 'Notebooks & Journals'
				WHEN product_name LIKE '%Journal%' THEN 'Notebooks & Journals'
				WHEN product_name LIKE '%Tote%' THEN 'Shopping & Totes'
				WHEN product_name LIKE '%Shopping%' THEN 'Shopping & Totes'
				WHEN product_name LIKE '%Backpack%' THEN 'Backpacks'
				WHEN product_name LIKE '%Rucksack%' THEN 'Backpacks'
				WHEN product_name LIKE '%Flashlight%' THEN 'Flashlights'
				WHEN product_name LIKE '%Charger%' THEN 'Power'
				WHEN product_name LIKE '%Yoga%' THEN 'Sports & Fitness'
				WHEN product_name LIKE '%Android%' THEN 'Android' 
				WHEN product_name LIKE '%You%Tube%' THEN 'YouTube' 
				WHEN product_name LIKE '%Nest%' THEN 'Nest' 
				WHEN product_name LIKE '%Waze%' THEN 'Waze'
				WHEN product_name LIKE '%Google%' THEN 'Google'
				ELSE CASE 
					WHEN v2productcategory LIKE '%Kid%Infant%' THEN 'Kid''s-Infant'
					WHEN v2productcategory LIKE '%Kid%Toddler%' THEN 'Kids''s-Toddler'
					WHEN v2productcategory LIKE '%Kid%Youth%' THEN 'Kids''s-Youth'
					WHEN v2productcategory LIKE '%Kid%' THEN 'Kid''s'
					WHEN v2productcategory LIKE '%Women%' THEN 'Women''s'
					WHEN v2productcategory LIKE '%Men%' THEN 'Men''s'
					WHEN v2productcategory LIKE '%Headgear%' THEN 'Headgear'
					WHEN v2productcategory LIKE '%Audio%' THEN 'Audio'
					WHEN v2productcategory LIKE '%Power%' THEN 'Power'
					WHEN v2productcategory LIKE '%Flashlights%' THEN 'Flashlights'
					WHEN v2productcategory LIKE '%Mugs%' THEN 'Mugs & Cups'
					WHEN v2productcategory LIKE '%Bottle%' THEN 'Water Bottles & Tumblers'
					WHEN v2productcategory LIKE '%Sport%' THEN 'Sports & Fitness'
					WHEN v2productcategory LIKE '%Fun%' THEN 'Fun'
					WHEN v2productcategory LIKE '%Shopping%' THEN 'Shopping & Totes'
					WHEN v2productcategory LIKE '%Tote%' THEN 'Shopping & Totes'
					WHEN v2productcategory LIKE '%Backpack%' THEN 'Backpacks'
					WHEN v2productcategory LIKE '%Writing%' THEN 'Writing Instruments'
					WHEN v2productcategory LIKE '%Note%' THEN 'Notebooks & Journals'
					WHEN v2productcategory LIKE '%Android%' THEN 'Android' 
					WHEN v2productcategory LIKE '%YouTube%' THEN 'YouTube' 
					WHEN v2productcategory LIKE '%Nest%' THEN 'Nest' 
					WHEN v2productcategory LIKE '%Waze%' THEN 'Waze'
					WHEN v2productcategory LIKE '%Google%' THEN 'Google'
					WHEN v2productcategory LIKE '%Fruit Games%' THEN 'Fruit Games'
					ELSE 'N/A'
					END
				END subcategory 				
	FROM all_sessions_clean
	ORDER BY category, subcategory
	)
	SELECT DISTINCT(productsku),
			product_name,
			v2productcategory,
			category,
			subcategory
	INTO TABLE product_categories_clean
	FROM cleaned_categories
	;


Generic structure of queries used to check for duplicates in tables/columns:

--check for total number of rows in the table.

	SELECT COUNT(*)
	FROM table.name

--check for duplicates across multiple columns within a table.

	SELECT	column_1,
		column_2,
 		column_3,
  		COUNT(*)
   	FROM table.name
    	GROUP BY column_1, column_2, column_3
	HAVING COUNT(*) > 1

--check for number of distinct values within a single column. Compare this value with total number of rows in the table.

	SELECT COUNT(DISTINCT(column_name)
	FROM table.name

 
