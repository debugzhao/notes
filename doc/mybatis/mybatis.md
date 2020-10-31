#### 1.模糊匹配注意SQL注入

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

#### 2.配置解析

##### 2.1.environment配置文件

通过`environment`标签可以配置多数据源

```xml
<environments default="mysql">
    <!--MySQL数据源-->
    <environment id="mysql">
    </environment>
    <!--oracle数据源-->
    <environment id="oracle">
    </environment>
</environments>
```

##### 2.2.properties配置文件

可以通过引入外部`properties`配置文件，实现配置文件的松耦合

db.properties配置文件

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/spring_senior?useSSL=false&useUnicode=true&characterEncoding=utf-8
username=root
password=123456
```

mybatis-config.xml配置文件

```xml
<configuration>
    <!--通过properties标签来引入外部的配置文件，以供下面直接取值-->
    <properties resource="db.properties"/>
    <environments default="mysql">
        <environment id="mysql">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <!--这里通过${}来取外部的配置文件属性-->
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```

##### 2.3.typeAlias起别名

1. 通过配置文件指定pojo的包路径

   ```xml
   <typeAliases>
       <package name="com.example.demo.entity"/>
   </typeAliases>
   ```

2. 通过注解给实体类其别名

   ```java
   @Alias("dept")
   public class DeptEntity {}
   ```

#### 3.mapper映射器

MapperRegistry：注册绑定我们写的Mapper.xml文件，每一个mapper.xml文件都需要在mybatis核心配置文件中注册

1. 通过resource属性注册mapper文件【推荐使用】

   ```xml
   <mappers>
   	<mapper resource="DeptMapper.xml"/>
   </mappers>
   ```

2. 通过class属性注册mapper文件

   ```xml
   <mappers>
   	<mapper resource="DeptMapper.xml"/>
   </mappers>
   ```

   **注意点：**

   - 接口和它对应的mapper配置文件必须同名
   - 接口和它对应的mapper配置文件必须在同一个包下

3. 使用包扫描注册mapper文件

   ```xml
   <mappers>
   	<package name="com.geek.dao"/>
   </mappers>
   ```

   **注意点：**

   - 接口和它对应的mapper配置文件必须同名
   - 接口和它对应的mapper配置文件必须在同一个包下

#### 4.生命周期和作用域

声明周期和作用域是非常重要的，因为错误使用会导致严重的**并发问题**

**SqlSessionFactoryBuilder**

作用：创建SqlSessionFactory，一旦创建完毕就不再需要了

作用域：局部变量

**SqlSessionFactory**

作用：创建SqlSession的，可以理解为**数据库连接池**，一旦创建就应该在程序运行期间一直存活

作用域：应用作用域（单例模式）

**SqlSession**

创建一个SqlSession可以理解为得到一个数据库连接请求。

SqlSession是线程不安全的，因此不能被共享，它的最佳作用域是一个请求作用域或者方法作用域

用完SqlSession之后需要立即释放资源，避免资源的浪费

<img src="https://i.loli.net/2020/10/31/qWLfRCoaMY42ruA.png" alt="image-20201031120445508" style="zoom:80%;" />

#### 5.ResultMap结果映射集

*解决属性名和字段名不一致的问题*

```xml
<!--结果集映射-->
<resultMap id="deptMap" type="DeptEntity">
    <result column="id" property="id"></result>
    <result column="name" property="deptName"></result>
</resultMap>
	
<select id="list" resultType="DeptEntity" resultMap="deptMap">
    select * from department
</select>
```

#### 6.日志工厂

##### 6.1.STDOUT_LOGGING

```xml
<configuration>
    <properties resource="db.properties"/>

    <!--日志文件配置-->
    <settings>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>
</configuration>
```

##### 6.2.log4j

1. 什么是log4j

   - log4j是apache的开源项目，可以控制日志输出的目的地是控制台、文件、GUI组件
   - 可以控制每一条日志的输出格式
   - 可以控制每一条日志的级别，更加细致的控制日志的输出过程
   - 以上操作都可以通过配置文件来实现，不需要修改代码

2. 实现过程

   1. 导入依赖

      ```xml
      <dependency>
          <groupId>org.apache.logging.log4j</groupId>
          <artifactId>log4j-core</artifactId>
          <version>2.13.3</version>
      </dependency>
      ```

#### 分页查询

#### 注解开发

1. 查询
2. 删除
3. 更新
4. 新增

##### 使用注解实现CRUD

#### 复杂查询

##### 多对一查询

每个学生都要对应一个教师信息

```java
@Data
public class StudentEntity {
    private Integer id;
    private String name;
    private TeacherEntity teacher;
}
```

1. 方式1

   查询所有的学生信息，再根据学生信息的tid嵌套查询教师信息

   ```xml
   <select id="findAll" resultMap="StudentTeacher">
       select * from student
   </select>
   
   <resultMap id="StudentTeacher" type="StudentEntity">
       <result property="id" column="id"/>
       <result property="name" column="name"/>
       <!--复杂的属性我们需要单独处理，association:对象 collection:集合-->
       <association property="teacher" column="tid" javaType="TeacherEntity" select="getTeacher"/>
       <!--<result property="teacher" column="tid"/>-->
   </resultMap>
   
   <select id="getTeacher" resultType="TeacherEntity">
       select * from teacher where id = #{tid}
   </select>
   ```

2. 方式2

   按照结果嵌套处理

   ```xml
   <!--方式2：按照结果嵌套处理-->
   <select id="findAll2" resultMap="studentTeacher2">
       select
           s.id sid,s.name sname,t.name tname,t.id tid
       from
           student s,teacher t
       where
           s.tid = t.id
   </select>
   <resultMap id="studentTeacher2" type="StudentEntity">
       <result property="id" column="sid"/>
       <result property="name" column="sname"/>
       <association property="teacher" javaType="TeacherEntity">
           <result property="name" column="tname"/>
           <result property="id" column="tid"/>
       </association>
   </resultMap>
   ```

##### 一对多查询