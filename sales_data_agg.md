![image](https://user-images.githubusercontent.com/53164959/63658689-943e6280-c7e7-11e9-9f62-90d411738297.png)



```sql
WITH daily_sales AS(
     SELECT date,
            product_line,
            SUM(toal) AS amount
            FROM sales_table
            GROUP BY date,product_line),
      monthly_sales AS(
      SELECT SUBSTRING(date,1,4) AS year,
             CAST(SUBSTRING(date,6,2) AS INT) AS month,
             product_line,
             SUM(amount) AS amount
             FROM daily_sales
             GROUP BY year,month,product_line)
     ,cal_index AS(
      SELECT year,
             month,
             product_line,
             FIRST_VALUE(amount) OVER(PARTITION BY product_line ORDER BY year,month,amount ROWS UNBOUNDED PRECEDING) 
             AS base,
             100*amount/FIRST_VALUE(amount) OVER(PARTITION BY product_line ORDER BY year,month,amount ROWS UNBOUNDED PRECEDING)
             AS rate
             FROM monthly_sales)
       SELECT CONCAT(year,'-',month) AS year_month,
              product_line,
              base,
              rate 
              FROM cal_index
              WHERE month<5;
```


```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib as mpl
import seaborn as sns

data=pd.read_csv(r"\test.csv")
data["rate"]=round(data["rate"]
```
![image](https://user-images.githubusercontent.com/53164959/63651180-22d1c600-c78d-11e9-943b-7f25671a274d.png)


```python
fig, ax = plt.subplots(figsize=(10, 6))
test.plot(ax=ax,kind='line',marker='.')
plt.legend(loc="best")
plt.ylabel("Sales rate(%)",fontsize=15)
plt.xlabel("Month",fontsize=14)
plt.title("Fan Chart: Sales in 2019",fontsize=15)
```
![image](https://user-images.githubusercontent.com/53164959/63651252-cd49e900-c78d-11e9-88bb-ae87b9a2486a.png)


```sql

WITH stat AS(
      SELECT 
        product_line,
        MAX(revenue)+1 AS max_revenue,
        MIN(revenue) AS min_revenue,
        MAX(revenue)+1-MIN(revenue) AS range_revenue
        FROM salestb
        GROUP BY product_line),
     sales_by_product AS(
     SELECT  s.product_line,
             t.revenue,
             s.max_revenue,
             s.min_revenue,
             s.range_revenue,
             10 AS bucket_num
             FROM stat AS s
             CROSS JOIN salestb AS t
                   where s.product_line=t.product_line),
     sales_with_buckets AS(
     SELECT product_line, 
            revenue,
            min_revenue,
            revenue-min_revenue AS diff_revenue,
            range_revenue/bucket_num AS bucket_range,
            FLOOR((revenue-min_revenue)/(range_revenue/bucket_num))+1 AS bucket
            FROM  sales_by_product)
     SELECT product_line,
            bucket,
            min_revenue+bucket_range*(bucket-1) AS lower_limit,
            min_revenue+bucket_range*bucket AS upper_limit,
            COUNT(revenue) OVER(PARTITION BY product_line,bucket),
            SUM(revenue) OVER(PARTITION BY product_line,bucket)
            FROM sales_with_buckets
            ORDER BY product_line,bucket;
 ```
 
