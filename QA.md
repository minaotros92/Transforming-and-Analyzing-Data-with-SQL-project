What are your risk areas? Identify and describe them.


The risk areas are an incomplete country and city pairs. For instance, there were many unknown cities that I changed to the country name. Also, I noticed some pairs were incorrect upon answering the questions like United States and Paris. 

There is a chance duplicates still exist in both the All Sessions and Analytics table due to error in my code. In addition, there also may be nulls still present.

I performed my QA on the views I created to answer the assignment questions and they can be found in the Cleaning Data file.

QA Process:
Describe your QA process and include the SQL queries used to execute it.


All Sessions:

``` select full_visitor_id, visit_id, count(*)
from all_sessions_view
group by full_visitor_id, visit_id
having count(*) > 1 ```

``` select country, city, product_sku, product_name, product_category
from all_sessions_view
where country is null 
	or city is null
	or product_sku is null 
	or product_name is null 
	or product_category is null
group by country, city, product_sku, product_name, product_category
 ```

``` select distinct count(product_category)
from all_sessions_view
where product_category like '%/%' 
``` 














Analytics:

 select full_visitor_id, visit_id, count(*)
from analytics_view
group by full_visitor_id, visit_id
having count(*) > 1 

select units_sold, formatted_price
from analytics_view
where units_sold is null or formatted_price is null

 select full_visitor_id, visit_id
from analytics_view
where full_visitor_id is null or visit_id is null 



Products:

``` select sku, count(*)
from products_view
group by sku
having count(*) > 1 ```

``` select *
from products_view
where sku is null
	or product_name is null
	or ordered_quantity is null
	or stock_level is null
	or restocking_lead_time is null
	or sentiment_score is null
	or sentiment_magnitude is nulll ```

``` select product_name
from products_view
where product_name like ' ' ```


Sales by SKU:

``` select product_sku, count(*)
from sbs_view
group by product_sku
having count(*) > 1 ```

``` select *
from sbs_view
where product_sku is null
	or total_ordered is null ```






Sales Report:

``` select product_sku, count(*)
from sales_report_view
group by product_sku
having count(*) > 1 ```

``` select *
from sales_report_view
where product_sku is null
	or total_ordered is null
	or product_name is null
	or stock_level is null
	or sentiment_score is null
	or sentiment_magnitude is null
	or ratio is null ```
