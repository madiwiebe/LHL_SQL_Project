Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**

Step 1: Define 'highest level of transaction revenues' as 'largest sum of transaction revenues'. The top three cities and countries will be identified.

Step 2: Calculate transaction revenue by combining quantity data from sales_report_clean and price data from all_sessions_clean. 

SQL Queries:

    SELECT	als.country,
		SUM(sr.total_ordered*als.productprice) AS prod_revenue
    FROM all_sessions_clean als
    JOIN sales_report_clean sr ON als.productsku = sr.productsku
    GROUP BY country
    ORDER BY prod_revenue DESC
    LIMIT 3;

    SELECT	als.city,
			SUM(sr.total_ordered*als.productprice) AS prod_revenue
	FROM all_sessions_clean als
	JOIN sales_report_clean sr ON als.productsku = sr.productsku
	GROUP BY city
	HAVING als.city <> 'not available in demo dataset'
	ORDER BY prod_revenue DESC
    LIMIT 3;
    
Answer:

The three countries with the highest total transaction revenue are:
1. United States      (5,367,386.59 USD)
2. United Kingdom     (257,003.62 USD)
3. Canada             (171,816.09 USD)

The three cities with the highest total transaction revenue are:
1. Mountain View (1,251,680.65 USD)
2. San Francisco (378,883.06 USD)
3. Sunnyvale (351,486.09 USD)
   

    
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







