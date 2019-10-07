```sql

WITH feature_measures AS(
    SELECT video_id,
           view,
           likes,
           dislikes,
           comment_count AS comments,
           CAST(SUBSTRING(publish_time,1,10) AS DATE) AS publish_time,
           CAST(trending_date AS DATE) AS action_date,
           MAX(CAST(trending_date AS DATE)) OVER() AS latest_date,
           CAST(SUBSTRING(publish_time,1,10) AS DATE)+'7 day'::interval AS one_week_interval
           FROM youtube)
     ,one_week_statics AS(
     SELECT DISTINCT percentile_cont(0) within group ( order by view ) AS mini_view,
                     percentile_cont(0.5) within group ( order by view ) AS median_view,
                     percentile_cont(0) within group ( order by likes ) AS mini_likes,
                     percentile_cont(0.5) within group ( order by likes ) AS median_likes,
                     percentile_cont(0) within group ( order by dislikes ) AS mini_dislikes,
                     percentile_cont(0.5) within group ( order by dislikes) AS median_dislikes,
                     percentile_cont(0) within group ( order by comments ) AS mini_comments,
                     percentile_cont(0.5) within group ( order by comments ) AS median_comments
            FROM feature_measures
            WHERE action_date<latest_date and action_date=one_week_interval)
     ,bucket_action AS(
      SELECT UNNEST(ARRAY['views','likes','dislikes','comments']) AS action
      )
     SELECT Distinct t.action,
            CASE WHEN t.action='views' THEN t.bucket_views  
                 WHEN t.action='likes' THEN t.bucket_likes  
                 WHEN t.action='dislikes' THEN t.bucket_dislikes   
                 WHEN t.action='comments' THEN t.bucket_comments END AS buckets,
             CASE WHEN t.action='views' THEN COUNT(1) OVER(PARTITION BY t.action,t.bucket_views)
                 WHEN t.action='likes' THEN COUNT(1)  OVER(PARTITION BY t.action,t.bucket_likes)
                 WHEN t.action='dislikes' THEN COUNT(1) OVER(PARTITION BY t.action,t.bucket_dislikes)
                 WHEN t.action='comments' THEN COUNT(1) OVER(PARTITION BY t.action,t.bucket_comments)6END AS counts
      FROM(SELECT  WIDTH_BUCKET(f.view,o.mini_view,o.median_view,4) AS bucket_views,  
                   WIDTH_BUCKET(f.likes,o.mini_likes,o.median_likes,4)  AS bucket_likes,
                   WIDTH_BUCKET(f.dislikes,o.mini_dislikes,o.median_dislikes,4) AS bucket_dislikes,
                   WIDTH_BUCKET(f.comments,o.mini_comments,o.median_comments,4) AS bucket_comments,
                   b.action
                  FROM bucket_action AS a,feature_measures AS f,one_week_statics AS o,bucket_action AS b) as t
           GROUP BY t.action,t.bucket_views,t.bucket_likes,t.bucket_dislikes,t.bucket_comments
           ORDER BY action;
           
```
