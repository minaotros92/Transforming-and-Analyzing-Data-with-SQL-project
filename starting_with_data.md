Question 1: What are the Top-Selling Products in Terms of Units Sold, and How Does Their Sales Distribution Look Across Different Countries?

SQL Queries:

SELECT p.product_name, sr.product_sku, SUM(sr.total_ordered) AS total_units_sold, a.country
FROM sales_report_view sr
JOIN products_view p ON sr.product_sku = p.sku
JOIN all_sessions_view a ON sr.product_sku = a.product_sku
GROUP BY p.product_name, sr.product_sku, a.country
ORDER BY total_units_sold DESC;



Answer: 


	product name 										product sku					total units Sold		country
"17oz Stainless Steel Sport Bottle"						"GGOEGDHC074099"				14028			"United States"
"Ballpoint LED Light Pen"								"GGOEGOAQ012899"				8664			"United States"
"Cam Outdoor Security Camera - USA"						"GGOENEBQ078999"				7280			"United States"
"Cam Indoor Security Camera - USA"						"GGOENEBB078899"				6732			"United States"
"Android 17oz Stainless Steel Sport Bottle"				"GGOEADHH073999"				6346			"United States"
"Leatherette Journal"									"GGOEGOCB017499"				6061		"United States"
"Learning Thermostat 3rd Gen-USA - Stainless Steel"		"GGOENEBJ079499"				4982			"United States"
"22 oz  Bottle Infuser"									"GGOEYDHJ056099"				4900			"United States"



Question 2: How does the sentiment score of products correlate with their sales performance?

SQL Queries:

SELECT p.product_name, p.sentiment_score, s.total_ordered
FROM products_view p
JOIN sales_report_view s ON p.sku = s.product_sku
ORDER BY p.sentiment_score DESC, s.total_ordered DESC;


Answer:

product name    								sentiment score 		total ORDER
"Metal Texture Roller Pen"								0.9					48
"Recycled Paper Journal Set"							0.9					30
"22 oz Water Bottle"									0.9					23
"Hard Cover Journal"									0.9					20
"Men's 100% Cotton Short Sleeve Hero Tee White"			0.9					17
	

Question 3: Which products have a high restocking lead time but low sales?

SQL Queries:
select
	SELECT p.product_name, p.restocking_lead_time, s.total_ordered
FROM products_view p
JOIN sales_by_sku_view s ON p.sku = s.product_sku
WHERE p.restocking_lead_time > (SELECT AVG(restocking_lead_time) FROM products_view)
ORDER BY s.total_ordered ASC;


Answer:

product name									restock lead time		 total ordered
"Android BTTF Cosmos Graphic Tee"					15						0
"Women's Tee Grey"									13						0
"Android Stretch Fit Hat"							16						0
"Men's Long Sleeve Raglan Ocean Blue"				17						0
"Android Men's Long & Lean Badge Tee Charcoal"		18						0
"Pet Feeding Mat"									18						0
"Collapsible Pet Bowl"								15						0
"Men's Long & Lean Tee Charcoal"					14						0
"Android Men's Short Sleeve Tri-blend Hero Tee Grey"	16					0
"Android Women's Short Sleeve Badge Tee Dark Heather"	15					0
"Android Lunch Kit"										12					0
"Women's Short Sleeve Tri-blend Badge Tee Grey"			15					0
"Men's Long Sleeve Raglan Ocean Blue"					18					0



Question 4: Analysis of Visitor Engagement and Sales by Channel Grouping

SQL Queries:
select 
	SELECT a.channel_grouping, AVG(a.visit_numbers) AS avg_visit_numbers, SUM(a.units_sold) AS total_units_sold
FROM analytics_view a
GROUP BY a.channel_grouping
ORDER BY total_units_sold DESC;

Answer:

channel grouping 			 avg visit number 		 total units sold
	
"Referral"					2.8851951126118286			3443
"Direct"					2.0290717903119805			3329
"Organic Search"			1.8855059753655248			2334
"Display"					17.1215986394557823			649
"Social"					1.9312711864406780			83
"Paid Search"				2.5003188097768332			80
"Affiliates"				1.8342089900758903			34
"(Other)"					2.5000000000000000			0