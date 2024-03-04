
# SecretLab Data Engineering Case Study
This section is a test of your conceptual data engineering understanding - you may code this out for illustration, or you may choose to simply discuss the issues with diagrams and sample code.

# Data Modelling

Create_DB_tables_pg.sql depicts how the data should be formatted after it has landed;
Please describe how you would design a base and subsequent mart layers for OLAP purposes?
Assume the business will need to perform frequent analysis of:

- Revenue
- Item Sales
- Variant Price Changes
- By Product
- By Customer
- By Date
- if payment has been made and the time it occurred

### Data Provided - Batchwise from OLTP
- Order
- Order Line
- Transaction Status
- Product
- Variant

Notes:
- The transaction table is a webhook which lands whenever an update is made on any transaction.

Discuss your architecture considerations as well as pk/fks (soft or hard), indexes and partitions where appropriate - you may assume the real orders table is about 20-30 M records.

# Data Pipelining

## Pipelining

How would you push these data sets assume each new file is **incremental** from S3. Assume that there is a central data warehouse which the BI tool connects to (not S3).

1. Provide a DFD or orchestration diagram, or write up an orchestration plan and tools you may use.


## Cloud Engineering

The Data Scientist has developed an unsupervised model to help analyse traffic flow and conversions into our online shop. 

The model is expected to analyse web-traffic data (sourced from the data warehouse) and output some model results daily. 
The model is delivered to you as a python module.
Model output from the main function is expected to be a dataframe.

You have been tasked to deploy the solution, and allow BI to develop Tableau dashboards based on all the data science model's output for downstream business users.
Discuss how you would implement this system, and provide a simple systems diagram. 

1. Provide a systems diagram only.

# Analytical SQL

### Problem 1

Sometimes products for whatever reason stop selling and a symptom can be an item that was selling well faces a stock out or delisting (or something else). Write a query that shows products that have sold for more than 30 days in the last 60 days, but hasn't had sales for the last week.

You may assume a sales table schema of your preference.

1. Date
2. Product_id
3. Total Items sold to date 
4. number of days with sales
5. number of dates in the recent history where sales have ceased

What would be the best way to implement this in Date/Product_id/Sum(sales)... type Data Mart with other sales information (i.e. without filters and group by)?

### Problem 2 - SQL Optimization

Our Data Analyst needs to compute a `fulfillment promised date` for each of the orderline deliveries based on the Service Level Agreements (SLA) of the logistics provider selected for the delivery. 

The fulfillment promised date is computed based on the fulfillment creation date plus the number of promised working days for delivery provided by the SLA of the logistics provider.

The `working_days` table contains the working days of the world up to 2030.

Part of the query he is using is as follows:

```
select
...
sum(is_working_day::integer order by working_days.date) as number_of_work_days

...
from order_line
left join working_days
on order_line.fulfillment_creation_date < working_days.date
```

He is facing query timeout issues consistently when computing the provider KPIs. What are the steps you would take to help him with this problem?
