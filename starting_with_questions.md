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


SQL Queries:



Answer:





**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:



Answer:





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:







