**Case Study Questions**

The following questions can be considered key business questions and metrics that the Balanced Tree team requires for their monthly reports.

Each question can be answered using a single query - but as you are writing the SQL to solve each individual problem, keep in mind how you would generate all of these metrics in a single SQL script which the Balanced Tree team can run each month.

**High Level Sales Analysis**

What was the total quantity sold for all products?

```sql
SELECT p.product_id,
       p.product_name,
       SUM(s.qty) AS total_sold
	FROM product_details AS p
	JOIN sales AS s ON p.product_id = s.prod_id
	GROUP BY p.product_id, p.product_name
	ORDER BY p.product_name;
```

| product_id | product_name | total_sold |
|------------|--------------|------------|
| e83aa3 | Black Straight Jeans - Womens |	3786 |
| 2a2353 | Blue Polo Shirt - Mens | 3819 |
| e31d39 | Cream Relaxed Jeans - Womens |	3707 |
| 9ec847 | Grey Fashion Jacket - Womens |3876 |
| 72f5d4 | Indigo Rain Jacket - Womens | 3757 |
| d5e9a6 | Khaki Suit Jacket - Womens | 3752 |
| c4a632 | Navy Oversized Jeans - Womens | 3856 |
| f084eb | Navy Solid Socks - Mens | 3792 | 
| 2feb6b | Pink Fluro Polkadot Socks - Mens | 3770 |
| c8d436 | Teal Button Up Shirt - Mens | 3646 |
| b9a74d | White Striped Socks - Mens | 3655 |
| 5d267b | White Tee Shirt - Mens | 3800 |


What is the total generated revenue for all products before discounts?

```sql
SELECT p.product_id,
       p.product_name,
       SUM(s.qty*s.price) AS total_sales
	FROM product_details AS p
	JOIN sales AS s ON p.product_id = s.prod_id
	GROUP BY p.product_id, p.product_name
	ORDER BY p.product_name;
```

| product_id | product_name | total_sales |
|------------|--------------|-------------|
| e83aa3 | Black Straight Jeans - Womens |	121152 |
| 2a2353 | Blue Polo Shirt - Mens | 217683 |
| e31d39 | Cream Relaxed Jeans - Womens |	37070 |
| 9ec847 | Grey Fashion Jacket - Womens | 209304 |
| 72f5d4 | Indigo Rain Jacket - Womens | 71383 |
| d5e9a6 | Khaki Suit Jacket - Womens | 86296 |
| c4a632 | Navy Oversized Jeans - Womens | 50128 |
| f084eb | Navy Solid Socks - Mens | 136512 |
| 2feb6b | Pink Fluro Polkadot Socks - Mens |	109330 |
| c8d436 | Teal Button Up Shirt - Mens |	36460 |
| b9a74d | White Striped Socks - Mens |	62135 |
| 5d267b | White Tee Shirt - Mens |	152000 |

What was the total discount amount for all products?

```sql
SELECT p.product_id,
       p.product_name,
       SUM(s.discount) AS total_discount
	FROM product_details AS p
	JOIN sales AS s ON p.product_id = s.prod_id
	GROUP BY p.product_id, p.product_name
	ORDER BY p.product_name; 
```

| product_id | product_name | total_discount |
|------------|--------------|----------------|
| e83aa3 | Black Straight Jeans - Womens |	15257 |
| 2a2353 | Blue Polo Shirt - Mens |	15553 |
| e31d39 | Cream Relaxed Jeans - Womens |	15065 |
| 9ec847 | Grey Fashion Jacket - Womens |	15500 |
| 72f5d4 | Indigo Rain Jacket - Womens |	15283 |
| d5e9a6 | Khaki Suit Jacket - Womens |	14669 |
| c4a632 | Navy Oversized Jeans - Womens |	15418 |
| f084eb | Navy Solid Socks - Mens |	15646 |
| 2feb6b | Pink Fluro Polkadot Socks - Mens |	14946 |
| c8d436 | Teal Button Up Shirt - Mens |	15003 |
| b9a74d | White Striped Socks - Mens |	14873 |
| 5d267b | White Tee Shirt - Mens |	15487 |

**Transaction Analysis**

How many unique transactions were there?

```sql
SELECT COUNT(DISTINCT(txn_id)) AS total_txn 
      FROM sales;
```
| total_txn |
|-----------|
| 2500 |

What is the average unique products purchased in each transaction?

```sql
WITH total_prod AS (SELECT COUNT(DISTINCT(prod_id)) AS total_prod,
                           txn_id
                       FROM sales
	                   GROUP BY txn_id)
	SELECT ROUND(AVG(total_prod)) AS AVG_prod
	    FROM total_prod;
```
| avg_prod |
|----------|
|      6 |

What are the 25th, 50th and 75th percentile values for the revenue per transaction?

```sql
WITH total_revenue AS (SELECT txn_id,
                              SUM(qty*price) as total_revenue
	                        FROM sales
	                        GROUP BY txn_id)
							
		SELECT PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY total_revenue) AS "25th Percentile",
               PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY total_revenue) AS "50th Percentile",
               PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY total_revenue) AS "75th Percentile"
			FROM total_revenue;
```

| 25th Percentile | 50th Percentile | 75th Percentile |
|-----------------|-----------------|-----------------|
| 375.75 |	509.5 |	647 |

What is the average discount value per transaction?

```sql
WITH total_discount AS (SELECT txn_id,
                               SUM(discount) as total_discount
                           FROM sales
	                       GROUP BY txn_id)
		SELECT ROUND(AVG(total_discount)) as Avg_discount
		      FROM total_discount;
```
		       
| avg_discount |
|--------------|
|    73  |

What is the percentage split of all transactions for members vs non-members?

```sql
SELECT CASE WHEN member = 't' THEN 'Member'
          ELSE 'Non-Member'
          END AS Members, 
       COUNT(DISTINCT txn_id) AS total_txn,
       ROUND(100.00 * COUNT(DISTINCT txn_id) / SUM(COUNT(DISTINCT txn_id)) OVER (), 2) AS txn_percent
   FROM sales
GROUP BY Members;
```

| members | total_txn | txn_percent |
|---------|-----------|-------------|
| Member |	1505 |	60.20 |
| Non-Member |	995 |	39.80 |

What is the average revenue for member transactions and non-member transactions?

```sql
SELECT CASE WHEN member = 't' THEN 'Member'
                          ELSE 'Non-Member'
                          END AS Members, 
                ROUND(SUM(qty * price) / COUNT(DISTINCT txn_id), 2) AS avg_revenue_per_txn
       FROM sales
       GROUP BY Members;
```

| members |  avg_revenue_per_txn |
|---------|----------------------|
| Member |	516.00 |
| Non-Member |	515.00 |
	  
**Product Analysis**

What are the top 3 products by total revenue before discount?

```sql
SELECT s.prod_id,
       p.product_name,
	   SUM(s.qty*s.price) AS total_sales
	 FROM sales AS s
	 JOIN product_details AS p ON p.product_id = s.prod_id
	 GROUP BY s.prod_id, p.product_name
	 ORDER BY p.product_name DESC
	 LIMIT 3; 
```

| prod_id | product_name | total_sales |
|---------|--------------|-------------|
| 5d267b | White Tee Shirt - Mens |	152000 |
| b9a74d | White Striped Socks - Mens |	62135 |
| c8d436 | Teal Button Up Shirt - Mens |	36460 |

What is the total quantity, revenue and discount for each segment?

```sql
SELECT p.segment_id,
       SUM(s.qty) AS total_quantity,
	   SUM(s.qty*s.price) AS total_revenue,
	   SUM(s.discount) as total_discount
	  FROM sales AS s
	  JOIN product_details AS p ON s.prod_id = p.product_id
    GROUP BY p.segment_id;
```

| segment_id | total_quantity | total_revenue | total_discount |
|------------|----------------|---------------|----------------|
| 3 |	11349 |	208350 |	45740 |
| 5 |	11265 |	406143 |	46043 |
| 4 |	11385 |	366983 |	45452 |
| 6 |	11217 |	307977 |	45465 |

What is the top selling product for each segment?

```sql
WITH total_revenue AS (SELECT p.segment_id,
                              p.product_id,
                              p.product_name,
                              SUM(s.qty * s.price) AS total_revenue
                        FROM sales AS s
                        JOIN product_details AS p ON s.prod_id = p.product_id
                        GROUP BY p.segment_id, p.product_id, p.product_name),
					
      ranked_products AS (SELECT segment_id,
                                 product_id,
                                 product_name, 
                                 total_revenue,
                                 DENSE_RANK() OVER (PARTITION BY segment_id ORDER BY total_revenue DESC) AS rank
                              FROM total_revenue)
  SELECT segment_id,
         product_id,
         product_name,
         total_revenue
     FROM ranked_products
     WHERE rank = 1
     ORDER BY segment_id, rank;
```

| segment_id | product_id | product_name | total_revenue |
|------------|------------|--------------|---------------|
| 3	| e83aa3 | Black Straight Jeans - Womens |	121152 |
| 4	| 9ec847 | Grey Fashion Jacket - Womens |	209304 |
| 5	| 2a2353 | Blue Polo Shirt - Mens |	217683 |
| 6	| f084eb | Navy Solid Socks - Mens |	136512 |

What is the total quantity, revenue and discount for each category?

```sql
SELECT p.category_id,
       SUM(s.qty) AS total_quantity,
	   SUM(s.qty*s.price) AS total_revenue,
	   SUM(s.discount) as total_discount
	  FROM sales AS s
	  JOIN product_details AS p ON s.prod_id = p.product_id
    GROUP BY p.category_id
	ORDER BY category_id;
```
| category_id | total_quantity | total_revenue | total_discount |
|-------------|----------------|---------------|----------------|
| 1 |	22734 |	575333 |	91192 |
| 2 |	22482 |	714120 |	91508 |

What is the top selling product for each category?

```sql
With product_rank AS (SELECT p.category_id,
                             p.product_id,
					         p.product_name,
                             SUM(s.qty) AS total_quantity,
	                         SUM(s.qty*s.price) AS total_revenue,
	                         SUM(s.discount) as total_discount,
	                         ROW_NUMBER()OVER(PARTITION BY p.category_ID ORDER BY SUM(s.qty*s.price) DESC) AS rank
	                     FROM sales AS s
	                     JOIN product_details AS p ON s.prod_id = p.product_id
                         GROUP BY p.product_id, p.category_id, p.product_name)
	    SELECT category_id,
		       product_name,
			   total_revenue
		   FROM product_rank
		   WHERE rank = 1
		   GROUP BY product_name, category_id, total_revenue
		   ORDER BY category_id;
```

| category_id | product_name | total_revenue |
|-------------|--------------|---------------|
| 1 |	Grey Fashion Jacket - Womens |	209304 |
| 2 |	Blue Polo Shirt - Mens |	217683 |
		   
What is the percentage split of revenue by product for each segment?

```sql
SELECT p.segment_id,
       s.prod_id,
       p.product_name,
       ROUND(100.00 * SUM(s.qty*s.price)/SUM(SUM(s.qty*s.price)) OVER (PARTITION BY p.segment_id), 2) AS product_percent
   FROM sales AS s
   JOIN product_details AS p ON s.prod_id = p.product_id 
GROUP BY p.segment_id, s.prod_id, p.product_name
ORDER BY segment_id;
```

| segment_id | prod_id | product_name | product_percent |
|------------|---------|--------------|-----------------|
| 3	| e83aa3 |	Black Straight Jeans - Womens |	58.15 |
| 3	| c4a632 |	Navy Oversized Jeans - Womens |	24.06 |
| 3	| e31d39 | 	Cream Relaxed Jeans - Womens |	17.79 |
| 4	| 9ec847 |	Grey Fashion Jacket - Womens |	57.03 |
| 4	| 72f5d4 |	Indigo Rain Jacket - Womens |	19.45 |
| 4	| d5e9a6 |	Khaki Suit Jacket - Womens |	23.51 |
| 5	| c8d436 |	Teal Button Up Shirt - Mens |	8.98 |
| 5	| 5d267b |	White Tee Shirt - Mens |	37.43 |
| 5	| 2a2353 |	Blue Polo Shirt - Mens | 53.60 |
| 6	| 2feb6b |	Pink Fluro Polkadot Socks - Mens |	35.50 |
| 6	| f084eb |	Navy Solid Socks - Mens |	44.33 |
| 6	| b9a74d |	White Striped Socks - Mens |	20.18 | 

What is the percentage split of revenue by segment for each category?

```sql
SELECT p.category_id,
       p.segment_id,
       ROUND(100.00 * SUM(s.qty*s.price)/SUM(SUM(s.qty*s.price)) OVER (PARTITION BY p.category_id), 2) AS segment_percent
   FROM sales AS s
   JOIN product_details AS p ON s.prod_id = p.product_id 
GROUP BY p.category_id, p.segment_id
ORDER BY category_id;
```

| category_id |	segment_id |	segment_percent |
|-------------|------------|--------------------|
| 1 |	3 |	36.21 |
| 1 |	4 |	63.79 |
| 2 |	6 |	43.13 |
| 2 |	5 |	56.87 |

What is the percentage split of total revenue by category?

```sql
SELECT p.category_id,
       ROUND(100.00 * SUM(s.qty*s.price)/SUM(SUM(s.qty*s.price)) OVER (), 2) AS category_percent
   FROM sales AS s
   JOIN product_details AS p ON s.prod_id = p.product_id 
GROUP BY p.category_id
ORDER BY category_id;
```

| category_id | category_percent |
|-------------|------------------|
| 1 |	44.62 |
| 2 |	55.38 |

What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)

```sql
WITH product_in_txn AS ( SELECT s.txn_id,
                                s.prod_id,
                                p.product_name,
                                1 AS is_product_in_txn
                           FROM sales AS s
	                   JOIN product_details AS p ON s.prod_id = p.product_id
                           GROUP BY s.txn_id, s.prod_id, p.product_name)
SELECT prod_id,
       product_name,
       ROUND(100.00 * COUNT(DISTINCT txn_id) / (SELECT COUNT(DISTINCT txn_id) FROM sales),2) AS txn_penetration_percentage
   FROM product_in_txn
   GROUP BY prod_id, product_name
   ORDER BY txn_penetration_percentage DESC;
```

| prod_id | product_name  | txn_penetration_percentage |
|---------|---------------|----------------------------|
| f084eb	| Navy Solid Socks - Mens |	51.24 |
| 9ec847	| Grey Fashion Jacket - Womens |	51.00 |
| c4a632	| Navy Oversized Jeans - Womens |	50.96 |
| 2a2353	| Blue Polo Shirt - Mens | 	50.72 |
| 5d267b	| White Tee Shirt - Mens | 	50.72 |
| 2feb6b	| Pink Fluro Polkadot Socks - Mens |	50.32 |
| 72f5d4	| Indigo Rain Jacket - Womens |	50.00 |
| d5e9a6	| Khaki Suit Jacket - Womens |	49.88 |
| e83aa3	| Black Straight Jeans - Womens |	49.84 |
| e31d39	| Cream Relaxed Jeans - Womens |	49.72 |
| b9a74d	| White Striped Socks - Mens |	49.72 |
| c8d436	| Teal Button Up Shirt - Mens |	49.68 |

What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?

```sql
WITH product_combinations AS (SELECT s1.prod_id AS prod1,
                                     s2.prod_id AS prod2,
                                     s3.prod_id AS prod3,
                                     s1.txn_id
                                  FROM sales AS s1
                                  JOIN sales s2 ON s1.txn_id = s2.txn_id 
                                       AND s2.prod_id > s1.prod_id
                                  JOIN sales s3 ON s1.txn_id = s3.txn_id 
                                       AND s3.prod_id > s2.prod_id)
          SELECT prod1, 
                 prod2, 
                 prod3,
                COUNT(*) AS combination_count
            FROM product_combinations
            GROUP BY prod1, prod2, prod3
            ORDER BY combination_count DESC
            LIMIT 1;
```

| prod1 | prod2 | prod3 | combination_count |
|-------|-------|-------|-------------------|
| 5d267b |	9ec847 |	c8d436 |	352 |



