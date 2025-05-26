# Data Analyst Workflow: Tasks, Hardskills, Strategy, Solutions

## Table of Contents
1. [Project Overview](#project-overview)
2. [Technologies Used](#technologies-used)
3. [Methodology](#methodology)
4. [Key Findings](#key-findings-and-perspectives)
5. [Next Steps and Recommendations](#next-steps-and-recommendations)

## Project Overview

This project brings together end-to-end data analysis and engineering practices for a client-profile database and product metrics evaluation. It covers:

- **Database Design & Implementation**: A normalized 3NF schema (`users`, `countries`, `emails_sent`, `emails_clicks`) with performance-oriented keys and indexes, plus synthetic data generation for testing.
    
- **SQL Query Optimization**: Best-practice rules and examples for writing efficient SQL, including the use of `EXPLAIN ANALYZE` to validate execution plans.
    
- **Product Metrics & Cohort Analysis**: Calculation of ARPU (Average Revenue Per User), LTV (Lifetime Value), CAC (Customer Acquisition Cost), retention curves, and total revenue by install cohorts using Python and pandas.
    
- **A/B Testing**: Design, execution, and analysis of experiments (e.g., heart vs. checkmark icon engagement), with effect size calculation (Cohen’s d), statistical tests (Mann–Whitney U or t-test), and power analysis.
    

## Technologies Used

- **Database**: Oracle 19c (DDL generated via Oracle SQL Developer Data Modeler), MySQL for local testing
    
- **ETL & Scripting**: python (pandas, SQLAlchemy, numpy, scipy, statsmodels, matplotlib)
    
- **SQL Optimization**: `ALTER INDEX`, composite indexes, partitioning, `EXPLAIN ANALYZE`
    
- **Visualization & Dashboards**: matplotlib plots; recommendations for interactive dashboards (Tableau Desktop, Power BI, Amplitude, or Python Streamlit)
    
    

## Methodology

1. **Database Development & Testing**  
    • Defined a clear data dictionary and ER schema in 3NF.  
    • Generated synthetic data via pandas + SQLAlchemy for unit testing and performance benchmarks.  
    • Applied SQL optimization principles (indexes, partitions, NOT NULL, CHECK constraints).
    
2. **Cohort Analysis & Product Metrics**  
    • Computed ARPU at 3, 6, and 44 weeks, retention rates, average user lifetime, LTV, CAC, and revenue by install-week intervals.  
    • _Future Enhancement_: develop a time-series forecasting model to predict an additional 8 weeks of revenue, extending the dataset from 44 to 52 weeks for true 1-year LTV projection.
    
3. **A/B Test Design & Analysis**  
    • Segmented users by parity of `sender_id` into control (heart icon) and test (checkmark).  
    • Calculated Average Likes Per User, Cohen’s d, power analysis, and applied Mann–Whitney U or Welch’s t-test.  
    • _Future Enhancement_: build interactive KPI dashboards for real-time monitoring and deeper segmentation (e.g., by gender, platform) to uncover targeted business insights.
    

## Key Findings and Perspectives

- **Database Performance**: Proper 3NF normalization combined with targeted indexing improved join and filter performance by up to 20× in benchmarks.
    
- **Monetization Metrics**: ARPU grew from $0.49 (3 mo) to $0.73 (44 wk - supposed to be 1 year), with an average user lifetime of ~6.8 weeks, informing CAC targets around $0.50 for profitable growth.
    
- **Revenue Cohorts**: Recent install cohorts (weeks 25–28) outperformed earlier ones, suggesting optimization of acquisition channels and creatives.
    
- **A/B Testing**: The icon change experiment yielded a negative Cohen’s d (–0.044), indicating lower engagement in the test group; overlapping confidence intervals underscore the need for larger sample sizes.
    

## Next Steps and Recommendations 

- Extend the revenue time series by forecasting the missing 8 weeks to complete a 52-week LTV analysis.
    
- Develop dynamic dashboards with key KPIs (ARPU, LTV, retention curves, A/B results) to enable the marketing team to drill into segments and monitor impact.
    
- Incorporate chi-squared tests for categorical variable interactions and stratify A/B results by demographic segments (e.g., gender, platform).
    
- Continuously validate SQL performance in production with `EXPLAIN ANALYZE` and automate alerting on query plan regressions.
