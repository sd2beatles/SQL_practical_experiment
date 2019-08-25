[https://www.kaggle.com/aungpyaeap/supermarket-sales]
![image](https://user-images.githubusercontent.com/53164959/63651124-a5a65100-c78c-11e9-9ae9-7b570eb30661.png)

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
