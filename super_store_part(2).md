### 2.3.2 Sales Revenue,profit,and cost of sales by Category and sub_category
![image](https://user-images.githubusercontent.com/53164959/68095287-79752200-feeb-11e9-86fc-6c7660441737.png)

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
![image](https://user-images.githubusercontent.com/53164959/68094619-afafa300-fee5-11e9-895f-3f50d7552cf1.png)

[table] A Table for Finanacial Measures

![image](https://user-images.githubusercontent.com/53164959/68096022-71b87c00-fef1-11e9-9039-2284a11bfc51.png)

[chart 1] A boxplot for Sale's Revenue


![image](https://user-images.githubusercontent.com/53164959/68095938-c27ba500-fef0-11e9-8d07-4bdcf6d5a8e6.png)

[Chart 2] A Boxtplot for Cost of Good Sold by Product Category

![image](https://user-images.githubusercontent.com/53164959/68095834-04f0b200-fef0-11e9-9a47-d2dca185bf11.png)

[Chart] A Boxtplot for Gross profit by Product Category



