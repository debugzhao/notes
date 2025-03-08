#### 常见操作符

##### UNION

> MySQL UNION 操作符用于连接两个以上的 SELECT 语句的结果组合到一个结果集合中。多个 SELECT 语句会删除重复的数据。
>
> UNION 操作符用于合并两个或多个 SELECT 语句的结果集。
> 请注意，UNION 内部的 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每条 SELECT 语句中的列的顺序必须相同。

##### group by

> GROUP BY 语句根据一个或多个列对结果集进行分组。在分组的列上我们可以使用 COUNT, SUM, AVG,等函数。

with rollup

> WITH ROLLUP 可以实现在分组统计数据基础上再进行**相同的统计**（SUM,AVG,COUNT…）
>
> ```mysql
> mysql> SELECT name, SUM(singin) as singin_count FROM  employee_tbl GROUP BY name WITH ROLLUP;
> ```
>
> 优化版
>
> 我们可以使用 coalesce 来设置一个可以取代 NUll 的名称，coalesce 语法：
>
> ```mysql
> select coalesce(a,b,c);
> 
> #参数说明：如果a==null,则选择b；如果b==null,则选择c；如果a!=null,则选择a；如果a b c 都为null ，则返回为null（没意义）。
> ```
>
> ```mysql
> mysql> SELECT coalesce(name, '总数'), SUM(singin) as singin_count FROM  employee_tbl GROUP BY name WITH ROLLUP;
> ```

#### mysql函数

##### CONCAT 

> 字符串 s1,s2 等多个字符串合并为一个字符串

##### FORMAT(x,n)

> 函数可以将数字 x 进行格式化 "#,###.##", 将 x 保留到小数点后 n 位，最后一位四舍五入。

##### REPLACE(s,s1,s2)

> 将字符串 s2 替代字符串 s 中的字符串 s1

##### REVERSE(s)

> 将字符串s的顺序反过来

##### ABS(x)

>  返回 x 的绝对值　　

##### CONNECTION_ID()

>  返回服务器的连接数

##### CAST(x AS type)

> 转换数据类型
>
> ```mysql
> SELECT CAST("2017-08-29" AS DATE);
> -> 2017-08-29
> ```

##### CURRENT_USER()

> 返回当前用户
>
> ```mysql
> SELECT CURRENT_USER();
> ```

##### DATABASE()

> 返回当前数据库名
>
> ```mysql
> SELECT DATABASE();   
> ```

##### IF(expr,v1,v2)

> 如果表达式 expr 成立，返回结果 v1；否则，返回结果 v2。
>
> ```mysql
> SELECT IF(1 > 0,'正确','错误') 
> ```

##### IFNULL(v1,v2)

>  如果 v1 的值不为 NULL，则返回 v1，否则返回 v2。
>
> ```mysql
> SELECT IFNULL(null,'Hello Word')
> ->Hello Word
> ```

##### Having关键字

> **having 关键字后面直接跟聚集函数**
>
> **在 SQL 中增加 HAVING 子句原因是，WHERE 关键字无法与合计函数一起使用。**
>
> 实例：查询选修3门课以上(含3门)的学生的学号和选修课程数
>
> ```mysql
> select Sno as 学号 ,count(course.Cno) as 选修课程数
> From SC,course
> Where course.Cno=SC.Cno
> Group by Sno
> Having Count(course.Cno)>=3;
> ```

#### MySQL连接查询

##### 自身连接

>   查询每个学生的间接选修课 
>
> ```mysql
> select SC.Sno as 学号,
> FIRST.Cname as 直接选修课,
> SECOND.Cname as 间接选修课
> from SC,
> course as FIRST,
> course as SECOND
> where FIRST.Cno=SC.Cno
> and FIRST.Cpno=SECOND.Cno;
> ```

#### velocity教程

> 在velocity中所有的关键字以#开头，所有的变量以  **美元**  开头，变量取值推荐使用    **美元{!}**  写法。
>
> 单行注释 ##
>
> 多行注释 #*   /n  *#
>
> 文档注释 #**    /n   *#