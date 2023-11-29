
What issues will you address by cleaning the data?


I searched  for the following errors
- Duplicates and nulls .
- Proper data types .
- Any additional formatting like capitalization or condensing of strings,
- Does values and data types make sense

Below is a summary of what I did to the data:
all_sessios file :time coulmn not specified tupe by h/m/s , alot of missing informayion like country 24 (not set) , city 24 (not set) 8302 not available in database , total_transaction 15053 Null, sessios_quality 139906 Null
, product_revenue 15081 Null ,item_quantity +item_revenue +transaction_id ( most of them null) and page_path_level1 (14318 google+resighn )

* use trim to trim the white space in product names.
* decreased the unit price column in analytics by divided by	1,000,000.
* converted dates to date data types with CAST (::)
* use CASE statements to fix any cities that were not cities. 
* used COALESCE to convert any null in the unit sold to zero 
* I attempted to condense the product categories, but only managed to 	remove a portion, thus leaving the product category improperly 	cleaned.
* I struggled with trying to map the incorrect values 	to the correct one, so I just made a case statement to make them 	‘Unknown’.
* The ratio column was determined to be (total ordered / stock level);divided by 1000,000 factor



Queries:
Below, provide the SQL queries you used to clean your data.

all_sessions table 

CREATE OR REPLACE VIEW all_sessions_view AS (
    WITH cte_asv AS (
SELECT 
 full_visitor_id,
visit_id,
 country,
 CASE
  WHEN city = '(not set)' THEN country
  WHEN city = 'not available in demo dataset' THEN country
ELSE city
  END AS city,
date::date AS date,
product_sku,
 TRIM(v2_product_name) AS product_name,
 CASE
 WHEN v2_product_category = '(not set)' THEN 'Unknown'
 ELSE v2_product_category
 END AS product_category,
 ROUND(product_price / 1000000.0, 2) AS formatted_price, -- Added formatted price
 ROW_NUMBER() OVER (PARTITION BY full_visitor_id, visit_id) AS row_cte_asv
 FROM all_sessions
 WHERE country != '(not set)'
    )
SELECT *
 FROM cte_asv
WHERE row_cte_asv = 1
)




table anayltics

create or replace view analytics_view as (
with cte_analytics as (
select
full_visitor_id,
visit_id,
date::varchar::date as date,
visit_numbers,
channel_grouping,
"social_engagement_type",
coalesce(units_sold, 0) as units_sold,
ROUND(unit_price::numeric / 1000000.0, 2) AS formatted_price,
row_number() over (partition by full_visitor_id, visit_id, visit_numbers) as row_cte_analytics
from analytics
    )
 select *
from cte_analytics
 where row_cte_analytics = 1
)


product table

create or replace view products_view as (
with cte_products as(
select
sku,
trim(name) as product_name,
ordered_quantity,
stock_level,
restocking_lead_time,
sentiment_score,
sentiment_magnitude,
row_number() over (partition by sku, name) as 			row_products_view
from products
	)
	select *
	from cte_products
	where row_products_view = 1
)




table sales_by_sku

create or replace view sales_by_sku_view as (
with cte_sales_by_sku as (
select 
product_sku,
total_ordered,
row_number() over (partition by product_sku, 					total_ordered) as row_sales_by_sku_view
from sales_by_sku
	)
	select *
	from cte_sales_by_sku
	where row_sales_by_sku_view = 1
)


table sales_report

create or replace view sales_report_view as (
with cte_sales_report as (
select
product_sku,
total_ordered,
trim(name) as product_name,
stock_level,
sentiment_score,
sentiment_magnitude,
case 
when round(ratio::numeric,2) is null then 'N/A'
else round(ratio::numeric,2)::varchar(100)
end as ratio,
row_number() over (partition by product_sku, name) as 			row_cte_sales_report
from sales_report
	)
	select *
	from cte_sales_report
	where row_cte_sales_report = 1
)

