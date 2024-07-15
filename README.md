---- Q1: What is the total revenue amount for the second half of 1997 (January 1st 1997 to June 30th 1997)? (Discount is on a percentage basis, e.g. 0.15 means 15% off)
SELECT SUM((1 - discount) * unit_price * quantity) AS total_revenue
FROM order_details
LEFT JOIN orders
ON order_details.order_id = orders.order_id
WHERE order_date BETWEEN '1997-01-01' AND '1997-06-30';

---- Q2: What are the top 5 product that got ordered the most in the Beverages category? Show the product name and the number of orders
SELECT product_name
	   , COUNT(*) AS order_numbers
FROM order_details
LEFT JOIN products
ON order_details.product_id = products.product_id
LEFT JOIN categories
ON products.category_id = categories.category_id
WHERE category_name = 'Beverages'
GROUP BY product_name
ORDER BY order_numbers DESC
LIMIT 5;

---- Q3: What are the top 5 product category that are sold with the highest discount on average? (Using simple average would be fine)
SELECT category_name
	   , ROUND(CAST(AVG(discount) * 100 AS Decimal), 2) AS average_discount
FROM order_details
LEFT JOIN products
ON order_details.product_id = products.product_id
LEFT JOIN categories
ON products.category_id = categories.category_id
GROUP BY category_name
ORDER BY average_discount DESC
LIMIT 5;

---- Q4: Northwind wants to learn a bit more about how popular each supplier's goods are. They plan to learn about how much volume each supplier's goods was sold to determine the future negotiations with them about pricing. They have classified the popularity of the supplier's goods as such: 
---------If the total order quantity of the supplier's goods is more than or equal to 3000, the supplier would be classified as 'Highly Popular'
---------If the total order quantity of the supplier's goods is more than 1500 but less than 3000, the supplier would be classified as 'Adequately Popular'
---------If the total order quantity of the supplier's goods is less than or equal to 1500, the supplier would be classified as 'Not Popular'
---------Please create a view that shows the average sales revenue for each popularity classification
WITH supplier_popularity AS (SELECT company_name
								   , SUM(quantity) AS sales_quantity
								   , CASE WHEN SUM(quantity) > 3000 THEN 'Highly Popular'
										  WHEN SUM(quantity) BETWEEN 1501 AND 3000 THEN 'Adequately Popular'
										  ELSE 'Not Popular' END AS supplier_goods_popularity
							       , SUM((1 - discount) * order_details.unit_price * quantity) AS sales_revenue
							FROM order_details
							LEFT JOIN products
							ON order_details.product_id = products.product_id
							LEFT JOIN suppliers
							ON products.supplier_id = suppliers.supplier_id
							GROUP BY company_name
							ORDER BY sales_quantity DESC)
SELECT supplier_goods_popularity
	   , AVG(sales_revenue) AS avg_sales_revenue
FROM supplier_popularity
GROUP BY supplier_goods_popularity
ORDER BY avg_sales_revenue;

---- Q5: What is the top 3 most popular product category in the country that contributes to the most amount of revenue? 

---- Solution 1: With Subquery Only
SELECT  country
		, category_name
		, category_revenue_ranking
FROM (SELECT country
		   	, category_name
	        , SUM((1 - discount) * order_details.unit_price * quantity) AS category_revenue
	        , RANK() OVER(PARTITION BY country ORDER BY SUM((1 - discount) * order_details.unit_price * quantity) DESC) AS category_revenue_ranking
	   FROM order_details 
		    LEFT JOIN orders 
		   		ON order_details.order_id = orders.order_id
			LEFT JOIN customers
				ON orders.customer_id = customers.customer_id
		    LEFT JOIN products 
		   		ON order_details.product_id = products.product_id
		    LEFT JOIN categories
		   		ON products.category_id = categories.category_id
			WHERE country = (SELECT country
							 FROM (SELECT country
								  		  , RANK() OVER(ORDER BY SUM((1 - discount) * order_details.unit_price * quantity) DESC) AS country_revenue_ranking
								   FROM order_details 
								   LEFT JOIN orders 
										ON order_details.order_id = orders.order_id
								   LEFT JOIN customers
										ON orders.customer_id = customers.customer_id
								   GROUP BY country)
							WHERE country_revenue_ranking = 1)
			GROUP BY country
					 , category_name)
WHERE category_revenue_ranking <= 3;

--- Solution 2: With CTE
WITH country_ranking AS (SELECT country
								, RANK() OVER(ORDER BY SUM((1 - discount) * order_details.unit_price * quantity) DESC) AS country_revenue_ranking
						 FROM order_details 
						 LEFT JOIN orders 
							 ON order_details.order_id = orders.order_id
						 LEFT JOIN customers
							 ON orders.customer_id = customers.customer_id
						 GROUP BY country),
	category_ranking_with_country AS (SELECT country
										   , category_name
										   , SUM((1 - discount) * order_details.unit_price * quantity) AS category_revenue
									       , RANK() OVER(PARTITION BY country ORDER BY SUM((1 - discount) * order_details.unit_price * quantity) DESC) AS category_revenue_ranking
										FROM order_details 
										LEFT JOIN orders 
											ON order_details.order_id = orders.order_id
										LEFT JOIN customers
											ON orders.customer_id = customers.customer_id
										LEFT JOIN products 
											ON order_details.product_id = products.product_id
										LEFT JOIN categories
											ON products.category_id = categories.category_id
									 GROUP BY country
										      , category_name)
SELECT country_ranking.country
	   , category_name
	   , category_revenue_ranking
FROM country_ranking
LEFT JOIN category_ranking_with_country
   ON country_ranking.country = category_ranking_with_country.country
WHERE country_revenue_ranking = 1
	AND category_revenue_ranking <= 3;
		

---- Q6: Who is the customer that orders the most from Northwind? How much revenue they contributed?

--- Solution 1:
SELECT customer_id 
	   , SUM((1 - discount) * unit_price * quantity) AS total_revenue
FROM orders a
LEFT JOIN order_details b
	ON a.order_id = b.order_id
GROUP BY customer_id
ORDER BY total_revenue DESC
LIMIT 1;

--- Solution 2 (Window Function):
SELECT customer_id 
	, SUM((1 - discount) * unit_price * quantity) AS total_revenue
	, RANK() OVER(ORDER BY SUM((1 - discount) * unit_price * quantity) DESC) AS revenue_ranking
FROM orders a
LEFT JOIN order_details b
	ON a.order_id = b.order_id
GROUP BY customer_id
ORDER BY revenue_ranking;


--- Q7: For the same customer from the last question, please create a view that shows the 3-month moving average total amount under each product category.
WITH a AS (SELECT customer_id
		   		  , DATE_TRUNC('MONTH', order_date) AS order_month
		   		  , category_name
		          , SUM((1 - discount) * order_details.unit_price * quantity) AS order_total
		   FROM order_details 
		   LEFT JOIN orders 
		   		ON order_details.order_id = orders.order_id 
		   LEFT JOIN products 
		   		ON order_details.product_id = products.product_id
		   LEFT JOIN categories
		   		ON products.category_id = categories.category_id
		  
		   GROUP BY customer_id
		   			, order_month
		  			, category_name
		   ORDER BY customer_id
		   )

SELECT a.customer_id
	   , order_month
	   , order_total
	   , category_name
	   , AVG(order_total) OVER(PARTITION BY a.customer_id, category_name ORDER BY order_month ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg_3_mths
FROM a
 WHERE a.customer_id='QUICK';
 
--- Q8: For the same customer, what proportion of revenue does the customer contribute to the total revenue in each product category?
WITH revenue_overall AS (SELECT category_name
						 		, SUM((1 - discount) * order_details.unit_price * quantity) AS total_revenue
							 FROM order_details 
							   LEFT JOIN products 
									ON order_details.product_id = products.product_id
							   LEFT JOIN categories
									ON products.category_id = categories.category_id
						 GROUP BY category_name)
SELECT customer_id
	   , categories.category_name
	   , SUM((1 - discount) * order_details.unit_price * quantity) AS customer_revenue
	   , total_revenue
	   , ROUND(CAST(SUM((1 - discount) * order_details.unit_price * quantity) / total_revenue * 100 AS numeric), 2) AS percent_of_total_sales
FROM order_details
LEFT JOIN orders 
	ON order_details.order_id = orders.order_id 
LEFT JOIN products 
	ON order_details.product_id = products.product_id
LEFT JOIN categories
	ON products.category_id = categories.category_id
LEFT JOIN revenue_overall
	ON categories.category_name = revenue_overall.category_name
WHERE orders.customer_id = 'QUICK'
GROUP BY customer_id
	   , categories.category_name
	   , total_revenue;
