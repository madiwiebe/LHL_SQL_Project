Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**

Step 1: Define 'highest level of transaction revenues' as 'largest sum of transaction revenues'. The top three cities and countries will be identified.

Step 2: Calculate transaction revenue by combining quantity data and price data from all_sessions_clean. Assume that if no quantity information is included in the row, no purchase was made in that session.  

SQL Queries:

Country:

 	WITH	calc_revenue AS(
			SELECT	visitid,
				"time",
				productsku,
				productquantity,
				productprice,
				((CASE	WHEN productquantity IS NULL THEN 0
			  		WHEN productquantity IS NOT NULL THEN productquantity
			  	  END) * productprice) AS prod_revenue
			FROM all_sessions_clean
			)
	
	SELECT	als.country,
		SUM(cr.prod_revenue) AS total_transaction_revenue
	FROM all_sessions_clean als
	JOIN calc_revenue cr USING(visitid, "time", productsku)
	GROUP BY als.country
	ORDER BY total_transaction_revenue DESC
	LIMIT 3;
 City:

	WITH	calc_revenue AS(
			SELECT	visitid,
				"time",
				productsku,
				productquantity,
				productprice,
				((CASE WHEN productquantity IS NULL THEN 0
					WHEN productquantity IS NOT NULL THEN productquantity
				  END) * productprice) AS prod_revenue
			FROM all_sessions_clean
			)
	
	SELECT	als.city,
		SUM(cr.prod_revenue) AS total_transaction_revenue
	FROM all_sessions_clean als
	JOIN calc_revenue cr USING(visitid, "time", productsku)
	GROUP BY city
	HAVING als.city <> 'not available in demo dataset'
	ORDER BY total_transaction_revenue DESC
	LIMIT 3;
    
Answer:

The three countries with the highest total transaction revenue are:
1. United States	(5,435.73 USD)
2. Ireland     		(99.99 USD)
3. Argentina		(99.99 USD)

The three cities with the highest total transaction revenue are:
1. Mountain View 	(785.97 USD)
2. Salem 		(639.36 USD)
3. New York 		(590.56 USD)
   

    
**Question 2: What is the average number of products ordered from visitors in each city and country?**

Step 1: Define "average number of products ordered from visitors in each city and country" as "total number of units sold divided by total number of unique visitids associated with a city or country".  Only visitids that are associated with a units_sold record will be used when calculating the average.

Step 2: Determine the total number of units sold per unique visit (total_units_sold CTE).

Step 3: Determine which visitids in all_sessions_clean correspond with visitids associated with a sale in analytics_clean_sales_info (shared_visitids CTE).

Step 4: Calculate the total number of visitids associated with a sale for each country (num_visits_per_country CTE).

Step 5: Calculate the average number of units purchased per sale for each country.

Step 6: Repeat steps 4 and 5 for city records.


SQL Queries:

Country:

	WITH total_units_sold AS(
		SELECT	visitid,
			SUM(units_sold) AS visit_num_units
		FROM analytics_clean_sales_info
		WHERE units_sold IS NOT NULL
		GROUP BY visitid
		)
		,
	
	    shared_visitids AS(
		SELECT	DISTINCT(als.visitid) AS visitid,
			als.country
		FROM 	all_sessions_clean als
		JOIN analytics_clean_sales_info acsi
		ON als.visitid = acsi.visitid
		)
		,
	
	    num_visits_per_country AS(
		SELECT	visitid,
			country,
			COUNT(*) OVER(PARTITION BY country) AS num_visits
		FROM shared_visitids
		WHERE country NOT LIKE '%not set%'
		GROUP BY country, visitid
		)

	SELECT	nvpc.country,
		SUM(tus.visit_num_units)/AVG(nvpc.num_visits) AS avg_num_units
	FROM num_visits_per_country nvpc
	JOIN total_units_sold tus
	ON nvpc.visitid = tus.visitid
	GROUP BY nvpc.country
	ORDER BY avg_num_units DESC
 	;

City:

	WITH total_units_sold AS(
		SELECT	visitid,
			SUM(units_sold) AS visit_num_units
		FROM analytics_clean_sales_info
		WHERE units_sold IS NOT NULL
		GROUP BY visitid
		)
		,
	
	    shared_visitids AS(
		SELECT	DISTINCT(als.visitid) AS visitid,
			als.city
		FROM 	all_sessions_clean als
		JOIN analytics_clean_sales_info acsi
		ON als.visitid = acsi.visitid
		WHERE	als.city <> 'not available in demo dataset' 
  			AND (als.city NOT LIKE '%not set%') 
     			AND als.city IS NOT NULL
		)
		,
	
	    num_visits_per_city AS(
		SELECT	visitid,
			city,
			COUNT(*) OVER(PARTITION BY city) AS num_visits
		FROM shared_visitids
		GROUP BY city, visitid
		)

	SELECT	nvpc.city,
		SUM(tus.visit_num_units)/AVG(nvpc.num_visits) AS avg_num_units
	FROM num_visits_per_city nvpc
	JOIN total_units_sold tus
	ON nvpc.visitid = tus.visitid
	GROUP BY nvpc.city
	ORDER BY avg_num_units DESC
	;


Answer:

The largest average number of products ordered from a country is associated with the Maldives (2.00 units per order).

The largest average number of products ordered from a city is associated with Detroit (3.00 units per order).



**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**

Step 1: Use standardized category name CTE developed during data cleaning to create more uniform category and subcategory names for products.

Step 2: Identify visitids associated with orders (shared_visitid CTE).

Step 3: Group the relevant visitids by country.

Step 4: Identify productsku records associated with the relevant visitids.

Step 5: Identify the category associated with those productskus

Step 6: Count how many instances of each category are present in orders for each country.

Step 7: Repeat steps 3-6 for city records.

Step 8: Explore data further.


SQL Queries:

Country:

	WITH 
	cleaned_categories AS(
		SELECT	DISTINCT(productsku),
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
	,
	shared_visitids AS(
		SELECT	DISTINCT(als.visitid) AS visitid,
				als.country
		FROM all_sessions_clean als
		JOIN analytics_clean_sales_info acsi
		ON als.visitid = acsi.visitid
		)
	,
	visit_products AS(
		SELECT	als.visitid,
				als.productsku
		FROM shared_visitids sv
		JOIN all_sessions_clean als
		ON sv.visitid = als.visitid
		)
	,
	orders_by_country AS(
		SELECT	sv.visitid,
			sv.country,
			vp.productsku,
			cc.category,
			COUNT(*) OVER(PARTITION BY country, category) AS num_products_ordered
		FROM shared_visitids sv
		JOIN visit_products vp ON sv.visitid = vp.visitid
		JOIN cleaned_categories cc ON vp.productsku = cc.productsku
		WHERE country NOT LIKE '%not set%'	
		)

	SELECT	DISTINCT(obc.country),
		obc.category,
		obc.num_products_ordered
  	INTO TEMPORARY TABLE order_categories_by_country
	FROM orders_by_country obc
	ORDER BY country, num_products_ordered, category;

City:

	WITH 
	cleaned_categories AS(
		SELECT	DISTINCT(productsku),
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
	,
	shared_visitids AS(
		SELECT	DISTINCT(als.visitid) AS visitid,
				als.city
		FROM all_sessions_clean als
		JOIN analytics_clean_sales_info acsi
		ON als.visitid = acsi.visitid
		)
	,
	visit_products AS(
		SELECT	als.visitid,
			als.productsku
		FROM shared_visitids sv
		JOIN all_sessions_clean als
		ON sv.visitid = als.visitid
		)
	,
	orders_by_city AS(
		SELECT	sv.visitid,
			sv.city,
			vp.productsku,
			cc.category,
			COUNT(*) OVER(PARTITION BY city, category) AS num_products_ordered
		FROM shared_visitids sv
		JOIN visit_products vp ON sv.visitid = vp.visitid
		JOIN cleaned_categories cc ON vp.productsku = cc.productsku
		WHERE city <> 'not available in demo dataset' AND (city NOT LIKE '%not set%') AND city IS NOT NULL	
		)

	SELECT	DISTINCT(obc.city),
		obc.category,
		obc.num_products_ordered
	INTO TEMPORARY TABLE order_categories_by_city
	FROM orders_by_city obc
	ORDER BY city, num_products_ordered, category;



Further exploration (countries):

	--number of countries that have ordered from a given category
	SELECT	DISTINCT(category),
	COUNT(*) OVER(PARTITION BY category) AS num_countries
	FROM order_categories_by_country
	ORDER BY num_countries DESC;

	--number of distinct categories
	SELECT DISTINCT(category)
	FROM order_categories_by_country;

	--number of unique categories that each country has ordered products from
	SELECT	DISTINCT(country),
		COUNT(*) OVER(PARTITION BY country) AS num_categories
	FROM order_categories_by_country
	ORDER BY num_categories DESC;

	--total number of units sold from within each category 
	SELECT	DISTINCT(category),
		SUM(num_products_ordered) OVER (PARTITION BY category) AS units_sold
	FROM order_categories_by_country
	GROUP BY category, num_products_ordered
	ORDER BY units_sold DESC;

Futher exploration (cities): 

	--number of cities that have ordered from a given category
 	SELECT	DISTINCT(category),
		COUNT(*) OVER(PARTITION BY category) AS num_cities
	FROM order_categories_by_city
	ORDER BY num_cities DESC;

	--number of distinct categories
	SELECT DISTINCT(category)
	FROM order_categories_by_city;

	--number of unique categories that each city has ordered products from
	SELECT	DISTINCT(city),
		COUNT(*) OVER(PARTITION BY city) AS num_categories
	FROM order_categories_by_city
	ORDER BY num_categories DESC;

	--total number of units sold from within each category 
	SELECT	DISTINCT(category),
		SUM(num_products_ordered) OVER (PARTITION BY category) AS units_sold
	FROM order_categories_by_city
	GROUP BY category, num_products_ordered
	ORDER BY units_sold DESC;


Answer:

Of the 13 distinct categories, the category that is most commonly ordered from is Shop by Brand (81 countries, 102 cities), closely followed by Apparel (76 countries, 92 cities). The United States has ordered products from all 13 categories, while the highest number of categories a single city has ordered from is 12 (a three-way tie between New York, Mountain View, and Sunnyvale). The category having the most overall units sold is Apparel (6023 units recorded for sales by country, 2067 units recorded for sales by city).



**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:



Answer:





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:







