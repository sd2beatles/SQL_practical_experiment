### 2.3.2 Sales Revenue,profit,and cost of sales by Category and sub_category
![image](https://user-images.githubusercontent.com/53164959/68095287-79752200-feeb-11e9-86fc-6c7660441737.png)


#### 2.3.1 Sales Revnue By Category and Sub_category

```sql
WITH raw_superstore AS(
       -- reference to the previous section 
          ,sales_by_sub_category AS(
          SELECT  category,
                 sub_category,
                 SUM(sales) AS sales
                 FROM raw_superstore
                 GROUP BY category,sub_category
                 ORDER BY category,sub_category)
         ,sales_by_category AS(
         SELECT category, 
                SUM(sales) AS sales_by_category
                FROM raw_superstore
                GROUP BY category)
         ,composition_ratio AS(
         SELECT a.category,
                a.sub_category,
                a.sales,
                a.sales/b.sales_by_category*100 AS proportion
                FROM sales_by_sub_category as a
                LEFT OUTER JOIN sales_by_category as b
                ON a.category=b.category)
        ,cum_composition_ratio AS(
        SELECT category,
               sub_category,
               sales,
               ROUND(CAST(proportion AS NUMERIC),2) AS proportion,
               ROUND(CAST(SUM(proportion) OVER(PARTITION BY category ORDER BY sales DESC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS NUMERIC),2) AS cum_proportion
               FROM composition_ratio)
        SELECT category,
               sub_category,
               sales,
               CONCAT(proportion,'%') AS proportion,
               CONCAT(cum_proportion,'%') AS cum_proportion
                FROM cum_composition_ratio;
         
  ```                 

![image](https://user-images.githubusercontent.com/53164959/68558524-fb7dc180-047c-11ea-88cb-123c8efc3f9a.png)

[Table 1] Table for Sales Revenue

![image](https://user-images.githubusercontent.com/53164959/68561241-0d189680-0488-11ea-8a66-17f9adefb4f3.png)


[Chart 1] Bar Graph for Sales Revenue by category and sub-category


### 2.3.2 ABC Analysis 

```sql

WITH raw_superstore AS(
     -- refer to the previous section 
     )
   ,sum_amount AS(
       SELECT sub_category,
              SUM(sales) AS sales
              FROM raw_superstore
              GROUP BY sub_category)
       ,proportion AS(   
       SELECT sub_category,
              sales,
              ROUND(CAST(sales/SUM(sales) OVER()*100 AS NUMERIC),2) AS proportion,
              ROUND(CAST(SUM(sales) OVER(ORDER BY sales DESC)/SUM(sales) OVER()*100 AS NUMERIC),2) AS cum_proportion
              FROM sum_amount)
       SELECT sub_category,
              sales,
              concat(proportion,'%') AS proportion,
              concat(cum_proportion,'%') AS cum_proportion,
              CASE WHEN cum_proportion<=70 THEN 'A' 
                   WHEN cum_proportion<=80 THEN 'B'
                   ELSE 'C' END AS class
                   FROM proportion;
      

```

![image](https://user-images.githubusercontent.com/53164959/68561984-21aa5e00-048b-11ea-9a17-ff0fcca8a2d2.png)

[table 2] table for ABC analysis

![image](https://user-images.githubusercontent.com/53164959/68564585-64bcff00-0494-11ea-9f3b-6f38d2212c17.png)

[Chart 2] Overlapped Chart for ABC analysis




If you decide to adopt the multiple income statements,  the general practice in evaluating the financial performance of a company is to segregate expenses into two types( operating and non-operating expenses)  after subtracting revenue from the cost of goods sold. Because detailed information is not currently given, we **_consider all the expenses as a cost of goods sold_** even though the approach is not logical and reasonable.

```sql


WITH raw_superstore AS(
    --refer  to the previous section
    ),
   sales_by_category AS(
    SELECT category, 
           sub_category,
           SUM(sales) AS sales_amount,
           SUM(profit) AS profit_amount,
           SUM(cost_of_sales) AS cost_of_good_sold
           FROM raw_superstore
           GROUP BY category,sub_category
           ORDER BY category),
     sales_by_all AS(
     SELECT category,
    CAST('all' AS VARCHAR) AS sub_category,
     SUM(sales) AS sales_amount,
     SUM(profit) AS profit_amount,
     SUM(cost_of_sales) AS cost_of_good_sold
     FROM raw_superstore
     GROUP BY category)
     ,total_sale AS(
     SELECT CAST('all' AS VARCHAR) AS category,
            CAST('all' AS VARCHAR) AS sub_category,
            SUM(sales) AS sales_amount,
            SUM(profit) AS profit_amount,
            SUM(cost_of_sales) AS cost_of_good_sold
            FROM raw_superstore)
    SELECT category,sub_category,sales_amount,profit_amount,cost_of_good_sold FROM sales_by_category
        UNION ALL SELECT category,sub_category,sales_amount,profit_amount,cost_of_good_sold  FROM sales_by_all
        UNION ALL SELECT category,sub_category,sales_amount,profit_amount,cost_of_good_sold  FROM total_sale;
```

#### 2.3.3 Fan Charts

From the few charts we have gone through, we come to realize that the fluctuation of profit is quite higher than we would expect it to be. Therefore, we should step further to draw another chart for the movement of the profitability of each sector. 

In this section, We will use a visual tool called a fan chart joining a line graph for observed the past point with the very first date fixed as the base period. Denote our base value as 100 % and see how much the rate of each record changes in amount compared to that of the base period.

On top of that, we will see the growth of profits across three separated intervals.   

```sql
WITH aggregated_profit AS(
    SELECT SUBSTRING(order_date,1,7) AS date,
           category,
           SUM(profit) AS total_profit
           FROM superstore
           GROUP BY SUBSTRING(order_date,1,7),category)
     SELECT date,
            category,
            total_profit,
            FIRST_Value(total_profit) OVER(PARTITION BY category ORDER BY date,category ROWS UNBOUNDED PRECEDING) AS base_amount,
            total_profit/FIRST_VALUE(total_profit) OVER(PARTITION BY category ORDER BY date,category ROWS UNBOUNDED PRECEDING)*100 
            AS rate
            FROM aggregated_profit 
```


