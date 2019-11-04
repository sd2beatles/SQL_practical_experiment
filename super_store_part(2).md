### 2.3.2 Sales Revenue,profit,and cost of sales by Category and sub_category

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
![image](https://user-images.githubusercontent.com/53164959/68094619-afafa300-fee5-11e9-895f-3f50d7552cf1.png)
