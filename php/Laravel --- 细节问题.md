### ORM

*  使用模型的关联关系，如果过滤了关联对象的字段，注意要把关联的外键字段放出来，不然关联的对象无法关联到被查对象上，即没有数据出现。
* 软删除使用的 null 来判断，如果你想自定义为 0 表示未删除需要重新实现一下它的 traits。具体可以参考 https://learnku.com/articles/63989
* 使用 create 方法创建对象时，会报 Add xxx to fillable property to allow mass assignment on xxx 错误，可以参考 https://www.jb51.net/article/124878.htm 或者设置 model 的 $unguarded 变量为 true。

### 表单验证
* 对于数据库的验证规则，比如 exists，unique，它们的查询不会包括软删除的字段，需要加上 withoutTrashed() 的方法，但是它的软删除还是使用 null 来判断的。
