# SQL-Google-Analytics-Data
a publicly available sample dataset in Google BigQuery
# Overview
This project analyzes Google Analytics data using SQL queries in Google BigQuery. It explores e-commerce performance metrics such as visits, pageviews, transactions, bounce rates, revenue, and user behavior from the bigquery-public-data.google_analytics_sample dataset. The queries focus on data from 2017, providing insights into monthly trends, traffic sources, product performance, and conversion rates.

The bigquery-public-data.google_analytics_sample.ga_sessions_2017* dataset is about web and e-commerce analytics for a sample online store in 2017. It captures:

* Who: Anonymized users visiting the site.
* What: Their interactions (views, clicks, purchases) and purchases (products, revenue).
* When: Daily sessions throughout 2017.
* How: Traffic sources driving them to the site.
It’s a rich dataset for practicing SQL and deriving insights into online user behavior and sales performance
# Project Structure

There were 8 queries covered several points of view of this E-commerce website from general to details
** Query 01: calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)

```SELECT 
  distinct format_date('%Y%m',parse_date('%Y%m%d',date)) as month
  ,count(visitId) as vistis
  ,sum(totals.pageviews) as pageviews
  ,sum(totals.transactions) as transactions
 FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*` 
 where _table_suffix between '0101' and '0331'
 group by 1
 order by 1````

![image](https://github.com/user-attachments/assets/9e472360-2c56-4158-b372-075db82203ba)
