**Question 1:** How many products do not have any associated transaction information (i.e. they have never been ordered)? Which categories do they belong to?

SQL Queries:
```
WITH products_not_ordered AS(
	SELECT	DISTINCT(pcc.productsku),
			pcc.category,
			als.productquantity
	FROM product_categories_clean pcc
	LEFT JOIN all_sessions_clean als USING (productsku)
	WHERE als.productquantity IS NULL OR als.productquantity = 0
	)
SELECT	category,
		COUNT(category) AS number_of_products
FROM products_not_ordered
GROUP BY category
ORDER BY number_of_products DESC;
```
Answer: 

There are 578 products that do not have an associated productquantity value in all_sessions_clean.
The top three categories with items that have not been ordered are Apparel (198), Shop by Brand (184), and Office (59).



**Question 2:** What is the average amount of time a visitor spends on the website? What is the average amount of time a visitor spends on the website when they end up ordering a product?

SQL Queries:

```
SELECT AVG(acvi.timeonsite::interval) AS overall_average
FROM analytics_clean_visitor_info acvi

UNION

SELECT AVG(acvi.timeonsite::interval) AS purchasing_average
FROM analytics_clean_visitor_info acvi
JOIN all_sessions_clean als
USING (fullvisitorid, visitid)
WHERE als.productquantity IS NOT NULL;
```

Answer:

The overall average for the amount of time a visitor spends on the website is 00:04:59.87.
The average amount of time spent by visitors who purchase a product is 00:05:41.92.



**Question 3:** How many visitors access the website through each type of channel? What percentage of visitors does each channel represent? 

SQL Queries:
```
WITH cat_num AS(
	SELECT	channelgrouping,
		COUNT(fullvisitorid) AS num_visitors
	FROM analytics_clean
	GROUP BY channelgrouping)

SELECT channelgrouping,
		num_visitors,
		TO_CHAR(num_visitors * 100 / SUM(num_visitors) OVER(), 'fm90D000%') AS percent_users
		FROM cat_num
		ORDER BY num_visitors DESC;
```
Answer:

The means by which the largest percentage of users access the website is through an organic search (48.170%). The least frequent access methods are Display (0.831%) and Other (0.001%).
