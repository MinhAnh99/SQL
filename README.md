# SQL - Explore E-Commerce Dataset
Apply Google BigQuery to execute queries to calculate a conversion rate, track visitor checkout progress.

I. Introduction
This project contains an eCommerce dataset that I will explore using SQL on Google BigQuery. The dataset is based on the Google Analytics public dataset and contains data from an eCommerce website.

II. Dataset Access
The eCommerce dataset is stored in a public Google BigQuery dataset. To access the dataset, follow these steps:

Log in to your Google Cloud Platform account and create a new project.
Navigate to the BigQuery console and select your newly created project.
In the navigation panel, select "Add Data" and then "Search a project".
Enter the project ID "bigquery-public-data.google_analytics_sample.ga_sessions" and click "Enter".
Click on the "ga_sessions_" table to open it.

III. Exploring the Dataset
In this project, I will write 08 query in Bigquery base on Google Analytics dataset. 

Query 01:  Calculate total visit, pageview, transaction for Jan, Feb and March 2017 order by month.
SQL code
![image](https://github.com/MinhAnh99/SQL/assets/74374068/b488ae99-41ce-4826-8ada-8c28a1c2c056)
Query results
![image](https://github.com/MinhAnh99/SQL/assets/74374068/8d234178-6268-42e6-b199-52b893173d8c)


Query 02: Bounce rate per traffic source in July 2017 (Bounce_rate = num_bounce/total_visit) order by total_visit DESC
SQL code
![image](https://github.com/MinhAnh99/SQL/assets/74374068/f7697179-1c9e-4782-b351-9e0277e04336)

Query results
![image](https://github.com/MinhAnh99/SQL/assets/74374068/8ec04198-fa1a-4611-85a5-9990f9f3fff0)


Query 03:  Revenue by traffic source by week, by month in June 2017 
SQL code
![image](https://github.com/MinhAnh99/SQL/assets/74374068/dca1bebc-ca2d-4f87-be07-af8639402df8)

Query results
![image](https://github.com/MinhAnh99/SQL/assets/74374068/f0a1c04a-d6e4-4d92-8c1e-4449dd11aef6)


Query 04: Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017

SQL code
![image](https://github.com/MinhAnh99/SQL/assets/74374068/35bb3965-f9b4-4463-8d30-4c1b9393f3ec)

Query results
![image](https://github.com/MinhAnh99/SQL/assets/74374068/08b5006f-d783-48d2-a207-8ccf0b133f56)


Query 05: Average number of transactions per user that made a purchase in July 2017

SQL code
![image](https://github.com/MinhAnh99/SQL/assets/74374068/2391d1fd-ca57-4598-a1aa-967b47815494)


Query results
![image](https://github.com/MinhAnh99/SQL/assets/74374068/6f4a19c3-4f1d-48d5-9025-ca2ad72bdb4a)


Query 06: Average amount of money spent per session. Only include purchaser data in July 2017

SQL code
![image](https://github.com/MinhAnh99/SQL/assets/74374068/8449aabe-a9c1-4f47-bfee-eb644fc096f5)


Query results
![image](https://github.com/MinhAnh99/SQL/assets/74374068/593f4890-b7fd-45cd-9c99-4d740faba115)


Query 07: Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017. Output should show product name and the quantity was ordered.

SQL code
![image](https://github.com/MinhAnh99/SQL/assets/74374068/31a7f56a-8267-464f-bd82-d31b342ad98f)

Query results
![image](https://github.com/MinhAnh99/SQL/assets/74374068/77cc25b4-76c9-4c95-8291-051402dfe733)


Query 08: Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017. For example, 100% product view then 40% add_to_cart and 10% purchase.

SQL code

WITH product_view as (
    SELECT
      FORMAT_DATE("%Y%m",PARSE_DATE('%Y%m%d',date)) as month,
      COUNT(product.productSKU) as num_product_view
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST(hits) as hits,
    UNNEST(product) as product
    WHERE _table_suffix between '0101' and '0331'
  AND eCommerceAction.action_type = '2'
    GROUP BY month
), 

  addtocart as (
    SELECT
      FORMAT_DATE("%Y%m",PARSE_DATE('%Y%m%d',date)) as month,
      COUNT(product.productSKU) as num_addtocart
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST(hits) as hits,
    UNNEST(product) as product
    WHERE _table_suffix between '0101' and '0331'
    AND eCommerceAction.action_type = '3'
    GROUP BY month
),

  purchase as (
    SELECT
      FORMAT_DATE("%Y%m",PARSE_DATE('%Y%m%d',date)) as month,
      COUNT(product.productSKU) as num_purchase
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST(hits) as hits,
    UNNEST(product) as product
    WHERE _table_suffix between '0101' and '0331'
    AND eCommerceAction.action_type = '6'
    AND product.productRevenue IS NOT NULL
    GROUP BY month
)

SELECT
    product_view.month,
    product_view.num_product_view,
    addtocart.num_addtocart,
    purchase.num_purchase,
    ROUND((num_addtocart/num_product_view)*100,2) as add_to_cart_rate,
    ROUND((num_purchase/num_product_view)*100,2) as purchase_rate
FROM product_view
JOIN addtocart USING(month)
JOIN purchase USING(month)
ORDER BY month

Query results
![image](https://github.com/MinhAnh99/SQL/assets/74374068/0829e411-7fc4-4b6c-bd8b-e8c50d884230)

