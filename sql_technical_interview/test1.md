### Q1

![image](https://user-images.githubusercontent.com/53164959/73118811-983ffd80-3f9c-11ea-8d3a-dbab7bc5566b.png)


```sql

WITH stats AS(
SELECT city,
       LENGTH(city) AS len,
       MAX(LENGTH(city)) OVER() AS maxlen,
       MIN(LENGTH(city)) OVER() AS minlen
       FROM station)

# To get only one min and max, we need to place min over city, which helps us to get the first alphabetical value when 2 or more values 
   are avaialbe.
   
SELECT min(city) AS city,
       len
       FROM stats
       GROUP BY len
```


### Q2
![image](https://user-images.githubusercontent.com/53164959/73119341-c37a1b00-3fa3-11ea-87a0-6b67785e3717.png)

```sql
WITH modified_data AS(
   SELECT distinct CITY,
          SUBSTRING(CITY,LENGTH(city),LENGTH(city)) AS lw,
          SUBSTRING(CITY,1,1) AS fw
   FROM station)
   SELECT city
          FROM modified_data
          WHERE lw  in ('a','e','i','o','u')
            and fw in('a','e','i','o','u');
```


