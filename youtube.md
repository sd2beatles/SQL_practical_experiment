```sql

WITH fourteen_days_interval(index_name,begin_date,end_date) AS(
      VALUES('14 day rentation',8,14)),
    feature_measures AS(
    SELECT y.video_id,
           y.view ,
           y.likes ,
           y.dislikes,
           y.comment_count AS comments,
           CAST(SUBSTRING(y.publish_time,1,10) AS DATE) AS publish_time,
           CAST(y.trending_date AS DATE) AS action_date,
           MAX(CAST(y.trending_date AS DATE)) OVER() AS latest_date,
           CAST(SUBSTRING(y.publish_time,1,10) AS DATE)+'7 day'::interval AS one_week_interval,
           CAST(SUBSTRING(y.publish_time,1,10) AS DATE)+'1 day'::interval*f.begin_date AS index_begin_date,
           CAST(SUBSTRING(y.publish_time,1,10) AS DATE)+'1 day'::interval*f.end_date AS index_end_date 
           FROM youtube AS y,fourteen_days_interval AS f)
     ,action_log_with_index_dates AS(
      SELECT f.video_id,
             f.publish_time,
             a.index_name,
             SIGN(SUM(CASE WHEN f.action_date<=f.latest_date THEN 
                  CASE WHEN f.action_date BETWEEN index_begin_date AND index_end_date  THEN 1 ELSE 0 END END)) AS index_date_action
             FROM feature_measures AS f,fourteen_days_interval AS a
             GROUP BY video_id,publish_time,index_name)

     ,feature_limited AS(
         SELECT video_id,
                MAX(view) AS views,
                MAX(likes) AS likes,
                MAX(dislikes) AS dislikes,
                MAX(comments) AS comments
                FROM feature_measures
                WHERE action_date<latest_date and action_date=one_week_interval
                GROUP BY video_id)
     ,one_week_statics AS(
     SELECT    percentile_cont(0) within group ( order by views ) AS mini_views,
                     percentile_cont(0.5) within group ( order by views ) AS median_views,
                     percentile_cont(0) within group ( order by likes ) AS mini_likes,
                     percentile_cont(0.5) within group ( order by likes ) AS median_likes,
                     percentile_cont(0) within group ( order by dislikes ) AS mini_dislikes,
                     percentile_cont(0.5) within group ( order by dislikes) AS median_dislikes,
                     percentile_cont(0) within group ( order by comments ) AS mini_comments,
                     percentile_cont(0.5) within group ( order by comments ) AS median_comments
            FROM feature_limited
            )
        ,interval_statics AS(SELECT mini_views,
               median_views,
              FLOOR((median_views-mini_views)/5) AS views_interval,
              mini_likes,
              median_likes,
              FLOOR((median_likes-mini_likes)/5) AS likes_interval,
              mini_dislikes,
              median_dislikes,
              FLOOR((median_likes-mini_likes)/5) AS dislikes_interval,
              mini_comments,
              median_comments,
              FLOOR((median_comments-mini_comments)/5) AS comments_interval,
              GENERATE_SERIES(1,5) AS bucket
              FROM one_week_statics)
         ,action_bucket AS(
         SELECT DISTINCT t.action,
                CASE WHEN t.action='views' THEN lower_limit_views 
                     WHEN t.action='likes' THEN lower_limit_likes
                     WHEN t.action='dislikes' THEN lower_limit_dislikes
                     WHEN t.action='comments' THEN lower_limit_comments END AS lower_limit,
                CASE WHEN t.action='views' THEN upper_limit_views 
                     WHEN t.action='likes' THEN upper_limit_likes
                     WHEN t.action='dislikes' THEN upper_limit_dislikes
                     WHEN t.action='comments' THEN upper_limit_comments END AS upper_limit
                FROM (SELECT   
                   UNNEST(ARRAY['views','likes','dislikes','comments']) as action,
                   mini_views+(bucket-1)*views_interval AS lower_limit_views,
                   mini_views+bucket*views_interval AS upper_limit_views,
                   mini_likes+(bucket-1)*likes_interval AS lower_limit_likes,
                   mini_likes+bucket*likes_interval AS upper_limit_likes,
                   mini_dislikes+(bucket-1)*dislikes_interval AS lower_limit_dislikes,
                   mini_dislikes+bucket*dislikes_interval AS upper_limit_dislikes,
                   mini_comments+(bucket-1)*comments_interval AS lower_limit_comments,
                   mini_comments+bucket*comments_interval AS upper_limit_comments
                   FROM interval_statics) AS t
                   ORDER BY t.action)
          ,user_action_bucket AS( 
           --this is a stage of combining information on video_id and bucket raanges
           SELECT f.video_id,
           f.views,
           f.likes,
           f.dislikes,
           f.comments,
           a.action,
           a.lower_limit,
           a.upper_limit
           FROM feature_limited AS f
                CROSS JOIN action_bucket AS a)
          ,register_action_flag AS(
          --count the number for each action for seven days after the first publishing date
          --compute the 14 days consistent rate 
          SELECT u.video_id,
                 u.action,
                 u.lower_limit,
                 u.upper_limit,
                 CASE WHEN action='views' THEN CASE WHEN u.views BETWEEN u.lower_limit and u.upper_limit THEN 1 ELSE 0 END
                      WHEN action='likes' THEN CASE WHEN u.likes BETWEEN u.lower_limit and upper_limit THEN 1 ELSE 0 END 
                      WHEN action='dislikes' THEN CASE WHEN u.dislikes BETWEEN u.lower_limit AND u.upper_limit THEN 1 ELSE 0 END 
                      WHEN action='comments' THEN CASE WHEN comments BETWEEN u.lower_limit AND u.upper_limit THEN 1 ELSE 0 END END AS achieve,
                 a.index_date_action
                 FROM user_action_bucket AS u
                 LEFT JOIN action_log_with_index_dates AS a
                      ON a.video_id=u.video_id) 
          SELECT action,
                 CONCAT(lower_limit,'~',upper_limit) AS count_range,
                 SUM(CASE WHEN achieve=1 THEN 1 ELSE 0 END) AS achieve,
                 '14 days retention' AS index_name,
                 AVG(CASE WHEN achieve=1 THEN 100*index_date_action END) as achieve_index_rate
                 FROM register_action_flag 
                GROUP BY action,lower_limit,upper_limit;
 ```
 ![image](https://user-images.githubusercontent.com/53164959/66327098-357f1380-e965-11e9-9514-e01ab35e0570.png)

```sql
 WITH period (index_last,begin_date,end_date) AS(
     values ('28 days rentation',14,28)
            ),
     users_info AS(
     SELECT video_id,
            CAST(SUBSTRING(publish_time,1,10) AS DATE) AS publish_time,
            CAST(trending_date AS DATE) AS trending_date, 
            MAX(CAST(trending_date AS DATE)) OVER() AS latest_date
            FROM youtube),
     last_week_rentation AS(
     SELECT video_id,
            SIGN(SUM(CASE WHEN trending_date<=latest_date THEN 
                 CASE WHEN trending_date BETWEEN publish_time+'1 day'::interval*p.begin_date
                           AND publish_time+'1 day'::interval*p.end_date THEN 1 ELSE 0 END END)) AS rentation
            FROM users_info AS u,period  AS p
            GROUP BY video_id),  
     first_week_use AS(
        SELECT u.video_id,
               COUNT(DISTINCT(u.trending_date)) AS counts
               FROM users_info AS u,period AS p
               WHERE u.trending_date BETWEEN u.publish_time+'1 day'::interval AND u.publish_time+'7 day'::interval
               GROUP BY u.video_id,u.trending_date
               ORDER BY u.video_id
               ),
     statics AS(
           SELECT f.video_id,
                  SUM(f.counts) AS dt_counts,
                  l.rentation AS rentation
           FROM first_week_use AS f
                LEFT JOIN last_week_rentation AS l
                ON f.video_id=l.video_id
                GROUP BY  f.video_id,l.rentation
                HAVING SUM(f.counts) <8)
       SELECT DISTINCT(dt_counts) AS dates,
              SUM(dt_counts) AS counts,
              ROUND(CAST(SUM(dt_counts) AS NUMERIC)/SUM(SUM(dt_counts)) OVER()*100,2) as rate,
              ROUND(CAST(AVG(100*rentation) AS NUMERIC),2) AS achieve_ratio
              FROM statics 
              GROUP BY dt_counts
              ORDER BY dates;
```

![image](https://user-images.githubusercontent.com/53164959/66366698-4c0c8580-e9cc-11e9-9849-5d617c6cfe97.png)


WITH interval(index_name,index) AS(
    SELECT CONCAT(GENERATE_SERIES(1,12),'-month'), 
           GENERATE_SERIES(1,12)),
    data_modified AS(
    SELECT video_id,
           MIN(CAST(trending_date AS DATE))  AS first_date_clicked,
           CAST(trending_date AS DATE) AS trending_date,
           MAX(CAST(trending_date AS DATE)) OVER() AS latest_date
          FROM youtube
          GROUP BY video_id,trending_date)
    SELECT video_id,
           first_date_clicked,
           trending_date
           FROM data_modified
           ORDER BY video_id,trending_date
           limit 100;
  ![image](https://user-images.githubusercontent.com/53164959/66702690-31783a80-ed45-11e9-9441-2ff90ede7821.png)
