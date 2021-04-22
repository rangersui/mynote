# 1、简介

**什么是 MyBatis？**

MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

![MyBatis logo](https://mybatis.org/images/mybatis-logo.png)

### 1、简介

MyBatis 本是apache的一个[开源项目](https://baike.baidu.com/item/开源项目/3406069)iBatis, 2010年这个[项目](https://baike.baidu.com/item/项目/477803)由apache software foundation 迁移到了[google code](https://baike.baidu.com/item/google code/2346604)，并且改名为MyBatis 。2013年11月迁移到[Github](https://baike.baidu.com/item/Github/10145341)。

iBATIS一词来源于“internet”和“abatis”的组合，是一个基于Java的[持久层](https://baike.baidu.com/item/持久层/3584971)框架。iBATIS提供的持久层框架包括SQL Maps和Data Access Objects（DAOs）

当前，最新版本是MyBatis 3.5.6 ，其发布时间是2020年10月6日。

### 2、特点

简单易学：本身就很小且简单。没有任何第三方依赖，最简单安装只要两个jar文件+配置几个sql映射文件易于学习，易于使用，通过文档和源代码，可以比较完全的掌握它的设计思路和实现。

灵活：mybatis不会对应用程序或者数据库的现有设计强加任何影响。 sql写在xml里，便于统一管理和优化。通过sql语句可以满足操作数据库的所有需求。

解除sql与程序代码的耦合：通过提供DAO层，将业务逻辑和数据访问逻辑分离，使系统的设计更清晰，更易维护，更易单元测试。sql和代码的分离，提高了可维护性。

提供映射标签，支持对象与数据库的orm字段关系映射

提供对象关系映射标签，支持对象关系组建维护

提供xml标签，支持编写动态sql。 [2]

### 3、流程

Mybatis -> Spring -> SprintMVC -> SpringBoot

# 2、第一个Mybatis程序

思路：搭建环境-->导入mybatis-->编写代码-->测试！

### 1、搭建环境

搭建mybatis数据库

建立user表

设置user表信息为id  name  password

增加成员

### 2、新建项目

新建一个普通的maven项目

删除src目录

导入maven依赖

​		mysql

​		mybatis

​		junit

### 3、创建一个模块

创建mybatis-01的modules 

编写mybatis核心配置文件

​		见mybatis-config.xml

编写mybatis工具类

​		见mybatisUtils.java

### 3、编写代码

实现类

​		见User.java

Dao接口

​		见UserDao.java

接口实现类

​		见UserDaoImpl

### 4、可能遇到的问题

配置文件没有注册

绑定接口错误

方法名不对

返回类型不对

Maven导出资源问题

# 3、CRUD

### 1、namespace

mybatis-config.xml中的namespace

namespace中的包名要跟Dao/mapper的接口的包名一致

### 2、select

选择，查询语句

其中id对应namespace的方法名

resultType：sql语句执行的返回值

parameterType：参数类型

1、编写接口

2、编写对应mapper中的sql语句

3、测试

### 3、insert

见代码

### 4、update

见代码

### 5、delete

见代码

注意：增删改需要提交事务

### 6、分析错误

标签不要匹配错

resource绑定mapper，需要使用路径

程序配置文件必须符合规范！

NullPointerException，表示没有注册到资源

输出的xml文件中存在资源乱码问题

maven资源没有导出问题

### 7、万能Map

Map传递参数直接在sql取出key值就行

```java
    User getUserById2(Map<String, Object> map);
```

```xml
    <select id="getUserById2" parameterType="map" resultType="com.kuang.pojo.User">
        select * from mybatis.user where id = #{id} and name = #{name}
    </select>
```

```java
	@Test
    public void getUserById2() {
        //第一步：获取sqlsession对象
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        Map<String, Object> map = new HashMap<String, Object>();
        map.put("id", 1);
        map.put("name", "wql");
        User user = mapper.getUserById2(map);
        System.out.println(user);
        sqlSession.close();
    }
```

### 8、其他

#### 1、模糊查询

sql语言使用like来模糊查询

```java
    List<User> getUserLike(String value);
```

```xml
    <select id="getUserLike" resultType="com.kuang.pojo.User">
        select * from mybatis.user where name like #{value}
    </select>
```

```java
    @Test
    public void getUserLike() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        List<User> lis = mapper.getUserLike("%z%");
        for (User it : lis){
            System.out.println(it);
        }
        sqlSession.close();
    }
```

# 4、配置解析

### 1、核心配置文件

MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置和属性信息。 配置文档的顶层结构如下：

```xml
configuration（配置）
properties（属性）
settings（设置）
typeAliases（类型别名）
typeHandlers（类型处理器）
objectFactory（对象工厂）
plugins（插件）
environments（环境配置）
environment（环境变量）
transactionManager（事务管理器）
dataSource（数据源）
databaseIdProvider（数据库厂商标识）
mappers（映射器）
```

### 2、环境配置

**不过要记住：尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。**

学会配置多套运行环境！

Mybatis默认的事务管理器就是JDBC，连接池：POOLED

### 3、属性（properties）

我们可以通过properties属性来实现配置文件

这些属性可以在外部进行配置，并可以进行动态替换。你既可以在典型的 Java 属性文件中配置这些属性，也可以在 properties 元素的子元素中设置。【db.properties】

db.properties

**个人特殊情况：**

(句子太长需要加\\\\\来换行 单个\不行)

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?\\\
    useSSL=true&useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
```

也可在xml中引入外部配置文件properties

```xml
    <!--引入外部配置文件-->
    <properties resource="db.properties">
        <property name="username" value="root"/>
        <property name="password" value="wql89027485"/>
    </properties>
```

### 4、类型别名（typeAliases）

类型别名是为java类型设置一个短的名字

存在的意义仅在于用来减少类完全限定名的冗余

```xml
    <!--可以给实体类起别名-->
    <typeAliases>
        <typeAlias type="com.kuang.pojo.User" alias="User" />
    </typeAliases>
```

也可以指定一个包名，Mybatis会在包名下面搜索需要的java bean，比如：

扫描实体类的包，它的默认别名就为这个类的类名，首字母小写 

```xml
    <!--可以给实体类起别名-->
    <typeAliases>
        <package name="com.kuang.pojo"/>
    </typeAliases>
```

在实体类比较少的时候，使用第一种方式

如果比较多，使用第二种

第一种可以diy，第二种需要通过注解来取别名

```java
@Alias("wql")
public class User {}
```

### 5、设置

就这些

```xml
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="multipleResultSetsEnabled" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```

### 6、其他设置

![image-20210409205831907](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20210409205831907.png)

### 7、映射器（mappers）

MapperRegistry：注册绑定我们的mapper文件

```xml
    <!--每一个Mapper.XML都需要在Mybatis核心配置文件中注册！-->
    <mappers>
        <mapper resource="com/kuang/dao/UserMapper.xml"/>
    </mappers>
```

其他方式：通过class或者package都可以 替换掉mapper  但是注意点增多

### 8、生命周期

理解我们之前讨论过的不同作用域和生命周期类别是至关重要的，因为错误的使用会导致非常严重的**并发**问题。

**SqlSessionFactoryBuilder**

一旦创建了 SqlSessionFactory，就不再需要它了。

局部变量

**SqlSessionFactory**

数据库连接池

SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，**没有任何理由丢弃它或重新创建另一个实例**

SqlSessionFactory 的最佳作用域是应用作用域

最简单的就是使用单例模式或者静态单例模式

**SqlSession**

连接到连接池的一个请求

SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域

用完之后需要赶紧关闭，否则资源会被占用

# 5、解决属性名和字段名不一致的问题

### 1、resultMap

结果集映射

数据库获取数据与对应的赋值元素形成一一映射

```xml
    <resultMap id="UserMap" type="User">
        <!--column数据库中的子段 property实体类中的属性-->
        <result column="id" property="id"></result>
        <result column="name" property="name"></result>
        <result column="pwd" property="password"></result>
    </resultMap>

    <select id="getUserById" resultMap="UserMap">
        select * from mybatis.user where id = #{id}
    </select>
```

# 6、日志

### 1、日志工厂

如果一个数据库操作，出现了异常，我们需要排错，日志就是最好的助手！

曾经：sout、debug

现在：日志工厂

**LOG4**J以及**STDOUT_LOGGING**

在mybatis核心配置文件中，配置我们的日志！

```xml
    <settings>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>
```

# 7、分页

**思考：为什么要分页？**

减少数据的处理量

**使用Limit分页**

```xml
SELECT * from user limit startIndex.pageSize;
SELECT * from user limit 3; #[0, n]
```

**RowBounds分页**

老方法不需要掌握

遇到搜文档即可

**分页插件——PageHelper**

了解即可，如果以后遇到，csdn

# 8、使用注解开发

### 1、面向接口编程

**根本原因：``解耦``，可拓展，提高复用，分层开发中，上层不用管具体实现，规范性更好**

面向对象系统中，不同系统设计人员对于对象内部的设计来说不是那么重要了

面向接口编程，在于使各个对象之间的协作关系，这也是系统设计的主要内容

**关于接口的理解**：

接口本身反映了系统设计人员对系统的抽象理解

接口分两类：

​	第一类是对一个个体的抽象，它可对应为一个抽象体（abstract class）

​	第二类是对一个个体某一方面的抽象，即形成一个抽象面（interface）

一个体有可能有多个抽象面，抽象体与抽象面是有区别的

**三个对象的区别**

面向过程是指，我们考虑问题时，以一个具体的流程（事物过程）为单位，考虑他的实现。

面向对象是指，我们考虑问题时，以对象为单位，考虑它的属性及方法。

接口设计与非接口设计师针对复用技术而言的，与面向对象（过程）不是一个问题，更多地体现是对系统整体的架构。

### 2、使用注解开发

1、注解在接口上实现

```java
    @Select("select * from user")
    List<User> getUsers();
```

2、需要在核心配置文件中绑定接口

```xml
    <!--绑定接口-->
    <mappers>
        <mapper class="com.kuang.dao.UserMapper"/>
    </mappers>
```

3、测试

```java
    @Test
    public void test() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        List<User> users = mapper.getUsers();
        for (User user : users) {
            System.out.println(user);
        }
        sqlSession.close();
    }
```

作用：代码量不多时，可以简化代码，多的时候还是建议用xml编写

本质：反射机制实现

底层：动态代理

### 3、CRUD

我们可以在工具类创建时自动提交事务

```java
    public static SqlSession getSqlSession() {
        return sqlSessionFactory.openSession(true);
    }
```

编写接口

```java
public  interface UserMapper {
    //查询全部用户
    @Select("select * from user")
    List<User> getUsers();

    //方法存在多个参数
    @Select("select * from user where id = #{id}")
    User getUserByID(@Param("id") int id, @Param("name") String name);

    @Insert("insert into user(id, name, pwd) values (#{id}, #{name}, #{pwd})")
    int addUser(User user);

    @Update("update user set name=#{name}, pwd=#{pwd} where id=#{id}")
    int updateUser(User user);

    @Delete("delete from user where id=#{uid}")
    void deleteUser(@Param("uid") int id);
}
```

测试类

```java

import java.util.List;

public class UserMapperTest {

    @Test
    public void test() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        //mapper.addUser(new User(10, "wqlsb", "wqlsb"));

        //mapper.updateUser(new User(10, "wqlnb", "wqlsb"));

        mapper.deleteUser(10);

        List<User> users = mapper.getUsers();
        for (User user : users) {
            System.out.println(user);
        }
        sqlSession.close();
    }
}
```

注意：我们必须要将接口注册绑定到我们的核心配置文件中！

**关于@Param()注解**

基本类型的参数或者String类型，需要加上

引用类型不需要加

如果只有一个基本类型的话，可以忽略，建议还是加上

我们在SQL中引用的就是我们这里的@Param()中设定的属性名

**${} #{} 区别：**

#{}可以防止sql注入 更安全

# 9、Lombok

```text
Project Lombok is a java library that automatically plugs into your editor and build tools, spicing up your java.
Never write another getter or equals method again, with one annotation your class has a fully featured builder, Automate your logging variables, and much more.
```

1、在IDEA中安装Lombok插件！

2、在IDEA中导入jar包

```xml
<!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.16</version>
</dependency>
```

3、在实体类中加注释即可！

@Data：无参构造，get，set，tostring，hashcode，equals

@AllArgsConstructor：有参构造

@NoArgsConstructor：无参构造

4、其他人的看法：

知乎上有位大神发表过对Lombok的一些看法:

这是一种低级脚味的插件。不建议使用。Java发展到今天,各种插件层出不穷，如何甄别各种插件的优劣？能从架构上优化你的设计的。能提高应用程序性能的.实现高度封装可扩展的...像Lombok这种， 像这种插件。已经不仅仅是插件了。改变了你如何编写源码。事实上，少去了的代码，你写上去又如何?如果Java家族到处充斥这样的东西， 那只不过是一坨披着金色颜色的屎，迟早会被其它的语言取代。

# 10、多对一处理

### 1、mysql环境搭建

```sql
CREATE TABLE `teacher` (
  `id` INT(10) NOT NULL,
  `name` VARCHAR(30) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8

INSERT INTO teacher(`id`, `name`) VALUES (1, '秦老师'); 

CREATE TABLE `student` (
  `id` INT(10) NOT NULL,
  `name` VARCHAR(30) DEFAULT NULL,
  `tid` INT(10) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `fktid` (`tid`),
  CONSTRAINT `fktid` FOREIGN KEY (`tid`) REFERENCES `teacher` (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('1', '小明', '1'); 
INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('2', '小红', '1'); 
INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('3', '小张', '1'); 
INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('4', '小李', '1'); 
INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('5', '小王', '1');​
```

### 2、测试环境搭建

1、导入lombok

2、新建实体类Teacher，Student

```java
@Data
public class Student {
    private int id;
    private String name;

    //学生需要关联一个老师
    private Teacher teacher;
}
```

```java
@Data
public class Teacher {
    private int id;
    private String name;
}
```

3、建立Mapper接口

```java
public interface TeacherMapper {

    @Select("select * from teacher where id = #{tid}")
    Teacher getTeacher(@Param("tid") int id);

}
```

4、建立Mapper.XML文件

```xml
<?xml version="1.0" encoding="UTF8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis,org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.kuang.dao.StudentMapper">

</mapper>
```

```xml
<?xml version="1.0" encoding="UTF8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis,org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.kuang.dao.TeacherMapper">

</mapper>
```

5、在核心配置文件中绑定注册我们的Mapper接口或者文件！【方式很多，随心选】

```xml
    <mappers>
        <mapper class="com.kuang.dao.TeacherMapper"/>
        <mapper class="com.kuang.dao.StudentMapper"/>
    </mappers>
```

6、检测查询是否能够成功！

```java
public class MyTest {

    public static void main(String [] args) {
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        TeacherMapper mapper = sqlSession.getMapper(TeacherMapper.class);
        Teacher teacher = mapper.getTeacher(1);
        System.out.println(teacher);
        sqlSession.close();
    }
}
```

### 3、按照查询嵌套处理

**目标：**查询所有的学生信息，以及对应的老师的信息

```java
public List<Student> getStudent();
```

```xml
<select id="getStudent" resultMap="StudentTeacher">
    select * from student
</select>
<resultMap id="StudentTeacher" type="Student">
    <result property="id" column="id" />
    <result property="name" column="name" />
    <!--复杂的属性需要单独处理
        对象：association
        集合：collection
    -->
    <association property="teacher" column="tid" javaType="Teacher" select="getTeacher"/>
</resultMap>

<select id="getTeacher" resultType="Teacher">
    select * from teacher where id = #{id}
</select>
```

### 4、按照结果嵌套处理

**目标：**同上查询所有的学生信息，以及对应的老师的信息

```java
public List<Student> getStudent2();
```

```xml
    <select id="getStudent2" resultMap="StudentTeacher2">
        select s.id sid, s.name sname, t.name tname, t.id
        from student s, teacher t
        where s.tid = t.id
    </select>
    <resultMap id="StudentTeacher2" type="Student">
        <result property="id" column="sid"/>
        <result property="name" column="sname"/>
        <association property="teacher" javaType="Teacher">
            <result property="id" column="id"/>
            <result property="name" column="tname"/>
        </association>
    </resultMap>
```

### 5、回顾Mysql多对一查询方式

子查询

联表查询

# 11、一对多处理

比如：一个老师拥有多个学生

### 1、环境搭建

跟10节一样

### 2、实体类

Student.java

```java
@Data
public class Student {
    private int id;
    private String name;
    private int tid;
}
```

Teacher.java

```java
@Data
public class Teacher {
    private int id;
    private String name;

    //一个老师拥有多个学生
    private List<Student> students;
}
```

### 3、按照查询嵌套处理

```java
    Teacher getTeacher2(@Param("tid") int id);
```

```xml
    <!--按照查询嵌套处理-->
    <select id="getTeacher2" resultMap="TeacherStudent2">
        select * from mybatis.teacher where id = #{tid}
    </select>

    <resultMap id="TeacherStudent2" type="Teacher">
        <collection property="students" javaType="ArrayList" ofType="Student" select="getStudentByTeacherId" column="id"/>
    </resultMap>

    <select id="getStudentByTeacherId" resultType="Student">
        select * from mybatis.student where tid = #{tid}
    </select>
```

### 4、按照结果嵌套处理

```java
    //获取指定老师下的所有学生及老师的信息
    Teacher getTeacher(@Param("tid") int id);
```

```xml
    <!--按结果嵌套查询-->
    <select id="getTeacher" resultMap="TeacherStudent">
        select s.id sid, s.name sname, t.name tname, t.id tid
        from student s, teacher t
        where s.tid = t.id and t.id = #{tid}
    </select>
    <resultMap id="TeacherStudent" type="Teacher">
        <result property="id" column="tid"/>
        <result property="name" column="tname"/>
        <!--复杂的属性需要单独处理
            对象：association
            集合：collection
            javaType="" 指定属性的类型
            集合中的泛型信息，我们使用ofType获取
        -->
        <collection property="students" ofType="Student">
            <result property="id" column="sid"/>
            <result property="name" column="sname"/>
            <result property="tid" column="tid"/>
        </collection>
    </resultMap>
```

### 5、小结

1、关联——association【多对一】

2、集合——collection【一对多】

3、javaType & ofType

​		① javaType 用来指定实体类中属性的类型

​		② ofType 用来指定映射到List或者集合中的pojo类型，泛型中的约束类型

注意点：

​		保证sql的可读性，尽量保证通俗易懂

​		注意一对多和多对一中属性名和子段的问题

​		如果问题不好排查错误，可以使用日志，建议使用Log4j

尽量避免 **慢sql  1s  1000s**

面试高频

​		mysql引擎

​		InnoDB底层原理

​		索引

​		索引优化

# 12、动态SQL

**什么是动态sql：动态sql就是根据不同的条件生成不同的sql语句**

利用动态sql这一特性可以彻底摆脱这种痛苦。

如果你之前用过 JSTL 或任何基于类 XML 语言的文本处理器，你对动态 SQL 元素可能会感觉似曾相识。在 MyBatis 之前的版本中，需要花时间了解大量的元素。借助功能强大的基于 OGNL 的表达式，MyBatis 3 替换了之前的大部分元素，大大精简了元素种类，现在要学习的元素种类比原来的一半还要少。

- if
- choose (when, otherwise)
- trim (where, set)
- foreach

### 1、if

使用动态 SQL 最常见情景是根据条件包含 where 子句的一部分。比如：

```xml
<select id="findActiveBlogWithTitleLike"
     resultType="Blog">
  SELECT * FROM BLOG
  WHERE state = ‘ACTIVE’
  <if test="title != null">
    AND title like #{title}
  </if>
</select>
```

### 2、choose、when、otherwise

有时候，我们不想使用所有的条件，而只是想从多个条件中选择一个使用。针对这种情况，MyBatis 提供了 choose 元素，它有点像 Java 中的 switch 语句。

还是上面的例子，但是策略变为：传入了 “title” 就按 “title” 查找，传入了 “author” 就按 “author” 查找的情形。若两者都没有传入，就返回标记为 featured 的 BLOG（这可能是管理员认为，与其返回大量的无意义随机 Blog，还不如返回一些由管理员精选的 Blog）。

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
```

### 3、trim、where、set

前面几个例子已经方便地解决了一个臭名昭著的动态 SQL 问题。现在回到之前的 “if” 示例，这次我们将 “state = ‘ACTIVE’” 设置成动态条件，看看会发生什么。

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG
  WHERE
  <if test="state != null">
    state = #{state}
  </if>
  <if test="title != null">
    AND title like #{title}
  </if>
  <if test="author != null and author.name != null">
    AND author_name like #{author.name}
  </if>
</select>
```

### 4、foreach

动态 SQL 的另一个常见使用场景是对集合进行遍历（尤其是在构建 IN 条件语句的时候）。比如：

```xml
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT *
  FROM POST P
  WHERE ID in
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
</select>
```

# 13、缓存

### 1、简介

①、什么是缓存【Cache 】？
	存在内存中的临时数据。
	将用户经常查询的数据放在缓存(内存)中，用户去查询数据就不用从磁盘上(关系型数据库数据文件)查询，从缓存中查询,从而提高查询效率,解决了高并发系统的性能问题。
②、为什么使用缓存?
	减少和数据库的交互次数,减少系统开销,提高系统效率。
③、什么样的数据能使用缓存?
	经常查询并且不经常改变的数据。

```tex
查询：连接数据库，耗资源
	一次查询的结果，给他缓存到一个可以直接取到的地方！  --> 内存：缓存
我们再次查询相同数据的时候，直接走缓存，就不用走数据库了
```

### 2、Mybatis缓存

MyBatis包含- 个非常强大的查询缓存特性, 它可以非常方便地定制和配置缓存。缓存可以极大的提升查询效
率。
MyBatis系统中默认定义了两级缓存：一级缓存和二级缓存
	默认情况下，只有一级缓存开启。 (SqISession级别的缓存，也称为本地缓存)
	二级缓存需要手动开启和配置，他是基于namespace级别的缓存。
	为了提高扩展性，MyBatis定义了缓存接口Cache。我们可以通过实现Cache接口来自定义二级缓存

### 3、一级缓存

一级缓存也叫本地缓存：
	与数据库同一次会话期间查询到的数据会放在本地缓存中。
	以后如果需要获取相同的数据，直接从缓存中拿,没必须再去查询数据库。

### 4、二级缓存

二级缓存也叫全局缓存，一级缓存作用域太低了，所以诞生了二级缓存
基于namespace级别的缓存，一个名称空间，对应-一个二级缓存; .
工作机制
	一个会话查询一条数据，这个数据就会被放在当前会话的一-级缓存中;
	如果当前会话关闭了，这个会话对应的一级缓存就没了；但是我们想要的是，会话关闭了，一级缓存中的
数据被保存到二级缓存中;
	新的会话查询信息，就可以从二级缓存中获取内容;
	不同的mapper查出的数据会放在自己对应的缓存(map) 中;

### 5、缓存原理

csdn~

### 6、自定义缓存——ehcache

csdn~

# 完结撒花！