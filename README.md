# Final-Project-Transforming-and-Analyzing-Data-with-SQL

## Project/Goals

This project involved cleaning and querying an ecommerce database containing raw information from an undisclosed website.

The goal of this project was to successfully download, import, and transform the data to a state having enough structure to perform relational queries in pgAdmin4. Query analysis emphasized visitor location data and revenue information.

Working knowledge of GitHub, Git Bash, and pgAdmin4 was required.     

## Process

1. Clone GitHub assignment repository to local machine
2. Download provided .csv files
3. Retrieve column header values from .csv files using Git Bash
4. Create ecommerce database in pgAdmin4
5. Create tables in pgAdmin4 corresponding to .csv filenames, having column names corresponding to .csv column headers
6. Import all data as VARCHAR datatype
7. Examine data and redefine columns with appropriate datatypes
8. Identify shared column elements between tables
9. Begin creating [database ERD](https://miro.com/app/live-embed/uXjVM1hMeYQ=/?moveToViewport=-1436,211,4468,2018&embedId=281970049829) 
10. Determine primary and foreign keys (where possible)
11. Assess data completeness, validity, and uniqueness
12. Fill in missing data where possible
13. Standardize and reformat data values to have consistent structure
14. Check for duplicates
15. Create clean versions of tables (where appropriate)
16. Develop queries to answer [assignment questions](/starting_with_questions.md)
17. Develop queries to answer [further questions](/starting_with_data.md)
18. Summarize findings

## Results

This type of database provides website traffic info. Information about users (e.g. location, time spent on site, date accessed, means of access, number of pages viewed) and products (product SKUs, item category, quantity sold, unit price) are able to be combined to analyze which users are purchasing which products, as well as overall sales patterns. This type of database is powerful for marketing and advertising.

I used the database to determine which cities and countries generated the most web traffic and the most revenue from the website by combining user location and session data with calculated revenue data from the all_sessions_clean table. By creating a function to standardize product category labels as well as assign them subcategories, I was able to identify which category of products each region ordered from the most. I used date information to investigate whether there was enough data to analyze annual revenue increase or decrease by region. With further exploration of product and sales information, I examined which products are the least popular, the relationship between the time spent on the website and a transaction event, as well as what the most common means of accessing the website is. 


## Challenges 

The first challenge with this project was simply accessing the data. In order to import data into pgAdmin4, the database needed to contain pre-existing tables with matching column names to the column headers in the .csv file. Since not all of the .csv files were able to be opened in Excel, I wanted to figure out a method for extracting just the header names from each .csv file without having to actually open the file. I decided to use the $ head function in git bash to accopmlish this.

The second challenge I had with this project was figuring out how all of the tables fit together (or didn't fit together). There was so much missing data that it was difficult to keep track of which tables had any reliable information at all, and whether that information was consistent across other tables. I spent a large amount of time identifying just how many records were missing from each column, and comparing the values of one column with the values of a similar column in a different table.

The third challenge was in gaining context for what the data meant. I was unfamiliar with several aspect of the tables (e.g. sessionqualitydim, ratio, sentimentscore), and spent some time learning about web services such as Google Analytics to better understand how the data was generated.  

The fourth and final challenge was in deciding what to focus on! There were so many queries I ran and small notes I made that it was difficult to know how much to include here. Narrowing down my queries and findings to something streamlined and coherent was (and still is) a work in progress.

## Future Goals

I would like to restructure the database to make each table simpler and to better distinguish the various facts and dimensions. I started to separate components of the analytics table, but I feel that more work can be done there. Similarly, although I defined a multi-column primary key for all_sessions_clean (fullvisitorid, visitid, productsku), there are still 5 instances where there are non-unique values for that key.

I would also like to consolidate and unify product-related information, namely SKU values and unit price values. Many records do not agree with each other, and creating a product facts table which other tables may refer to would help increase the integrity of this database.

I would like to have better understanding the various date/time columns; in particular, all_sessions_clean.time has an unknown significance to me. 

Further exploration of the website user data would also be interesting to me. Investigating the relationship between channelgrouping and timeonsite would be useful for finding the best way to promote the website. Analyzing how user experiences may be variable across regions by relating bounce rates, page views, and location may help to optimize web delivery.
