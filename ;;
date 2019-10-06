```sql
WITH features_measures AS(
    SELECT video_id,
           view,
           likes,
           dislikes,
           comment_count,
           CAST(SUBSTRING(publish_time,1,10) AS DATE) AS publish_time,
           CAST(trending_date AS DATE) AS action_date,
           MAX(CAST(trending_date AS DATE)) OVER() AS latest_date,
           CAST(SUBSTRING(publish_time,1,10) AS DATE)+'7 day'::interval AS one_week_interval
           FROM youtube)
     ,one_week_statics AS(
     SELECT DISTINCT MAX(view) OVER() AS max_view,
                     MIN(view) OVER() AS min_view,
                     MAX(likes) OVER() AS max_likes,
                     MIN(likes) OVER() AS min_likes,
                     MAX(dislikes) OVER() AS max_dislikes,
                     MIN(dislikes) OVER() AS min_dislikes,
                     MAX(comment_count) OVER() AS max_comment,
                     MIN(comment_count) OVER() AS min_comment
            FROM features_measures
            WHERE action_date<latest_date and action_date=one_week_interval)
     ,interval_range AS(
      SELECT (max_view-min_view)/5 AS view_interval,
             (max_likes-min_likes)/5 AS likes_interval,
             (max_dislikes-min_dislikes)/5 AS dislikes_interval,
             (max_comment-min_comment)/5 AS comment_interval
             FROM one_week_statics)
     ,action_bucket AS(
      SELECT UNNEST(ARRAY['view','view','view','view','view','likes','likes','likes','likes','likes','dislikes'
                    ,'dislikes','dislikes','dislikes','dislikes','comment_count','comment_count','comment_count','comment_count','comment_count'])
                   AS action,
             UNNEST(ARRAY[o.min_view,1*i.view_interval+1,2*i.view_interval+1,3*i.view_interval+1,4*i.view_interval+1,
                          o.min_likes,1*i.likes_interval+1,2*i.likes_interval+1,3*i.likes_interval+1,4*i.likes_interval+1,
                          o.min_dislikes,1*i.dislikes_interval+1,2*i.dislikes_interval+1,3*i.dislikes_interval+1,4*i.dislikes_interval+1,
                          o.min_comment,1*i.comment_interval+1,2*i.comment_interval+1,3*i.comment_interval+1,4*i.comment_interval+1])
                   AS min_count,
              UNNEST(ARRAY[1*i.view_interval,2*i.view_interval,3*i.view_interval,4*i.view_interval,o.max_view,
                           1*i.likes_interval,2*i.likes_interval,3*i.likes_interval,4*i.likes_interval,o.max_likes,
                           1*i.dislikes_interval,2*i.dislikes_interval,3*i.dislikes_interval,4*i.dislikes_interval,o.max_dislikes,
                           1*i.comment_interval,2*i.comment_interval,3*i.comment_interval,4*i.comment_interval,o.max_comment])
            
             FROM one_week_statics AS o,interval_range AS i)
```

