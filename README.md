# E-commerce business – Interpret dataset in different views – SQL BigQuery
![image](https://github.com/user-attachments/assets/a6d436f2-9797-4948-97a8-c82a7b945e22)


a publicly available sample dataset in Google BigQuery

# 📌 Overview

This project analyzes Google Analytics data using SQL queries in Google BigQuery. It explores e-commerce performance metrics such as visits, pageviews, transactions, bounce rates, revenue, and user behavior from the bigquery-public-data.google_analytics_sample dataset. The queries focus on data from 2017, providing insights into monthly trends, traffic sources, product performance, and conversion rates.

# 📂 Dataset Description & Data Structure

## 📌 Data Source

The bigquery-public-data.google_analytics_sample.ga_sessions_2017* dataset is about web and e-commerce analytics for a sample online store in 2017. It captures:

* Who: Anonymized users visiting the site.
* What: Their interactions (views, clicks, purchases) and purchases (products, revenue).
* When: Daily sessions throughout 2017.
* How: Traffic sources driving them to the site.
It’s a rich dataset for practicing SQL and deriving insights into online user behavior and sales performance

* Size: the dataset for 2017 has more than 467K rows
* Format: .CSV

## 📊 Data structure

| Field Name                         | Data Type |
|------------------------------------|-----------|
| `fullVisitorId`                    | STRING    |
| `date`                             | STRING    |
| `totals`                           | RECORD    |
| `totals.bounces`                   | INTEGER   |
| `totals.hits`                      | INTEGER   |
| `totals.pageviews`                 | INTEGER   |
| `totals.visits`                    | INTEGER   |
| `totals.transactions`              | INTEGER   |
| `trafficSource.source`             | STRING    |
| `hits`                             | RECORD    |
| `hits.eCommerceAction`             | RECORD    |
| `hits.eCommerceAction.action_type` | STRING    |
| `hits.product`                     | RECORD    |
| `hits.product.productQuantity`     | INTEGER   |
| `hits.product.productRevenue`      | INTEGER   |
| `hits.product.productSKU`          | STRING    |
| `hits.product.v2ProductName`       | STRING    |

# ⚒️  Project Structure

There were 8 queries covered several points of view of this E-commerce website from general to details

* 👉🏻 Query 01: calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)

    * The goal was to summarize website activity (visits, pageviews, transactions) by month for Q1 2017. This required aggregating data from a time-partitioned dataset and formatting the output.
      
```sql
  SELECT 
  distinct format_date('%Y%m',parse_date('%Y%m%d',date)) as month
  ,count(visitId) as vistis
  ,sum(totals.pageviews) as pageviews
  ,sum(totals.transactions) as transactions
 FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*` 
 where _table_suffix between '0101' and '0331'
 group by 1
 order by 1
````
![image](https://github.com/user-attachments/assets/9e472360-2c56-4158-b372-075db82203ba)

![image](https://github.com/user-attachments/assets/18bc7906-18f9-4649-bc64-10c44f976d02)

=> Visits show session counts; growth may indicate rising traffic.

=> Pageviews reflect engagement; high ratios suggest sticky content.
   
=> Transactions track purchases; peaks might tie to January sales.
      
=> Data covers Q1 2017 only, filtered by table suffix.

* 👉🏻 Query 02: Bounce rate per traffic source in July 2017 (Bounce_rate = num_bounce/total_visit) (order by total_visit DESC)

     * This query is executed to evaluate the effectiveness of different traffic sources driving visitors to the website in July 2017. By calculating the bounce rate (bounces divided by visits) for each source, it helps assess user engagement and traffic quality, focusing on the top 4 sources by visit volume.
       
```sql
SELECT  
  trafficSource.source as source
  ,sum(totals.visits) as total_visits
  ,sum(totals.bounces) as total_no_of_bounces
  ,round(100.00 * sum(totals.bounces) / sum(totals.visits),3) as bounce_rate
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*` 
group by 1
order by 2 desc
limit 4
```

![image](https://github.com/user-attachments/assets/e7c818a9-8f80-4a46-bbb0-0145fa073f05)

* 👉🏻 Query 3: Revenue by traffic source by week, by month in June 2017
```sql
SELECT  
  'Month' as time_type
  ,format_date('%Y%m', parse_date('%Y%m%d',date)) as time
  ,trafficSource.source as source
  ,round(sum(productRevenue) / 1000000,4) as revenue
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*` 
,unnest (hits) as hits
,unnest(hits.product) as product
where productRevenue is not null
group by 3,2
union all
SELECT  
  'Week' as time_type
  ,format_date('%Y%W', parse_date('%Y%m%d',date)) as time
  ,trafficSource.source as source
  ,round(sum(productRevenue) / 1000000,4) as revenue
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*` 
,unnest (hits) as hits
,unnest(hits.product) as product
where productRevenue is not null
group by 3,2
order by 4 desc
limit 4
```

![image](https://github.com/user-attachments/assets/c98cb4f9-8d23-4aa9-a362-8ba105b9189a)

**=> Insights Sought:**

Traffic Quality: High bounce rates show low engagement; low rates indicate quality traffic.

Source Performance: Reveals which top sources (e.g., Google, direct) keep users engaged.

Resource Allocation: Highlights where to optimize or invest marketing efforts.

Behavior Trends: Spots July 2017 patterns for seasonal or campaign insights.

* 👉🏻 Query 04: Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017.
```sql
with t1 as(
SELECT  
  format_date('%Y%m', parse_date('%Y%m%d',date)) as month
  ,sum(totals.pageviews)/count(distinct fullVisitorId) as avg_pageviews_purchase
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
,unnest(hits) as hits
,unnest(hits.product) as product
where (_table_suffix between '0601' and '0731')
and totals.transactions >=1 and productRevenue is not null
group by 1)
,t2 as(
SELECT  
  format_date('%Y%m', parse_date('%Y%m%d',date)) as month
  ,sum(totals.pageviews)/count(distinct fullVisitorId) as avg_pageviews_non_purchase
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
,unnest(hits) as hits
,unnest(hits.product) as product
where (_table_suffix between '0601' and '0731')
and totals.transactions is null and productRevenue is  null
group by 1)
select
 t1.month
 ,avg_pageviews_purchase
 ,avg_pageviews_non_purchase
 from t1
 left join t2
 on t1.month = t2.month
 order by 1
```

![image](https://github.com/user-attachments/assets/10d021c9-e7a1-4731-87c2-c8c24e7a8ffc)

* 👉🏻 Query 05: Average number of transactions per user that made a purchase in July 2017
```sql
SELECT 
  format_date('%Y%m', parse_date('%Y%m%d', date)) as Month
  ,sum(totals.transactions) / count(distinct fullVisitorId) as Avg_total_transactions_per_user
 FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*` 
 ,unnest(hits) as hits
 ,unnest(hits.product) as product
 where totals.transactions >=1 and productRevenue is not null
 group by 1
```

![image](https://github.com/user-attachments/assets/df780825-828d-46ad-b6a8-478e4e95605d)

* 👉🏻 Query 06: Average amount of money spent per session. Only include purchaser data in July 2017
```sql
SELECT 
  format_date('%Y%m', parse_date('%Y%m%d', date)) as Month
  ,(sum(productRevenue)/sum(totals.visits))/1000000 as avg_revenue_by_user_per_visit
 FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*` 
 ,unnest(hits) as hits
 ,unnest(hits.product) as product
 where totals.transactions is not null and productRevenue is not null
 group by 1
```

![image](https://github.com/user-attachments/assets/1eea5e29-c7fe-498c-8cfc-1ba2f7355709)

* 👉🏻 Query 07: Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017. Output should show product name and the quantity was ordered.
```sql
SELECT 
   v2ProductName as other_purchased_products
  ,sum(productQuantity) as quantity
 FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
 ,unnest(hits) as hits
 ,unnest(hits.product) as product
 where fullVisitorId in (SELECT 
 fullVisitorId field
 FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
 ,unnest(hits) as hits
 ,unnest(hits.product) as product
 where v2ProductName = "YouTube Men's Vintage Henley")
 and productRevenue is not null
 and v2ProductName <> "YouTube Men's Vintage Henley"
 and totals.transactions IS NOT NULL
 group by 1
 order by 1, 2 desc
```

![image](https://github.com/user-attachments/assets/e611685c-6989-4ae0-8473-c2489dbc5911)

* 👉🏻 Query 08: Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017. For example, 100% product view then 40% add_to_cart and 10% purchase.
Add_to_cart_rate = number product  add to cart/number product view. Purchase_rate = number product purchase/number product view. The output should be calculated in product level.
```sql
with t1 as(
SELECT 
  format_date('%Y%m', parse_date('%Y%m%d', date)) as month
  ,count(hits.eCommerceAction.action_type) as num_product_view
 FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
 ,unnest(hits) as hits
where _table_suffix between '0101' and '0331'
  and hits.eCommerceAction.action_type  = '2'
 group by 1)
 ,t2 as(SELECT 
  format_date('%Y%m', parse_date('%Y%m%d', date)) as month
  ,count(hits.eCommerceAction.action_type) as num_addtocart
 FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
 ,unnest(hits) as hits
 where _table_suffix between '0101' and '0331'
  and hits.eCommerceAction.action_type  = '3'
 group by 1)
 ,t3 as (SELECT 
  format_date('%Y%m', parse_date('%Y%m%d', date)) as month
  ,count(hits.eCommerceAction.action_type) as num_purchase
 FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
  ,unnest(hits) as hits
  ,unnest(hits.product) as product
 where _table_suffix between '0101' and '0331'
  and hits.eCommerceAction.action_type  = '6'
  and product.productRevenue is not null
 group by 1)
 select
  t1.month
  ,num_product_view
  ,num_addtocart
  ,num_purchase
  ,round(100.00 * num_addtocart / num_product_view,2) as add_to_cart_rate
  ,round(100.00 * num_purchase / num_product_view,2) as purchase_rate
from t1
join t2 on t1.month = t2.month
join t3 on t2.month = t3.month
group by 1,2,3,4
order by 1
```

![image](https://github.com/user-attachments/assets/19c69a6a-0264-4c8d-8c3a-e8c43db70f05)

# 🔎 Conclusions

  - ➡️ Identified traffic trends, peak periods, and key user behavior metrics (pageviews, bounce rates, transactions).
  - ➡️ Analyzed top-performing products and traffic sources, offering insights into marketing effectiveness and conversion rates.

- **Contributions:**
  - Demonstrated SQL query applications to Google Analytics data in BigQuery for business insights.
  - Provided actionable recommendations for optimizing e-commerce strategies and marketing efforts.

- **Impact:**
  - Showcased the value of data analysis in improving website performance and driving data-driven decisions.


# 🤩 SQL Skills Learned from This Project 👍

* Working with Nested and Repeated Fields: how to access data within nested structures, transforming arrays into rows for analysis
* Using Common Table Expressions (CTEs): Breaking a problem into smaller, reusable pieces, Simplifying Logic, avoiding repetitive subqueries, improving readability.
* Aggregation and Grouping: Aggregating data over time (e.g., monthly), ordering results for clarity.
* Debugging and Problem-Solving: Comparing query logic to expected outcomes, identifying discrepancies, and iterating (a critical real-world skill).
