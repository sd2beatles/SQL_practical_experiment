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
### Chapter 2 Explore Data Analysis 

#### 2.1 Deliver Time

Many surveys suggest that among many factors responsible for business success, shortening the lead time between the placement of order and  shipment has become a critical one to determine customer satisfaction.  Therefore, it is worthwhile to look into what features affect the latency and to check if  any improvements are needed. 

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


The average day to prepare and send products is 3.96 days from the confirmation of order and except for peak seasons, most monthly figures are below the average. It seems to be inevitable that in some months of year the delivery lead time is higher than the average since volumes of package rise above the point where the company could handle at full capacity. 




#### 2.2 Shppping method and Time taken to ship 


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
![image](https://user-images.githubusercontent.com/53164959/67260264-76cf0180-f4d5-11e9-8820-c5107bc133df.png)

One good news is that for customers who placed expedited orders the requested products were shipped on the day of order confirmed. Apart from the special delivery method, the plot lines show that the variablity of lead time is not so huge that  the magnitude itself is not so huge that the company may manage the latency under strict regularisation or in an effective manner.

#### 2.3 Financial Performance 

Let's look at the financial performance for the last three years. 

```sql
WITH raw_superstore AS(
    --refer to the previous section ),
           year_monthly_figure AS(
           SELECT year,
                  month,
                  SUM(sales)   AS sales,
                  SUM(profit)  AS profit,
                  SUM(SUM(sales)) OVER(ORDER BY year,month ROWS UNBOUNDED PRECEDING) AS agg_sales,
                  SUM(SUM(profit)) OVER(ORDER BY year,month ROWS UNBOUNDED PRECEDING)AS agg_profit
                  FROM raw_superstore
                  GROUP BY year,month)
           SELECT CONCAT(year,'-',month) AS year_month,
                  sales,
                  profit,
                  agg_sales,
                  agg_profit FROM year_monthly_figure;
          
  ```
  
  ```python
result2=pd.read_csv(r'~\agg_sales_profit.csv')
sns.set()
fig,ax1=plt.subplots(figsize=(14,5))
ax2=ax1.twinx()
sns.barplot(x=result2.year_month,y=result2.sales,ax=ax1)
sns.lineplot(x=result2.year_month,y=result2.agg_sales,ax=ax2,color='R',label='Aggregated_Sales')
sns.lineplot(x=result2.year_month,y=result2.agg_profit,ax=ax2,color='B',label='Aggregate_profit')
ax1.set_xticklabels(result2.year_month,rotation=90)
ax.set_title('BarCharts for Monthly revenue and profit From 2014 to 2017',fontsize=16)
```

![image](https://user-images.githubusercontent.com/53164959/67847661-91911e00-fb46-11e9-8fb6-ffcf7e5cb49d.png)

                 
           




From the charts, three characteristics especially attract our attention. 
First, the sales were growing over the time interval. The highest amount of revenue earned in 2017 reached almost 120 thousand, about a 50 % increase compared to in 2014. 

Second, we are aware that there is seasonlaity in some of the months a year. There is a dramatic increase in sales from October to November, followed by a massive fall from December to January of next year. The seasonal trend is highly anticipated in that most companies begin to promote thier products and service massively and actively at the begin of November, trying thier conumser feel that thier products are inviting and attractive. This is maninly beacuse of _black Friday_ on which many consumers become enamored with business strategies, leading to a boost in sales of companies.   The expenditure usually continues up to the end of December. The fall in sales from November to December of every year is  predictable ; since many people decided to buy a bulk of products in advance to prepare for Christmas and New Year, consumers tend to be tight on budget for expenditure in December.  On the closer examination,  the magnitude of fall in values in the year of 2017 is, however,  _exceptionally larger than any other interval_.

Third, The company did not enjoy the impressive moment in the year of 2014; as indicated by a blue line, for the first 10 months,  the aggregated amount did not show any glimpse of a rise but kept staying at 0. This reminds us that the company did not retain any gross profit at the begin of the time interval. However, the aggregated sum started to arise right after November.  



