自己业务系统中用到的，根据业务修改 where 和 order 条件
* 下一篇
```sql
    select t1.id, t1.title
    from (select
                 ROW_NUMBER() over (w) as row_id,
                 a.id, a.title
        from article a
        where a.site_id = {#site_id}
          and a.category_id = {#category_id}
          and a.is_hidden = 0
          and a.delete_time = 0
          and a.trash_time = 0
        window w as (order by a.on_top desc, publish_time desc, create_time desc)) t1,
         (select cur
          from (select
                       ROW_NUMBER() over (w) as cur,
                       a.id
                from article a
                where a.site_id = {#site_id}
                and a.category_id = {#category_id}
                and a.is_hidden = 0
                and a.delete_time = 0
                and a.trash_time = 0
                window w as (order by a.on_top desc, publish_time desc, create_time desc)) a
          where a.id = article_id) t2
    where t1.row_id > t2.cur
    order by row_id limit 1;

```

* 上一篇
```sql

    select t1.id, t1.title
    from (select
                 ROW_NUMBER() over (w) as row_id,
                 a.id, a.title
        from article a
        where a.site_id = {#site_id}
          and a.category_id = {#category_id}
          and a.is_hidden = 0
          and a.delete_time = 0
          and a.trash_time = 0
        window w as (order by a.on_top desc, publish_time desc, create_time desc)) t1,
         (select cur
          from (select
                       ROW_NUMBER() over (w) as cur,
                       a.id
                from article a
                where a.site_id = {#site_id}
                and a.category_id = {#category_id}
                and a.is_hidden = 0
                and a.delete_time = 0
                and a.trash_time = 0
                 window w as (order by a.on_top desc, publish_time desc, create_time desc)) a
          where a.id = article_id) t2
    where t1.row_id < t2.cur
    order by row_id desc limit 1;


```
