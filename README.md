# SQL - Explore E-Commerce Dataset
Apply Google BigQuery to execute queries to calculate a conversion rate, track visitor checkout progress. Optimize marketing efforts and enhance the user experience, improved online business performance


--Query 01: calculate total visit, pageview, transaction for Jan, Feb and March 2017 order by month

SELECT 
  FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d',date)) as month,
  SUM(totals.visits) as visits,
  SUM(totals.pageviews) as pageviews,
  SUM(totals.transactions) as transaction
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
WHERE _table_suffix between '0101' and '0331'
GROUP BY month 
ORDER BY month

--Query 02: Bounce rate per traffic source in July 2017 (Bounce_rate = num_bounce/total_visit) order by total_visit DESC

SELECT 
  trafficSource.source as source,
  sum(totals.visits) as total_visits,
  sum(totals.bounces) as total_no_of_bounces,
  ROUND(100* (sum(totals.bounces) / sum(totals.visits)),8) as bounce_rate
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*` 
GROUP BY source
ORDER BY total_visits DESC

--Query 3: Revenue by traffic source by week, by month in June 2017

SELECT 
  'Month' as time_type,
  FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d',date)) as time,
  trafficSource.source as source,
  sum(productRevenue) as revenue
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
UNNEST(hits) hits,
UNNEST (hits.product) product
WHERE productRevenue IS NOT NULL
GROUP BY time, source
UNION ALL
SELECT 
 'Week' as time_type,
 FORMAT_DATE('%Y%W',PARSE_DATE('%Y%m%d',date)) as time,
 trafficSource.source as source,
  sum(productRevenue) as revenue
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
UNNEST(hits) hits,
UNNEST (hits.product) product
WHERE productRevenue IS NOT NULL
GROUP BY time, source

--Query 04: Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017.

WITH purchase AS (
    SELECT
      FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d', date)) as month,
      ROUND(sum(totals.pageviews)/count(distinct fullVisitorId),5) as age_pageviews_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
  UNNEST(hits) hits,
  UNNEST (hits.product) product
    WHERE _table_suffix between '0601' and '0731'
    AND totals.transactions >= 1,
    AND product.productRevenue IS NOT NULL
    GROUP BY month ),
  non_purchase as (
    SELECT
      FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d', date)) as month,
      ROUND(sum(totals.pageviews)/count (distinct fullVisitorId),5) as age_pageviews_non_purchase
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST(hits) hits,
    UNNEST (hits.product) product
      WHERE _table_suffix between '0601' and '0731'
      AND totals.transactions IS NULL
      AND product.productRevenue IS NULL
      GROUP BY month )

SELECT
  purchase.month,
  purchase.age_pageviews_purchase,
  non_purchase.age_pageviews_non_purchase
FROM purchase
JOIN non_purchase USING(month)

--Query 05: Average number of transactions per user that made a purchase in July 2017

WITH avg_num_transactions AS (
    SELECT
        FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d',date)) as month,
        fullVisitorId,
        SUM (totals.transactions) AS total_transactions_per_user,
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
    UNNEST(hits) hits,
    UNNEST (hits.product) product
    WHERE productRevenue IS NOT NULL
    GROUP BY fullVisitorId, month
)

SELECT 
    month,
    (SUM (total_transactions_per_user) / COUNT(fullVisitorId) ) AS avg_total_transactions_per_user
FROM avg_num_transactions
GROUP BY month

--Query 06: Average amount of money spent per session. Only include purchaser data in July 2017

WITH avg_amount AS (
  SELECT 
    FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d',date)) as month,
    fullVisitorId,
    SUM(totals.visits) AS total_visit,
    SUM(product.productRevenue) AS total_revenue
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
  UNNEST(hits) hits,
  UNNEST (hits.product) product
    WHERE productRevenue IS NOT NULL
    AND totals.transactions IS NOT NULL
    AND totals.visits > 0
    AND totals.totalTransactionRevenue IS NOT NULL
    GROUP BY month, fullVisitorId
)
SELECT month,
(SUM(total_revenue) / SUM(total_visit) /1000000) as avg_revenue_by_user_per_visit
FROM avg_amount
GROUP BY month

--Query 07: Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017. Output should show product name and the quantity was ordered.

SELECT 
    v2ProductName AS other_purchased_products, 
    SUM(productQuantity) AS quantity
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
  UNNEST(hits) hits,
  UNNEST (hits.product) product
  WHERE v2ProductName != "YouTube Men's Vintage Henley"
  AND productRevenue IS NOT NULL
  AND fullVisitorID IN
  ( 
    SELECT fullVisitorId
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
    UNNEST(hits) hits,
    UNNEST (hits.product) product
    WHERE v2ProductName = "YouTube Men's Vintage Henley"
    AND productRevenue IS NOT NULL
  )
  GROUP BY other_purchased_products
  ORDER BY quantity DESC

--Query 08: Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017. For example, 100% product view then 40% add_to_cart and 10% purchase.

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
      COUNT(product.productSKU) as num_add_to_cart
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
    addtocart.num_add_to_cart,
    purchase.num_purchase,
    ROUND((num_add_to_cart/num_product_view)*100,2) as add_to_cart_rate,
    ROUND((num_purchase/num_product_view)*100,2) as purchase_rate
FROM product_view
JOIN addtocart USING(month)
JOIN purchase USING(month)
ORDER BY month
