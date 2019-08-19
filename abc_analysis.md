```sql

WITH statics_cost AS(
     SELECT sku_number,
            pricereg,
            itemcount,
            ROUND(CAST(pricereg*itemcount AS NUMERIC),2) AS cost,
            (100*pricereg*itemcount)/SUM(pricereg*itemcount) OVER()  AS composition_rate 
            FROM sales),
      cum_rate_cost AS(
      SELECT sku_number,
             pricereg,
             itemcount,
             cost,
             composition_rate,
             SUM(composition_rate) OVER(ORDER BY composition_rate rows BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cumulative_rate
             FROM statics_cost)
       ,abc_analysis as(SELECT sku_number,
              pricereg,
              itemcount,
              composition_rate AS composition_rate,
              cumulative_rate,
              CASE WHEN cumulative_rate BETWEEN 0 AND 60 THEN 'A'
                   WHEN cumulative_rate BETWEEN 60 AND 85 THEN 'B'
                   WHEN cumulative_rate BETWEEN 85 AND 100.1 THEN 'C'
                   END AS abc_rank
              FROM cum_rate_cost 
              ORDER BY cumulative_rate)
           SELECT * FROM abc_analysis ;

   ```
