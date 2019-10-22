### Chapter 1 Load Data

We can download the sample data with 9994 rows and 20 columns. Even though it is a subset sample of the whole, I deceid to use it for 
practice purpose only. You can visit the below website for more infomration.

[https://community.tableau.com/docs/DOC-1236]

Create a datbase named supterstore with proper names and types assigned. 
```sql
DROP TABLE IF EXISTS superstore;
CREATE TABLE public.superstore
(
  order_id character varying,
  order_date character varying,
  ship_date character varying,
  ship_mode character varying,
  customer_id character varying,
  customer_name character varying,
  segement character varying,
  country character varying,
  city character varying,
  state character varying,
  postal_code integer,
  region character varying,
  product_id character varying,
  category character varying,
  sub_category character varying,
  product_name character varying,
  sales real,
  quantity integer,
  discount real,
  profit real
);

### Chapter 2 Explore Data

#### 2.1 Deliver Time

Many surveys suggest that among many factors responsible for business success, shortening delivery times has become a critical one to determine customer satisfaction.  Therefore, it is worthwhile to look into what features affect the delivery time and improvements over the current system. 


```sql
WITH raw_superstore AS(
    -- replicate the original data and add the cost of sale 
    SELECT order_id,
           customer_id,
           product_id,
           order_date,
           ship_date,
           ship_mode,
           segement,
           country,
           city,
           state,
           region,
           category,
           sub_category,
           sales,
           quantity,
           profit,
           (profit-sales*quantity) AS cost_sales
           FROM superstore),
    delivery_time AS(
    SELECT 
    order_id,
    customer_id,
    product_id,
    order_date,
    ship_date,
    ship_mode,
    CAST(ship_date AS DATE)-CAST(order_date AS DATE) AS delivery_time
    FROM raw_superstore),
    delivery_shipping AS(
    SELECT DISTINCT ship_mode,
           SUBSTRING(order_date,6,2) AS month,
           CASE WHEN ROUND(CAST(AVG(delivery_time) AS NUMERIC),2) < 1 THEN 0
                WHEN ROUND(CAST(AVG(delivery_time) AS NUMERIC),2)>=1 THEN ROUND(CAST(AVG(delivery_time) AS NUMERIC),2) END AS      
                average_time
           FROM delivery_time 
           GROUP BY SUBSTRING(order_date,6,2),ship_mode)
    SELECT * FROM delivery_shipping 
             ORDER BY month,average_time;
   
![image](https://user-images.githubusercontent.com/53164959/67253995-9d7d4000-f4b5-11e9-8d48-0fc5434fb6bc.png)
