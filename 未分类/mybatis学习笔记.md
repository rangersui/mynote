---
author:王启隆
---



## 3.7 万能Map

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

## 3.8 其他

### 1 模糊查询

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

## 4.1 核心配置文件

### 1 核心配置文件

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

### 2 环境配置

**不过要记住：尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。**

学会配置多套运行环境！

Mybatis默认的事务管理器就是JDBC，连接池：POOLED

### 3 属性（properties）

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







