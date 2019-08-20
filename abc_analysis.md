ABC ANALYSIS: Active Inventory FROM Kaggle 
===========================================

### 1.What is an ABC analysis?

ABC analysis is concerned with dividing the complex and broad system into sub-groups and rank them in descending order based on some factors.  We will use the public data, inventory held at a company, to divide the inventory into three segments. This commonly held practice will allow different management techniques to be adopted to different 
groups of the inventory in order to boost revenue and reduce the cost 

Each category A, B, and C is determined based on two factors, one of which is the total quality of items and the other is the total value of the items in the warehouse. 

### 2.ABC analysis steps

(1) Compute the total spending for all three units by multiplying 
     the unit cost with the annual unit demand

(2) Arrange the inventory in decreasing order of annual spending

(3) Divide the inventory into classes. Here, we set up  arbitrary numbers 
   of quantity percentage for the grouping. 

  -  A must be in the range between 0 and 60%.
  -  B must be somewhere between 60% and 85%.
   - C must be in the range greater than 85%.

(4) Summarize all the data in a table to include    
   - ranks
   - total_quantity
   - total_cost
   - quantity_percentage
   - cost_percentage 

(5) Analyze the classes and make a proper judgment.

### 3. SQL codes 



Some irrelevant information included in the original data should be filtered out for analytical purposes. We will leave three fields including  sku_number,priceregm, and itemcount.

![image](https://user-images.githubusercontent.com/53164959/63311281-8a74b500-c338-11e9-9720-9ddb7048ee23.png)


Create a subquery to contain cost per unit,quantity demand, total expenditure and the proportion of each cost to the total sum.  All this information is referenced to cum_rate_cost for calculating the cumulative sum of cost percentage. Up to this point, the order does not matter at all since our attention is placed on the computation of cumulative cost and its percentage. 

```sql

SELECT * FROM sales;

WITH statics_cost AS(
     SELECT sku_number,
            pricereg,
            itemcount,
            ROUND(CAST(pricereg*itemcount AS NUMERIC),2) AS cost,
            100*(pricereg*itemcount)/SUM(pricereg*itemcount) OVER()  AS composition_rate -- cumu
            FROM sales),
      cum_rate_cost AS(
      SELECT sku_number,
             pricereg,
             itemcount,
             cost,
             composition_rate,
             SUM(composition_rate) OVER(ORDER BY composition_rate rows BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cumulative_rate
             FROM statics_cost)
```
             
             
             
             
             
       ,abc_analysis as(SELECT sku_number,
              pricereg,
              itemcount,
              cost,
              composition_rate AS composition_rate,
              cumulative_rate,
              CASE WHEN cumulative_rate BETWEEN 0 AND 60 THEN 'A'
                   WHEN cumulative_rate BETWEEN 60 AND 85 THEN 'B'
                   WHEN cumulative_rate BETWEEN 85 AND 100.1 THEN 'C'
                   END AS abc_rank
              FROM cum_rate_cost 
              ORDER BY cumulative_rate)
         ,test_result AS(SELECT abc_rank AS class,
              SUM(sku_number) AS total_sku,
              SUM(cost) AS total_cost,
              ROUND(CAST(100*SUM(sku_number)/SUM(SUM(sku_number)) OVER() AS NUMERIC),0)  AS quantity_percentage,
              ROUND(CAST(100*SUM(cost)/SUM(SUM(cost)) OVER() AS NUMERIC),0) AS cost_percentage
              FROM abc_analysis 
              GROUP BY abc_rank
                   UNION ALL SELECT 'Total' AS abc_rank, SUM(sku_number) AS total_sku,SUM(cost) AS total_cost,
                                     100 AS quantity_percentage, 100 AS cost_percentage
                   FROM abc_analysis)
           SELECT class,total_sku,total_cost,CONCAT(quantity_percentage||'%') AS quantity_percentage,CONCAT(cost_percentage||'%') AS                       cost_percentage
                  FROM test_result
                  ORDER BY class;
                  
```
