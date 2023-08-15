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
![image](https://github.com/MinhAnh99/SQL/assets/74374068/a5193fd5-1e18-41cb-9114-956f8111b90a)
Query results
![image](https://github.com/MinhAnh99/SQL/assets/74374068/8ec04198-fa1a-4611-85a5-9990f9f3fff0)

Query 03:  Revenue by traffic source by week, by month in June 2017 
SQL code
![image](https://github.com/MinhAnh99/SQL/assets/74374068/daebe920-f1c0-46aa-adc8-e789b5f89cae)
Query results
![image](https://github.com/MinhAnh99/SQL/assets/74374068/f0a1c04a-d6e4-4d92-8c1e-4449dd11aef6)

Query 04: Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017
SQL code

Query results
