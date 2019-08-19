CASE STUDY : DEATH AND BIRTH OF NEWZEALANDER
                  BETWEEN 2000 AND 2012
================================================================

### 1. Introduction

You need to download the data, birth_death from the website below. 

[https://timeseries.weebly.com/data-sets.html]

This CSV-file consists of four different columns, and as the names imply, this gives information on fertility and mortality 
each grounded by sex. The period in which data were recorded ranges from 2000 to 2012 and each year is sub-divided into four quarters.  

![image](https://user-images.githubusercontent.com/53164959/63254990-b26c0600-c2af-11e9-85c3-7bd0e9b06e6b.png)


## 2.Data Handling 



Given that one of your immediate supervisors is curious about the total number of population separated by genders on a yearly base. 
He also appreciates your presenting total counts of male and female, respectively along with the total number of males and females. 
That is,under the separate field, there should be rows comprised of birth, death, and total number with each split by sex as well
as the grand total row from year to year. 


#### 2.1.1 Extraction of year 

The time we are interested in is yearly based rather than quarterly. Therefore, by using SUBSTRING function, we extract year 
from  'quarter' in the original data set.With the use of subquery (ie with as clause), we give it a name birth_death_temp 
to be referenced in several places within the main query. 

```sql
WITH birth_death_temp AS(
 SELECT SUBSTRING(quarter,1,4) AS year,
          SUBSTRING(quarter,5,2) AS quarter,
          male_birth AS mb,
          female_birth AS fb,
          male_death AS md,
          female_death as fd
          FROM birth_deatth) 
 ```
 
#### 2.1.2 Total Sum by gender on a yearly basis
 
The next step is to compute the following statics ;

 * total_male : total number  of males in a given year
 
 * total_female: total number of males in a given year
 
 * yearly_mb: total number of males who were born in a given year
 
 * yearly_fb: total number of females who were born in a given year
 
 * year_fd: total number of males who were dead in a given year 
 
 * year_fd: total number of females who were dead in a given year 
 

Since all the measures are a yearly basis, we need to implement GROUP BY function with the column of year. All the figures 
are determinded based on the data inferenced from birth_death_temp. 


 ```sql
  WITH birth_death_temp AS(
   SELECT SUBSTRING(quarter,1,4) AS year,
          SUBSTRING(quarter,5,2) AS quarter,
          male_birth AS mb,
          female_birth AS fb,
          male_death AS md,
          female_death as fd
          FROM birth_death),
      year_sum AS(
      SELECT year,
             -- in case where SUM(mb) is null and replace it with 0  
             COALESCE(SUM(mb),0)+COALESCE(SUM(md),0) AS total_male, 
             COALESCE(SUM(fb),0)+COALESCE(SUM(fd),0) AS total_female, 
             SUM(mb) AS yearly_mb,
             SUM(fb) AS yearly_fb,
             SUM(md) AS yearly_md,
             SUM(fd) AS yearly_fd
             FROM birth_death_temp 
             GROUP BY year
             ORDER BY year)
 ```
 
 #### 2.1.3 Conversion of Columns into Rows

 As instructed by our supervisor, we need to create a separate column that contains seven unique values, six of which are inferenced
 from  year_sum. irst, generate a serial number from 1 to 6. The number six represents the total number of columns except for year.This 
 can be realized by using UNION ALL function - combine one or more select statements. We name it idx and CROSS JOIN  year_sum. 
 After a CROSS JOINING with the mentioned sub_query,  using CASE WHEN condition to create two separate fields, category and number 
 in which corresponding value is stored. 
 
 ```sql
 WITH birth_death_temp AS(
   SELECT SUBSTRING(quarter,1,4) AS year,
          SUBSTRING(quarter,5,2) AS quarter,
          male_birth AS mb,
          female_birth AS fb,
          male_death AS md,
          female_death as fd
          FROM birth_death),
      year_sum AS(
      SELECT year,
             COALESCE(SUM(mb),0)+COALESCE(SUM(md),0) AS total_male,
             COALESCE(SUM(fb),0)+COALESCE(SUM(fd),0) AS total_female,
             SUM(mb) AS yearly_mb,
             SUM(fb) AS yearly_fb,
             SUM(md) AS yearly_md,
             SUM(fd) AS yearly_fd
             FROM birth_death_temp 
             GROUP BY year
             ORDER BY year),
       division_general AS(
       SELECT y.year,
              CASE WHEN p.idx=1 THEN 'total_male'
                   WHEN p.idx=2 THEN 'total_female'
                   WHEN p.idx=3 THEN 'male_birth'
                   WHEN p.idx=4 THEN 'female_brith'
                   WHEN p.idx=5 THEN 'male_death'
                   WHEN p.idx=6 THEN 'female_death'
                   END AS category,
               CASE WHEN p.idx=1 THEN total_male
                    WHEN p.idx=2 THEN total_female
                    WHEN p.idx=3 THEN yearly_mb
                    WHEN p.idx=4 THEN yearly_fb
                    WHEN p.idx=5 THEN yearly_md
                    WHEN p.idx=6 THEN yearly_fd END AS number
              FROM year_sum AS y
              CROSS JOIN (SELECT 1 AS idx
                             UNION ALL SELECT 2 AS idx
                             UNION ALL SELECT 3 AS idx
                             UNION ALL SELECT 4 AS idx
                             UNION ALL SELECT 5 AS idx
                             UNION ALL SELECT 6 AS idx) AS p)
 
 ```
 
 #### 2.1.4 Subtotal And Grandtoal 
 
Additionally, we are required to insert more information on subtotal along with the grand total row.  We can do this by using 
the ROLLUP clause with specifying year and category. Then, we can prepare a desicre outcome your supervisor wants to see. 

```sql
WITH birth_death_temp AS(
   SELECT SUBSTRING(quarter,1,4) AS year,
          SUBSTRING(quarter,5,2) AS quarter,
          male_birth AS mb,
          female_birth AS fb,
          male_death AS md,
          female_death as fd
          FROM birth_death),
      year_sum AS(
      SELECT year,
             COALESCE(SUM(mb),0)+COALESCE(SUM(md),0) AS total_male,
             COALESCE(SUM(fb),0)+COALESCE(SUM(fd),0) AS total_female,
             SUM(mb) AS yearly_mb,
             SUM(fb) AS yearly_fb,
             SUM(md) AS yearly_md,
             SUM(fd) AS yearly_fd
             FROM birth_death_temp 
             GROUP BY year
             ORDER BY year),
       division_general AS(
       SELECT y.year,
              CASE WHEN p.idx=1 THEN 'total_male'
                   WHEN p.idx=2 THEN 'total_female'
                   WHEN p.idx=3 THEN 'male_birth'
                   WHEN p.idx=4 THEN 'female_brith'
                   WHEN p.idx=5 THEN 'male_death'
                   WHEN p.idx=6 THEN 'female_death'
                   END AS category,
               CASE WHEN p.idx=1 THEN total_male
                    WHEN p.idx=2 THEN total_female
                    WHEN p.idx=3 THEN yearly_mb
                    WHEN p.idx=4 THEN yearly_fb
                    WHEN p.idx=5 THEN yearly_md
                    WHEN p.idx=6 THEN yearly_fd END AS number
              FROM year_sum AS y
              CROSS JOIN (SELECT 1 AS idx
                             UNION ALL SELECT 2 AS idx
                             UNION ALL SELECT 3 AS idx
                             UNION ALL SELECT 4 AS idx
                             UNION ALL SELECT 5 AS idx
                             UNION ALL SELECT 6 AS idx) AS p)
        SELECT 
           COALESCE(year,'all') AS year,
           COALESCE(category,'all') AS category,
           SUM(number) AS mount 
           FROM division_general
           GROUP BY ROLLUP(year,category) 
           ORDER BY year;
```

![image](https://user-images.githubusercontent.com/53164959/63261275-11d11280-c2be-11e9-84d7-7144bc44b3c7.png)

[table] The First 8 Entries of the Final Outcome 
 
  
![image](https://user-images.githubusercontent.com/53164959/63261198-ce76a400-c2bd-11e9-870a-66b28b077965.png)

[table] The Last 6 Entires of The Final Outcome 

_Note_
If you watn to get a last record of a table in Postgres, then put DESC next to the name of colum in ORDER BY caluse. 

```sql
   SELECT 
           COALESCE(year,'all') AS year,
           COALESCE(category,'all') AS category,
           SUM(number) AS mount 
           FROM division_general
           GROUP BY ROLLUP(year,category) 
           ORDER BY year DESC
           LIMIT 6;
 ```
