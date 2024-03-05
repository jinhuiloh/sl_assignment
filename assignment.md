# Data Modelling

Data warehouse layer
- A denormalized table with columns from multiple tables joined into a single table so that data can be queried faster, reducing the need for similar repeated join queries
```
Sample sql:
    select
        a.id as order_id
        ,a.processed_timestamp
        ,a.customer_id
        ,b.id as order_lines_id
        ,b.quantity
        ,b.revenue
        ,b.variant_id
        ,c.product_id
        ,c.price as variant_price
        ,d.status
        ,d.created_at as status_time
    from orders a
    left join order_lines b
    on a.id=b.ORDER_ID
    left join variant c
    on b.variant_id=c.id
    left join (select order_id,status,created_at from transaction where status='success') d
    on a.id=d.order_id
In this sql, I am only including the id of the products/variants because of storage consideration.
Name columns can be included in the table if there is a need to or we can just join the relevant tables in downstream pipeline.
```

Data mart layer
- Can create multiple aggregated tables from the denormalized table, aggregated with the dimension/s needed
- Dashboards can just query the aggregated table with just filter conditions without any additional aggregations

### Data Provided - Batchwise from OLTP

Architectual consideration:
Assuming that we are extracting to S3 and loading to Redshift as data warehouse

1) Data extraction
- Whether extracting data will affect the production databases
- can consider extracting from replica/read-only db
- data size(full/incremental extract)

2) File type to keep the data in(this depends on the data warehouse we are using)

3) Loading of data
- I will suggest to load all the data we extracted to the data warehouse first before doing any cleaning
(this is on the basis that what we extracted is the same as the source db)
- Suggest to create a temporary table to copy the data from S3 to redshift, then insert into a staging table
(this will reduce data issue resulting from potential human error, ie. runnning the job multiple times and copy data into the table resulting in duplicates)
- the staging table should have the same pk as source table in production db
- considering that transaction data has a record for every update, we will need to take note of this when using the data

4) data warehouse tables(DW layer)
- the tables in the DW layer should have sort keys and distribution keys selected according to the frequent usage of the table
(ie. time,customer_id,product_id)

# Data Pipelining

## Pipelining
Image attached in another file

## Cloud Engineering
Image attached in another file

# Analytical SQL
```
select
    date
    ,product_id
    ,items_sold_to_date
    ,days_with_sales
    ,date_diff('days',max_date,date) as days_sales_ceased
from(
    select
        distinct
        ,current_date as date
        ,product_id
        ,sum(item_sold) over (partition by product_id) as items_sold_to_date
        ,count(distint date) over (partition by product_id) as days_with_sales
        ,max(date) over (partition by product_id) max_date
    from sales
)a
```

### Problem 2 - SQL Optimization

Looking at the query, the main issue is the data exploding because of using the full working_days table.
Considering the fact that the sla should not be too long, we should not be using the full working_days table.
Instead we should restrict the date in working_days table(ie. <30 days from now,depending on the sla)
We can also filter on the order_line table to only include the range of the duration we are planning to analyse
