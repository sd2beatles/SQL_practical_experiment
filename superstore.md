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
```
### Chapter 2 Explore Data

#### 2.1 Deliver Time

Many surveys suggest that among many factors responsible for business success, shortening delivery times has become a critical one to determine customer satisfaction.  Therefore, it is worthwhile to look into what features affect the delivery time and improvements over the current system. 

##### 2.1.1 Monthly Deliver_Time
```sql
WITH raw_superstore AS(
    -- replicate the original data and add dlivery_time(days to proceed and finish deliverying products from order date)
    -- and the cost of sale
    SELECT order_id,
           customer_id,
           product_id,
           order_date,
           ship_date,
           ship_mode,
           SUBSTRING(order_date,1,4) AS year,
           SUBSTRING(order_date,6,2) AS month,
           CAST(ship_date AS DATE)-CAST(order_date AS DATE) AS deliver_time,
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
           FROM superstore)
      SELECT 
             DISTINCT month,
             ROUND(CAST(AVG(deliver_time) OVER(PARTITION BY month) AS NUMERIC),2) AS monthly_average,
             ROUND(CAST(AVG(deliver_time) OVER() AS NUMERIC),2) AS toal_average 
             FROM raw_superstore
             ORDER BY month; 
```

![image](https://user-images.githubusercontent.com/53164959/67255562-9823f380-f4bd-11e9-8aee-a3763c3a65e2.png)
![image](https://user-images.githubusercontent.com/53164959/67256016-edf99b00-f4bf-11e9-8d17-77929893038f.png)


The average day to prepare and send products is 3.96 days and except for peak seasons, most monthly figures are below the average. It seems to be inevitable to witness that the delivery lead time is higher than the average since volumes of package rise above the point where the company could handle at full capacity. 





```sql
WITH raw_superstore AS(
    --refer to the previous section 
    ),
 avg_by_mode AS(
      SELECT Distinct ship_mode,
             month,
             ROUND(CAST(AVG(deliver_time) OVER(PARTITION BY month,ship_mode) AS NUMERIC),2) AS avg
             FROM raw_superstore
             ORDER BY month,avg)
      SELECT ship_mode,
             month,
             CASE WHEN avg<1 THEN 0 ELSE AVG END AS avg_deliver 
             FROM avg_by_mode;
```
![image](https://user-images.githubusercontent.com/53164959/67253995-9d7d4000-f4b5-11e9-8d48-0fc5434fb6bc.png)
