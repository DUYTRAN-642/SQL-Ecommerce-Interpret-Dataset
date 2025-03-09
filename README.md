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
├── /queries/           # SQL scripts
│   ├── query1.sql      # Monthly visits, pageviews, and transactions
│   ├── query2.sql      # Traffic source analysis with bounce rates
│   ├── query3.sql      # Revenue by source (month and week)
│   ├── query4.sql      # Avg pageviews for purchasers vs. non-purchasers
│   ├── query5.sql      # Avg transactions per user
│   ├── query6.sql      # Avg revenue per user per visit
│   ├── query7.sql      # Products co-purchased with "YouTube Men's Vintage Henley"
│   ├── query8.sql      # Conversion funnel (views, add-to-cart, purchases)
├── /docs/              # Documentation
│   └── README.md       # This file
└── /data/              # Optional: Sample outputs or schemas
