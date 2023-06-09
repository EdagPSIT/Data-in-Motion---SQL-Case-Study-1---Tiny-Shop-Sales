-- Case Study Questions

--1) Which product has the highest price? Only return a single row.
SELECT 
	product_name
FROM products
WHERE price = (SELECT max(price) FROM products)
;

--2) Which customer has made the most orders?
WITH most_orders AS (
  SELECT 
      customer_id,
      count(customer_id) as most_order
  FROM orders
  GROUP BY customer_id
  -- ORDER BY most_orders DESC
  ),
  most_order_customer AS(
    SELECT customer_id
    FROM most_orders
    WHERE most_order = (SELECT max(most_order) FROM most_orders)
    )
  SELECT first_name , last_name
  FROM customers
  WHERE customer_id IN (SELECT customer_id FROM most_order_customer);
  ;

--3) What’s the total revenue per product?
WITH product_orders AS(
  	SELECT product_name,
  			price, 
  			quantity
  	FROM products p
  	INNER JOIN order_items oi
  	ON p.product_id = oi.product_id
  	)
    SELECT product_name,
    		sum(price * quantity) AS revenue	
    FROM product_orders
    GROUP BY product_name
    ORDER BY revenue DESC;

--4) Find the day with the highest revenue.
SELECT 
	order_date,
    sum(price * quantity) AS revenue
FROM orders o
INNER JOIN order_items oi
ON o.order_id = oi.order_id
INNER JOIN products p
ON oi.product_id = p.product_id
GROUP BY order_date
ORDER BY revenue DESC;


--5) Find the first order (by date) for each customer.
SELECT first_name || ' '|| last_name AS Customer_Name,
    min(order_date) AS first_order_date
FROM customers c
INNER JOIN orders o
ON c.customer_id = o.customer_id
GROUP BY first_name || ' '|| last_name
ORDER BY Customer_Name;

--6) Find the top 3 customers who have ordered the most distinct products
SELECT 
	first_name || ' ' || last_name AS Customer_Name,
    COUNT(DISTINCT product_id) AS prodcut_count
FROM customers c
INNER JOIN orders o
ON c.customer_id = o.customer_id
INNER JOIN order_items oi
ON o.order_id = oi.order_id
GROUP BY first_name || ' ' || last_name
ORDER BY prodcut_count DESC
LIMIT 3;
	

--7) Which product has been bought the least in terms of quantity?
SELECT product_name,
	COUNT(p.product_id) AS product_count
FROM products p
INNER JOIN order_items oi
ON p.product_id = oi.product_id
GROUP BY product_name
ORDER BY product_count ASC;

--8) What is the median order total?

--9) For each order, determine if it was ‘Expensive’ (total over 300), ‘Affordable’ (total over 100), or ‘Cheap’.
-- I Method : Using Common Table Expression
WITH order_status AS (
  SELECT o.order_id,
      sum(quantity * price) AS total_cost
  FROM orders o
  LEFT JOIN order_items oi
  ON o.order_id = oi.order_id
  INNER JOIN products p
  ON oi.product_id = p.product_id
  GROUP BY o.order_id
  )
  SELECT order_id,
  	CASE 
    	WHEN total_cost > 300 THEN 'Expensive'
        WHEN total_cost > 100 THEN 'Affordable'
        ELSE 'Cheap'
    END AS Price
  FROM order_status;
  
  -- II Method : Complex to read query
  SELECT o.order_id,
  	CASE 
    	WHEN sum(quantity * price) > 300 THEN 'Expensive'
        WHEN SUM(quantity * price) > 100 THEn 'Affordable'
        ELSE 'Cheap'
    END AS Price
  FROM orders o
  LEFT JOIN order_items oi
  ON o.order_id = oi.order_id
  INNER JOIN products p
  ON oi.product_id = p.product_id
  GROUP BY o.order_id;



--10) Find customers who have ordered the product with the highest price.
/*
	Steps:
    	1. Find the product with highest price
        2. Find the customers who have brought this product
*/

WITH expensive_Product AS(
  SELECT product_id 
  FROM products
  WHERE price = (
    SELECT MAX(price) FROM products
  )),
  cust_prod_order AS(
    SELECT first_name || ' ' || last_name AS customer_name,
    product_name,
    price
    FROM customers c
    INNER JOIN orders o
    ON c.customer_id = o.customer_id
    INNER JOIN order_items oi
    ON o.order_id = oi.order_id
    INNER JOIN products p
    ON oi.product_id = p.product_id
    )
  SELECT customer_name,
  	product_name,
  	price
  FROM cust_prod_order
  WHERE product_id = (SELECT product_id FROM expensive_product);


	