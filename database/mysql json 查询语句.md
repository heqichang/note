### 如果保存的是一个 array 数组，里面又是 object 对象，类似结构如下

|  id   | draft  |
|  ----  | ----  |
| 1  | [{"id": 1, "name": "t1"}, {"id": 2, "name": "t2"}] |
| 2  | [{"id": 2, "name": "t2"}, {"id": 3, "name": "t3"}] |


如果我想查所有包含 name = t3 的数据，之前查 mysql 文档以为可以想下面这样写，但是查不到
 ``` sql
select * from hc_draft where draft->'$[*].name' = 't3'
 ```

 需要这样写

 ``` sql
select * from hc_draft where json_contains(draft, json_object('name', 't3'))
 ```

 参考:
 https://www.cnblogs.com/inkyi/p/15745242.html

