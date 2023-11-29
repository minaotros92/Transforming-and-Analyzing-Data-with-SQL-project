Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:
select
    country,
    city,
    sum(units_sold * av.formatted_price) as total_revenue
from analytics_view av
join all_sessions_view asv
on av.full_visitor_id = asv.full_visitor_id
group by country, city
order by total_revenue desc
limit 10


Answer:

"United States"		"San Bruno"			74479.00
"United States"		"United States"		35717.80
"United States"		"Mountain View"		5237.65
"United Kingdom"	"London"			417.45
"United States"		"Chicago"			383.99
"United States"		"Jersey City"		377.85
"United States"		"New York"			368.00
"Canada"			"Toronto"			367.38
"United States"		"Sunnyvale"			249.00
"Switzerland"		"Zurich"			212.99



**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:

select
	country,
	city,
	round(avg(units_sold), 0) as avg_products_ordered
from analytics_view av
join all_sessions_view asv
on av.full_visitor_id = asv.full_visitor_id
where units_sold > 0
group by country, city
order by avg(units_sold) desc
limit 10



Answer:

"United States"		"United States"		49
"United States"		"Jersey City"		10
"United States"		"Sunnyvale"			10
"United States"		"Paris"				5
"United States"		"Milpitas"			5
"Canada"			"Canada"			2
"United States"		"Mountain View"		2
"United States"		"San Francisco"		2
"United States"		"Los Angeles"		2
"United States"		"Chicago"			1





**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:
with product_category_country as (
	select
	country,
	city,
	product_category,
	asv.product_name,
	units_sold,
	dense_rank() over (partition by country, city, 			product_category order by units_sold) as product_rank
	from analytics_view av
	join all_sessions_view asv
	on av.full_visitor_id = asv.full_visitor_id
	join products_view pv
	on asv.product_sku = pv.sku
	where units_sold > 0
	group by country, city, product_category, asv.product_name, units_sold
	order by country desc
)
select country, city, product_category, product_name, units_sold, product_rank
from product_category_country
where product_rank = 1
group by country, city, product_category, product_name, units_sold, product_rank
order by units_sold desc





Answer:

Top Product Categories per City
In several cities across different countries, a variety of product categories emerge as the most popular. For instance:

"United States"		"United States	"		"Home/Office/Notebooks & Journals/"	"YouTube Leatherette Notebook Combo"
"United States"		"United States"			"Unknown"	"Google Women's Fleece Hoodie"
"United States"		"Chicago"				"Home/Shop by Brand/YouTube/"	"YouTube Custom Decals"
"United States"		"New York"				"Home/Nest/Nest-USA/"	"Nest® Protect Smoke + CO White Wired Alarm-USA"
"United States"		"United States"			"Home/Accessories/Pet/"	"7&quot; Dog Frisbee"
"Canada"			"Canada"				"Apparel"	"Google Men's Vintage Badge Tee White"
"Canada"			"Canada"				"Home/Shop by Brand/YouTube/"	"YouTube Leatherette Notebook Combo"
"Canada"			"Toronto"				"Home/Apparel/Headgear/"	"Android Stretch Fit Hat Black".

These patterns suggest that product preferences can vary significantly both within and across countries. For instance, while 'Apparel' is the overall leader in Canada, specific cities like Toronto show a preference for different categories like 'Home/Apparel/Headgear/'. Similarly, in the U.S., while 'Home/Office/Notebooks & Journals/' is popular nationally, cities have their distinct favorites.





**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:

with product_category_country as (
	select
	country,
	city,
	product_category,
	asv.product_name,
	units_sold,
	dense_rank() over (partition by country, city order by units_sold desc) as product_rank
	from analytics_view av
	join all_sessions_view asv
	on av.full_visitor_id = asv.full_visitor_id
	join products_view pv
	on asv.product_sku = pv.sku
	where units_sold > 0
	group by country, city, product_category, asv.product_name, units_sold
	order by units_sold desc
)
select country, city, product_category, product_name, units_sold, product_rank
from product_category_country
where product_rank = 1
group by country, city, product_category, product_name, units_sold, product_rank
order by country desc
limit 10



Answer:

country				 city 				product_category					product_name 									units_sold 	product_rank
"United States"		"Charlotte"		"Home/Bags/"						"Google Lunch Bag"									1			1
"United States"		"Chicago"		"Home/Shop by Brand/YouTube/"		"YouTube Custom Decals"								2			1
"United States"		"Fremont"		"Home/Nest/Nest-USA/"				"Nest® Protect Smoke + CO White Battery Alarm-USA"	1			1
"United States"		"Los Angeles"	"Home/Nest/Nest-USA/"				"Nest® Cam Indoor Security Camera - USA"			2			1
"United States"		"Mountain View"	"Home/Nest/Nest-USA/"				"Nest® Learning Thermostat 3rd Gen-USA - White"		10			1
"United States"		"New York"		"Home/Nest/Nest-USA/"				"Nest® Protect Smoke + CO White Wired Alarm-USA"	2			1
"United States"		"Palo Alto"		"Home/Nest/Nest-USA/"				"Nest® Learning Thermostat 3rd Gen-USA - Stainless Steel"	2	1
"United States"		"Paris"			"Home/Apparel/Women's/Women's-Outerwear/"	"Google Women's Quilted Insulated Vest Black"	5		1
"United States"		"San Francisco"	"Home/Nest/Nest-USA/"				"Nest® Learning Thermostat 3rd Gen-USA - Stainless Steel"	3	1
"United States"		"Sunnyvale"		"Home/Office/Notebooks & Journals/"	"Google Hard Cover Journal"	40	1





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:

with revenue_summary as (
    select
        country,
        city,
        sum(units_sold * av.formatted_price) as total_revenue, -- Specify the table alias here
        (select sum(units_sold * formatted_price)
         from analytics_view
         where units_sold > 0) as site_revenue
    from analytics_view av
    join all_sessions_view asv
    on av.full_visitor_id = asv.full_visitor_id
    where units_sold > 0
    group by country, city
    order by total_revenue desc
)
select country, city, round((total_revenue / site_revenue), 5) * 100 as percent_of_total_revenue
from revenue_summary
group by country, city, (total_revenue / site_revenue)
order by (total_revenue / site_revenue) desc





Answer:

country				 city 			percent_of_total_revenue
"United States"		"San Bruno"			8.44500
"United States"		"United States"		4.05000
"United States"		"Mountain View"		0.59400
"United Kingdom"	"London"			0.04700
"United States"		"Chicago"			0.04400
"United States"		"Jersey City"		0.04300
"United States"		"New York"			0.04200
"Canada"			"Toronto"			0.04200
"United States"		"Sunnyvale"			0.02800
"Switzerland"		"Zurich"			0.02400
"Panama"			"Panama"			0.02300
"United States"		"Paris"				0.01900
"United States"		"Austin"			0.01500
"Hong Kong"			"Hong Kong"			0.01400
"United States"		"San Francisco"		0.00900
"United States"		"Seattle"			0.00700
"Japan"				"Japan"				0.00600
"United States"		"Charlotte"			0.00400
"Indonesia"			"Jakarta"			0.00400
"United States"		"Ann Arbor"			0.00200
"Netherlands"		"Netherlands"		0.00200
"Japan"				"Minato"			0.00200







