Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**

Step 1: Define 'highest level of transaction revenues' as 'largest sum of transaction revenues'. The top three cities and countries will be identified.

Step 2: Calculate transaction revenue by combining quantity data from sales_report_clean and price data from all_sessions_clean. 

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


SQL Queries:



Answer:





**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:



Answer:





**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:



Answer:





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:







