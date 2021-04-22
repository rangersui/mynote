# 1、Spring

### 1.1、简介

* Sprin：春天······>给软件行业带来了春天！
* 2002，首次推出了Spring框架的雏形：interface21
*  2004年3月24日，*Spring*框架以interface21框架为基础,经过重新设计,发布了1.0正式版本。
* Rod Johnson——Spring Framework创始人，著名作者。 Rod在悉尼大学不仅获得了计算机学位，同时还获得了音乐学位。更令人吃惊的是在回到软件开发领域之前，他还获得了音乐学的博士学位。
* Spring理念：使现有的技术更加容易使用，本身是一个大杂烩
* SSH：Struct2 + Spring + Hibernate
* SSM：SpringMVC + Spring + Mybatis
* 官网文档：https://docs.spring.io/spring-framework/docs/current/reference/html/index.html
* Github：https://github.com/spring-projects/spring-framework

```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.14.RELEASE</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.2.14.RELEASE</version>
</dependency>
```

### 1.2、优点

* Spring是一个开源的免费的框架（容器）！
* Spring是一个轻量级的、非入侵式的框架！
* 控制反转（IOC），面向切面编程（AOP）！
* 支持事务的处理，对框架整合的支持！

**总结一句话：Spring就是一个轻量级的控制反转（IOC），面向切面编程（AOP）的框架！**

### 1.3、组成

![img](file:///C:\Users\y\AppData\Roaming\Tencent\Users\1840347063\TIM\WinTemp\RichOle\ED`{_567$EOI5{74YO$94F2.png)

### 1.4、拓展

在Spring的官网有这个介绍：现代化的java开发，说白就是基于Spring的开发！

* Spring Boot

  * 一个快速开发的脚手架
  * 基于 SpringBoot 可以快速的开发单个微服务
  * 约定大于配置！

* Spring Cloud

  * SpringCloud 是基于 SpringBoot 实现的

  

现在大多数公司就在使用SpringBoot进行快速开发，学习SpringBoot的前提，需要完全掌握Spring及SpringMVC！承上启下的作用！

Spring弊端：

**弊端：发展了太久之后，违背了原来的理念！配置十分繁琐，人称 “ 配置地狱！”**

# 2、IOC理论推导

1、UserDao接口

2、UserDaoImpl实现类

3、UserService业务接口

4、UserServiceImpl业务实现类



在我们之前的业务中，用户的需求可能会影响我们原来的代码，我们需要根据用户的需求去修改源代码！如果程序代码量十分大，修改一次的成本代价十分昂贵。

我们使用一个set接口实现，已经发生了革命性的变化！

```java
    //利用set进行动态实现值的注入！
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
```

* 之前程序是主动创建对象，控制在程序员手上！
* 使用了set注入之后，程序不再具有主动性，而是变成了被动的接收对象！
* 这就是控制反转的思想！

这种思想，从本质上解决了问题，我们程序员不用再去管理对象的创建了。系统的耦合性大大降低，可以更加专注在业务的实现上。这是IOC的原型。

# 3、IOC创建对象的方式

1. 使用无参构造创建对象，默认！

2. 假设我们要使用有参构造创建对象！

   1. 下标赋值

      ```xml
          <bean id="user" class="com.kuang.pojo.User">
              <constructor-arg index="0" value="czw" />
          </bean>
      ```

   2. 类型（不建议使用）

      ```xml
          <bean id="user" class="com.kuang.pojo.User">
              <constructor-arg type="java.lang.String" value="czy" />
          </bean>
      ```

   3. 参数名

      ```xml
          <bean id="user" class="com.kuang.pojo.User">
              <constructor-arg name="name" value="czw" />
          </bean>
      ```

**总结：在配置文件加载的时候，容器中管理的对象就已经初始化了！**

# 4、Spring配置

###  4.1、别名

就是起个别名

```xml
<alias name="user" alias="user123123123" />
```

### 4.2、Bean配置

```xml
<!--
    id：bean的唯一标识符，也就是相当于我们学的对象名
    class：bean对象所对应的全限定名：包名+类型
    name：也是别名，可以取多个别名
-->
<bean id="user" class="com.kuang.pojo.User" name="user12321">
</bean>
```

### 4.3、import

这个import，一般用于团队开发使用，csdn

# 6、依赖注入

### 6.1、构造器注入

已经学过

### 6.2、Set方式注入【重点】

* 依赖注入
  * 依赖：bean对象的创建依赖于容器
  * 注入：bean对象中的所有属性，由容器来注入！

【环境搭建】

1. 复杂类型

```java
package com.kuang.pojo;

public class Address {
    private String address;

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }
}
```

2. 真实测试对象

 ```java
package com.kuang.pojo;
import java.util.*;
public class Student {
    private String name;
    private Address address;
    private String[] books;
    private List<String> hobbys;
    private Map<Student, String> card;
    private Set<String> games;
    private Properties info;
    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", address=" + address +
                ", books=" + Arrays.toString(books) +
                ", hobbys=" + hobbys +
                ", card=" + card +
                ", games=" + games +
                ", info=" + info +
                ", wife='" + wife + '\'' +
                '}';
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public Address getAddress() {
        return address;
    }

    public void setAddress(Address address) {
        this.address = address;
    }
    public String[] getBooks() {
        return books;
    }
    public void setBooks(String[] books) {
        this.books = books;
    }
    public List<String> getHobbys() {
        return hobbys;
    }
    public void setHobbys(List<String> hobbys) {
        this.hobbys = hobbys;
    }
    public Map<Student, String> getCard() {
        return card;
    }
    public void setCard(Map<Student, String> card) {
        this.card = card;
    }
    public Set<String> getGames() {
        return games;
    }
    public void setGames(Set<String> games) {
        this.games = games;
    }
    public Properties getInfo() {
        return info;
    }
    public void setInfo(Properties info) {
        this.info = info;
    }
    public String getWife() {
        return wife;
    }
    public void setWife(String wife) {
        this.wife = wife;
    }
    private String wife;
}
 ```

3. beans.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="student" class="com.kuang.pojo.Student">
        <!--第一种，普通值注入，value-->
        <property name="name" value="wql"/>
    </bean>
</beans>
```

4. 测试类

```java
public class MyTest {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        Student student = (Student) context.getBean("student");
        System.out.println(student.getName());
    }
}
```

**完善注入方式**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="address" class="com.kuang.pojo.Address"/>

    <bean id="student" class="com.kuang.pojo.Student">
        <!--第一种，普通值注入，value-->
        <property name="name" value="wql"/>

        <!--第二种，Bean注入，ref-->
        <property name="address" ref="address"/>

        <!--数组注入，ref-->
        <property name="books">
            <array>
                <value>wql</value>
                <value>czy</value>
                <value>zzy</value>
            </array>
        </property>

        <!--list-->
        <property name="hobbys">
            <list>
                <value>play wql</value>
                <value>play czw</value>
                <value>play zzy</value>
            </list>
        </property>

        <!--map-->
        <property name="card">
            <map>
                <entry key="1" value="111"/>
                <entry key="2" value="222"/>
                <entry key="3" value="333"/>
            </map>
        </property>

        <!--set-->
        <property name="games">
            <set>
                <value>LOL</value>
                <value>COC</value>
                <value>HeartStone</value>
            </set>
        </property>

        <!--null-->
        <property name="wife">
            <null/>
        </property>

        <!--Properties-->
        <property name="info">
            <props>
                <prop key="学号">19120228</prop>
                <prop key="性别">男</prop>
                <prop key="姓名">wql</prop>
            </props>
        </property>
    </bean>

</beans>
```

### 6.3、拓展方式注入

分为c命名注入与p命名注入

见官方文档

功能：简化简单对象的注入方式

注意点：p命名和c命名不能直接使用，需要导入约束，其在官方文档中有

### 6.4、bean的作用域

| Scope                                                        | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [singleton](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-singleton) | (Default) Scopes a single bean definition to a single object instance for each Spring IoC container. |
| [prototype](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-prototype) | Scopes a single bean definition to any number of object instances. |
| [request](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-request) | Scopes a single bean definition to the lifecycle of a single HTTP request. That is, each HTTP request has its own instance of a bean created off the back of a single bean definition. Only valid in the context of a web-aware Spring `ApplicationContext`. |
| [session](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-session) | Scopes a single bean definition to the lifecycle of an HTTP `Session`. Only valid in the context of a web-aware Spring `ApplicationContext`. |
| [application](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-application) | Scopes a single bean definition to the lifecycle of a `ServletContext`. Only valid in the context of a web-aware Spring `ApplicationContext`. |
| [websocket](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-stomp-websocket-scope) | Scopes a single bean definition to the lifecycle of a `WebSocket`. Only valid in the context of a web-aware Spring `ApplicationContext`. |

1. 单例模式（Spring默认模式）
2. 原型模式：每次从容器中get的时候，都会产生一个新对象！
3. 其余的request、session、application，这些个只能在web开发中使用到！

# 7、Bean的自动装配

* 自动装配是Spring满足bean依赖的一种方式！
* Spring会在上下文中自动寻找，并自动给bean装配属性！

在Spring中有三种装配的方式

1. 在xml中显示的配置
2. 在java中显示配置
3. 隐式的自动装配bean【重要】

```text
byName:会自动在容器上下文中查找，和自己对象set方法后面的值对应的beanid！
byType:会自动在容器上下文中查找，和自己对象属性类型相同的bean！
通过autowire="byName"
```

小结：

* byName的时候，需要保证所有bean的id唯一，并且这个bean需要和自动注入的属性的set方法的值一致！
* byType的时候，需要保证所有bean的class唯一，并且这个bean需要和自动注入的属性类型一致！

### 7.1、使用注解实现自动装配

The introduction of annotation-based configuration raised the question of whether this approach is “better” than XML.

要使用注解须知：

1. 导入约束——xmlns:context约束
2. 配置注解的支持——<context:annotation-config/>

**@Autowired注解**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>
</beans>
```

```xml
<bean id="cat" class="com.kuang.pojo.Cat"/>
<bean id="dog" class="com.kuang.pojo.Dog"/>
<bean id="People" class="com.kuang.pojo.People"/>
```

```java
@Autowired
private Cat cat;
@Autowired
private Dog dog;
```

如果@Autowired自动装配的环境比较复杂，自动装配无法通过一个注解【@Autowired】完成的时候，我们可以使用@Qualifier(value="xxx")去配置@Autowired的使用，指定一个唯一的bean对象注入！

**@Resource**注解

同理

小结：

@Resource与@Autowired区别：

* 都是用来自动装配的，都可以放在属性子段上
* @Autowired通过byType实现，要求这个对象必须存在【重要】
* @Resource默认用byName实现，如果找不到名字，就通过byType实现！两个都找不到则报错【重要】
* 执行顺序不同：@Autowired通过byType实现。@Resource默认用byName实现。

 # 8、使用注解开发

使用注解需要导入context约束，增加注解的支持！

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>
</beans>
```

1. bean

2. 属性如何注入

   ```java
   //等价于 <bean id="user" class="com.kuang.pojo.User"/>
   //@Component 组件
   @Component
   public class User {
   
       //等价于 <property name="name" value="wql"/>
       // @Value("wql")
       public String name;
   
   
       @Value("wql")
       public void setName(String name) {
           this.name = name;
       }
   }
   ```

3. 衍生的注解

   @Component有几个衍生注解，我们在web开发中，会按照mvc三层架构分层！

   * dao【@Repository】

   * service【@Service】

   * controller【@Controller】

     这四个注解功能都是一样的，都是代表将某个类注册装配到Spring中，装配Bean

4. 自动装配置

   ```text
   ### 注解说明
   * @Autowired注解——自动装配通过类型、名字
     
     自动装配无法通过一个注解【@Autowired】完成的时候，我们可以使用@Qualifier(value="xxx")去配置@Autowired的使用，指定一个唯一的bean对象注入！
   * @Nullable——字段标记了这个注解，说明这个字段可以为null
   * @Resource——自动装配通过名字类型
   * @Component——组件，放在类上，说明这个类被Spring管理了，就是Bean！
   ```

5. 作用域

   | Scope                                                        | Description                                                  |
   | :----------------------------------------------------------- | :----------------------------------------------------------- |
   | [singleton](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-singleton) | (Default) Scopes a single bean definition to a single object instance for each Spring IoC container. |
   | [prototype](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-prototype) | Scopes a single bean definition to any number of object instances. |
   | [request](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-request) | Scopes a single bean definition to the lifecycle of a single HTTP request. That is, each HTTP request has its own instance of a bean created off the back of a single bean definition. Only valid in the context of a web-aware Spring `ApplicationContext`. |
   | [session](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-session) | Scopes a single bean definition to the lifecycle of an HTTP `Session`. Only valid in the context of a web-aware Spring `ApplicationContext`. |
   | [application](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-application) | Scopes a single bean definition to the lifecycle of a `ServletContext`. Only valid in the context of a web-aware Spring `ApplicationContext`. |
   | [websocket](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-stomp-websocket-scope) | Scopes a single bean definition to the lifecycle of a `WebSocket`. Only valid in the context of a web-aware Spring `ApplicationContext`. |

   ```java
   @Scope("prototype")
   public class User {}
   ```

6. 小结

   xml与注解：

   * xml更加万能，适用于任何场合！维护简单方便
   * 注解 不是自己类是用不了，维护相对复杂！

   xml与注解的最佳实践：

   * xml用来管理bean；

   * 注解只负责完成属性的注入；

   * 我们在使用的过程中，只需要注意一个问题：必须让注解生效，就需要开启注解的支持！

     ```xml
     <!--指定要扫描的包，这个包下的注解会生效-->
     <context:component-scan base-package="com.kuang"/>
     <context:annotation-config/>
     ```

# 9、完全使用Java的方式去配置Spring

我们现在要完全不使用Spring的xml配置了，全权交给Java来做！

JavaConfig是Spring的一个子项目，在$Spring4后，他成为了一个核心功能！

**实体类**

```java
//这里这个注解的意思，就是说明这个类被Spring接管了，注册到了容器中
@Component
public class User {
    private String name;

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                '}';
    }

    public String getName() {
        return name;
    }

    @Value("wql")
    public void setName(String name) {
        this.name = name;
    }
}
```

**配置文件**

```java
@Configuration
//这个也会被Spring容器托管，注册到容器中，因为他本来就是一个@Component
//Configuration代表这是一个配置类，就和我们之前看的beans.xml
@ComponentScan("com.kuang.pojo")
public class KuangConfig {

    //注册一个bean，就相当于我们之前写的一个bean标签
    //这个方法的名字就相当于bean标签中的id属性
    //这个方法的返回值，就相当于bean标签中的class属性
    @Bean
    public User getUser() {
        return new User();//就是返回要注入到bean的对象！
    }
}
```

**测试类**

```java
public class MyTest {
    public static void main(String[] args) {
        //如果完全使用了配置类方式去做，我们就只能通过ApplicationConfig上下文来获取容器，通过配置类的class对象加载！
        ApplicationContext context = new AnnotationConfigApplicationContext(KuangConfig.class);
        User getUser = (User) context.getBean("user");
        System.out.println(getUser.getName());
    }
}
```

这种纯java的配置方式，在SpringBoot中随处可见~

# 10、代理模式

为什么要学习代理模式？因为这是SpringAOP的底层！

代理模式的分类：

* 静态代理
* 动态代理

### 10.1、静态代理

角色分析：

* 抽象角色：一般会使用接口或者抽象类来解决
* 真实角色：被代理的角色
* 代理角色：代理真实角色，代理真实角色后，我们一般会做些附属操作
* 客户：访问代理对象的人

代码步骤

1. 接口

   ```java
   //租房
   public interface Rent {
       public void rent();
   
   }
   ```

2. 真实角色

   ```java
   //房东
   public class Host implements Rent{
   
       public void rent() {
           System.out.println("房东要出租房子");
       }
   }
   ```

3. 代理角色

   ```java
   public class Proxy implements Rent{
       private Host host;
   
       public Proxy(Host host) {
           this.host = host;
       }
   
       public Proxy() {
   
       }
   
       public void rent() {
           host.rent();
           hetong();
           fare();
       }
   
       //看房
       public void seeHouse() {
           System.out.println("中介带你看房");
       }
   
       //签合同
       public void hetong() {
           System.out.println("签合同");
       }
   
       //收中介费
       public void fare() {
           System.out.println("收中介费");
       }
   }
   ```

4. 客户端访问代理角色

   ```java
   public class Client {
       public static void main(String[] args) {
           //房东要租房子
           Host host = new Host();
           //代理，中介帮房东租房子，但是代理角色一般会有附属操作！
           Proxy proxy = new Proxy(host);
           //你不用面对房东，直接找中介租房即可！
           proxy.rent();
       }
   }
   ```

代理模式好处：

* 可以使真实角色的操作更加纯粹！不用去关注一些公共的业务
* 公共也就交给代理角色！实现了业务的分工！
* 公共业务发生拓展的时候，便于集中管理！

缺点：

* 一个真实角色就会产生一个代理角色，代码量会翻倍，开发效率会变低~

### 10.2、动态代理

* 动态代理和静态角色代理一样
* 动态代理的代理类是动态生成的，不是我们直接写好的
* 动态代理分为两大类：基于接口的动态代理，基于类的动态代理
  * 基于接口——JDK【这里使用】
  * 基于类：cglib
  * java字节码实现：javasist

需要了解两个类：Proxy（代理），InvocationHandler（调用处理程序）

动态代理的好处：

* 可以使真实角色的操作更加纯粹！不用去关注一些公共的业务
* 公共也就交给代理角色！实现了业务的分工！
* 公共业务发生拓展的时候，便于集中管理！
* 一个动态代理类代理的是一个接口，一般就是对应的一类业务
* 一个动态代理类可以代理多个类，只要是实现了同一个接口即可

代码步骤：

1. 接口

   ```java
   //租房
   public interface Rent {
       public void rent();
   
   }
   ```

2. 真实角色

   ```java
   //房东
   public class Host implements Rent {
   
       public void rent() {
           System.out.println("房东要出租房子");
       }
   }
   ```

3. 动态代理角色

   ```java
   //等会我们要用这个类，自动生成代理类！
   public class ProxyInvocationHandler implements InvocationHandler {
   
       //被代理的接口
       private Rent rent;
   
       public void setRent(Rent rent) {
           this.rent = rent;
       }
   
       //生成得到代理类
       public Object getProxy() {
           return Proxy.newProxyInstance(this.getClass().getClassLoader(), rent.getClass().getInterfaces(), this);
       }
   
       //处理代理实例并返回结果
       public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
           seeHouse();
           //动态代理的本质，就是使用反射机制实现！
           Object result = method.invoke(rent, args);
           fare();
           return result;
       }
   
       public void seeHouse() {
           System.out.println("中介带看房子");
       }
   
       public void fare() {
           System.out.println("中介收中介费");
       }
   }
   ```

4. 客户端访问代理角色

   ```java
   public class Client {
       public static void main(String[] args) {
           //真是角色
           Host host = new Host();
   
           //代理角色：现在没有
           ProxyInvocationHandler pih = new ProxyInvocationHandler();
           //通过调用程序处理角色来处理我们要调用的接口对象！
           pih.setRent(host);
   
           Rent proxy = (Rent) pih.getProxy();
           proxy.rent();
       }
   }
   ```

# 11、AOP

### 11.1、什么是AOP

AOP (Aspect Oriented Programming)意为:面向切面编程,通过预编译方式和运行期动态代理实现程序功能的统- -维护的一 种技术。AOP是0OP的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容, 是函数式编程的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

### 11.2AOP在Spring中的作用

**提供声明式服务，允许用户自定义切面**

* 横切关注点:跨越应用程序多个模块的方法或功能。即是，与我们业务逻辑无关的，但是我们需要关注的部
  分，就是横切关注点。如日志,安全,缓存,事务等等...
* 切面(ASPECT) :横切关注点被模块化的特殊对象。即，它是一一个类。
* 通知(Advice) :切面必须要完成的工作。即，它是类中的一个方法。
* 目标(Target) :被通知对象。
* 代理(Proxy) :向目标对象应用通知之后创建的对象。
* 切入点(PointCut) :切面通知执行的“地点”的定义。
* 连接点(JointPoint) :与切入点匹配的执行点。

### 11.3、使用Spring实现AOP

【重点】使用AOP织入，需要导入一个依赖包！

```xml
<!-- https://mvnrepository.com/artifact/org.aspectj/aspectjweaver -->
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.5</version>
</dependency>
```

**方式一：使用Spring的API接口**

AfterLog.java

```java
public class AfterLog implements AfterReturningAdvice {

    //returnValue：返回值
    public void afterReturning(Object returnValue, Method method, Object[] objects, Object target) throws Throwable {
        System.out.println("执行了：" + method.getName() + "方法，返回结果为：" + returnValue);
    }
}
```

Log.java

```java
public class Log implements MethodBeforeAdvice {
    //method：要执行的目标对象的方法
    //objects：参数
    //o：目标对象
    public void before(Method method, Object[] objects, Object o) throws Throwable {
        System.out.println(o.getClass().getName() + "的" + method.getName() + "被执行了");
    }
}
```

UserService.java

```java
public interface UserService {
    public void add();
    public void delete();
    public void update();
    public void query();
}
```

UserServiceImpl.java

```java
public class UserServiceImpl implements UserService{
    public void add() {
        System.out.println("增加了一个用户");
    }
    public void delete() {
        System.out.println("删除了一个用户");
    }
    public void update() {
        System.out.println("修改了一个用户");
    }
    public void query() {
        System.out.println("查询了一个用户");
    }
}
```

applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--注册bean-->
    <bean id="userService" class="com.kuang.service.UserServiceImpl"/>
    <bean id="log" class="com.kuang.log.Log"/>
    <bean id="afterLog" class="com.kuang.log.AfterLog"/>

    <!--方式一：使用原生Spring API接口-->
    <!--配置AOP:需要导入aop约束-->
    <aop:config>
        <!--切入点:expression：表达式，execution（要执行的位置！）-->
        <aop:pointcut id="pointcut" expression="execution(* com.kuang.service..UserServiceImpl.*(..))"/>

        <!--执行环绕增加-->
        <aop:advisor advice-ref="log" pointcut-ref="pointcut"/>
        <aop:advisor advice-ref="afterLog" pointcut-ref="pointcut"/>

    </aop:config>

</beans>
```

MyTest.java

```java
public class MyTest {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        //动态代理的是接口：注意点！
        UserService userService = (UserService) context.getBean("userService");
        userService.add();
    }
}
```

**方式二：自定义来实现AOP【主要是切面定义】**

见csdn or 视频

**方式三：使用注解实现**

AnnotationPointCut.java

```java
//方式三：使用注解方式实现AOP
@Aspect //标注这个类是一个切面
public class AnnotationPointCut {
    @Before("execution(* com.kuang.service.UserServiceImpl.*(..))")
    public void before() {
        System.out.println("======方法执行前======");
    }

    @After("execution(* com.kuang.service.UserServiceImpl.*(..))")
    public void after() {
        System.out.println("======方法执行后======");
    }

    //在环绕增强中，我们可以给定一个参数，代表我们要获取处理切入的点
    @Around("execution(* com.kuang.service.UserServiceImpl.*(..))")
    public void around(ProceedingJoinPoint jp) throws Throwable {
        System.out.println("======方法执行环绕前======");
        Object proceed = jp.proceed();
        System.out.println("======方法执行环绕后======");
    }
}
```

applicationContext.xml

```xml
<!--方式三-->
<bean id="annotationPointCut" class="com.kuang.diy.AnnotationPointCut"/>
<!--开启注解支持-->
<aop:aspectj-autoproxy/>
```

# 12、整合Mybatis

步骤：

1. 导入相关jar包
   * junit
   * mybatis
   * mysql数据库
   * spring相关的
   * aop织入
   * mybatis-spring【new】
2. 编写配置文件
3. 测试















