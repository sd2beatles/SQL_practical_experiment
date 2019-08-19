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

### 2.1 Subtotal of Each Year and Grand Total Row

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
 
 - 
 
 
 
 
 - ROLLUP approach 
 
 
 ```sql
 
 
  
