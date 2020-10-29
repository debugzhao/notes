#### 模糊匹配注意SQL注入

[mybatis中的#和$的区别](https://www.cnblogs.com/jokmangood/p/11705850.html)，在接收前端传过来的值的时候尽量用#{}接收，#将传过来的值作为一个字符串，会自动加一对引号，几乎可以避免SQL注入的问题

以下SQL可以避免SQL注入问题

```sql
select * from department where name like #{name}
map.put("name","'%端%' or 1 = 1");
```

以下情况会出现SQL注入问题（会查询出所有的数据）

```sql
select * from department where name like ${name}
map.put("name","'%端%' or 1 = 1");
```

#### 配置解析

#### 声明周期和作用域

#### ResultMap结果映射集

#### 日志工厂

#### 分页查询

#### 注解开发

##### 使用注解实现CRUD