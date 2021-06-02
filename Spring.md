#  Spring

![Spring IOC依赖注入的基本实现原理](E:\Markdown\JAVA\Spring.assets\v2-1bc9445e1c3c6786948929a128538219_1440w.jpg)

[面试题](https://www.jianshu.com/p/f77f8f2722a1)

## 介绍 5/12

[反射](https://www.zhihu.com/question/24304289)

SSH : Struct2 + Spring + Hibernate!

SSM : SpringMvc + Spring + Mybatis!

https://repo1.maven.org/maven2/org/springframework/spring/

### **优点**

- Spring是一个开源的免费的框架（容器）
- Spring是一个轻量级的、非入侵式的框架！
- spring本身是轻量级的，配置是非轻量级的
- 控制反转(IOC)，面向切面编程(AOP)
- 支持事务的处理，对框架整合的支持

Spring是一个轻量级的控制反转(IoC)和面向切面(AOP)编程的容器（框架）

### 组成

[七大模块](https://www.jb51.net/article/129146.htm)

![img](E:\Markdown\JAVA\Spring.assets\20171127110222693.jpg)

### 扩展

Spring Boot一个快速开发的脚手架，基于spring-boot可以快速开发单个微服务

Spring Boot构见一切、Spring Cloud协调一切、Spring Cloud Data Flow连接一切

## IOC

### IOC介绍

业务实现

1. UserDao接口
2. UserDaolmpl实现类
3. UserService业务接口
4. UserServicelmpl业务实现类

这里是Dao层，我们在业务层service使用的，如果dao层操作的数据发生改变，service层使用起来也要改变，引起耦合。

Dao层

```java
public class UserDaoImpl implements UserDao{
    public void getuser(){
        xxxx;
    }
}
```

Service层

```java
public class UserServiceImpl implements UserService{
    private UserDao userDao;
    // UserDao userDao = new UserDaoImpl(); //多态
    // 利用set动态实现值的注入
    public void setUserDao(UserDao userDao){ 
        this.userDao = userDao;
    }
    
    public void getuser(){
        userDao.getuser();
    }
}
```

Test

```java
public class test{
    public static void main(String[] args){
        UserService userService = new UserServiceImpl();
        
        (UserServiceImpl)userService.setUserDao(new UserDaoImpl());
        userService.getUser();
    }
}
```

以前是想用得自己得主动new类 ，现在是交给Spring容器，你想用直接说就行，不用自己造对象了。

这种思想，从本质上解决了问题，程序猿不用再去管理对象的创建了。系统的耦合性大大降低，可以更加专注的在业务的实现上!这是IOC的原型!

### IOC本质

​	控制反转loC(Inversion of Control),是一种设计思想，DI(依赖注入)是实现IoC的一种方法。依赖注入，依赖查找都是实现方式。

![container magic](E:\Markdown\JAVA\Spring.assets\container-magic.png)

​	没有IoC的程序中,我们使用面向对象编程,对象的创建与对象间的依赖关系完全硬编码在程序中，对象的创建由程序自己控制。

​	控制反转后将对象的创建转移给第三方，个人认为所谓控制反转就是获得依赖对象的方式反转了。有了[IoC容器](https://www.zhihu.com/question/335362570)后，把创建和查找依赖对象的控制权交给了容器，由容器进行注入组合对象，所以对象与对象之间是松散耦合

![img](E:\Markdown\JAVA\Spring.assets\v2-89e9e078c7705f5db3bbd64356301cac_720w.jpg)

采用XML方式配置Bean的时候，Bean的定义信息是和实现分离的，而采用注解的方式可以把两者合为一体，Bean的定义信息直接以注解的形式定义在实现类中，从而达到了零配置的目的。

控制反转是一种通过描述(XML或注解)并通过第三方去生产或获取特定对象的方式。在Spring中实现控制反转的是loC容器，其实现方法是依赖注入(Dependency Injection,DI)。

用另一种方式来说:ioc是将对于各种业务组件或者说对象的生命周期管理从业务代码本身给剥离出去，使业务代码指关心业务本身

## IOC原理

### [知乎](https://zhuanlan.zhihu.com/p/142359989)

**传统的思想（没有IOC容器的情况下）**

![img](E:\Markdown\JAVA\Spring.assets\v2-cdf4ab5893cec2f62467218ef0aadf55_720w.jpg)

传统的Bean创建

1. 程序员对A进行了主动的使用(new)即创建了A类，A类中又依赖B类然后在对B类进行创建
2. 创建对象的权利现在是程序员手上需要哪个类就会对那个类进行创建
3. B类是由于程序员调用了A类有对B类的依赖随着而创建
4. 总之就是自己动手丰衣足食，没有借助任何中间产物。

**IOC容器**

![img](E:\Markdown\JAVA\Spring.assets\v2-32bde737c4c3275a8571fc702a2cc03a_720w.jpg)

- 程序员只需要告诉IOC自己需要那个Bean。就不需要关系该Bean创建的细节已经该Bean的相关依赖。这一切IOC容器已经帮你做好了。
- 凡事有得必有失: 这个过程中我们失去了创建Bean的权利。

**控制反转**

- 控制: 指的就是我们上述说的我们失去的权利(创建对象的创建，或者说控制对象的权利)
- 反转: 指的是控制权的转变。在没有IOC的容器上我们程序员想创建谁就创建谁的权利。在IOC容器下程序员就只能委屈巴巴的向IOC容器索取对象。创建对象的权利由程序员到IOC容器手里。

**思想**

<img src="E:\Markdown\JAVA\Spring.assets\v2-bbb9fb7500c9c84115a5b9c3269c0e1d_720w.jpg" alt="img" style="zoom:200%;" />

### [源码解析](https://zhuanlan.zhihu.com/p/29344811)

描述了Spring容器从加载配置文件到创建出一个完整Bean的作业流程：

![img](E:\Markdown\JAVA\Spring.assets\v2-84ac79b31d7f08a12edf595df5e787ea_720w.jpg)

## Bean源码解析

[Spring源码解析(1)：Bean容器](https://zhuanlan.zhihu.com/p/74832770)

## 例子

com.xxx.pojo.Hello.java

```java
public class Hello{
    private String str;
    
    public String getStr(){ return str; }
    
    public void setStr(String str){ this.str = str; }
    
    @Override
    public String toString(){} //Hello继承了object，没有重写tostring的话默认输出地址,HashCode
}
```

beans.xml

```xml
<beans>
    <!--通过xml找到类的全限定名，然后加载类，最后反射创建-->
    <!-- 使用Spring来创建对象，在Spring这些都称为Bean
	在spring默认创建对象时，是在创建spring的容器时，会创建配置文件中所有的对象
	bean = 对象  new Hello();
	class取到id对应的全限定名，用全限定名以及反射机制创建出对象。
	id = 变量名
	class = new 的对象
	property对象属性设置值。-- 对应Hello中的set方法
 	bean，称为句柄，简单的说就是配置数据变量的set和get方法
	name, 参数名
	-->
    <bean id="hello" class="com.xxx.pojo.Hello">
        <!--属性注入-->
        <property name="str" value="Spring"/>
    </bean>
</beans>
```

```xml
<!--对象作为类型-->
<bean id="mysqlImpl" class="com.xx.dao.UserDaoMysqlImpl"/>
<bean id="oracleImpl" class="com.xx.dao.UserDaoracleImpl"/>
<bean id="UserServiceImpl" class="com.xx.service.UserServiceImpl" >
	<!-- ref : 引用Spring容器中创建好的对象--bean
		value：具体的值,基本数据类型!
	-->
    <!--使用oracleImpl，直接更改xml文件就行-->
	<property name="userDao" ref= "mysqlImpl" />
</bean>
```

Test

```java
@Test
public void test(){
    // 在maven的远程仓库搜索Spring-context然后复制粘贴
    // 获取Spring的上下文对象,拿到Spring的容器
    // 类似于工厂模式，拿到工厂值，工厂进行new对象
    ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml"); // CPX
    // 对象现在在Spring中管理，使用时直接取出。
    Hello hello = (Hello) context.getBean("hello"); // bean的id
    // Hello hello = context.getBean(“hello”,Hello.class);
    // 这样不用强转，和上面的语句作用相同
    hello.getStr();
}
```



这个过程就叫控制反转:

控制:谁来控制对象的创建，传统应用程序的对象是由程序本身控制创建的,使用Spring后, 对象是由Spring来创建的。

反转:程序本身不创建对象,而变成被动的接收对象。

依赖注入:就是利用set方法来进行注入的。

IOC是一种编程思想，由主动的编程变成被动的接收。

可以通过newClassPathXmlApplicationContext去浏览一下底层源码 。

OK，到了现在，我们彻底不用再程序中去改动了，要实现不同的操作，只需要在xmI配置文件中进行修改，所谓的IoC,一句话搞定:对象由Spring来创建,管理，装配!

## 创建对象的方式

在spring默认创建对象时，是在创建spring的容器时，会创建配置文件中所有的对象，默认调用无参构造。  

获取spring上下文的时候（new ClassPathXmlApplicationContext的时候就有了），所有的对象在容器中就已经被创建了。即实例化是在上下文就做了，不在getBean。

### 使用有参创建对象

```xml
<!--参数下标赋值-->
<!--property用setter注入，constructor-arg用构造方法注入-->
<bean id="hello" class="com.xxx.pojo.Hello">
    <constructor-arg index="0" value="Spring"/>
    <!--第一个比哪里-->
</bean>

<!--第二种方式，类型赋值-->
<bean id="hello" class="com.xxx.pojo.Hello">
    <constructor-arg type="java.lang.String" value="Spring"/>
</bean>

<!--第二种方式，参数名赋值-->
<bean id="hello" class="com.xxx.pojo.Hello">
    <constructor-arg name="name" value="Spring"/>
</bean>
```

### new ClassPathXmlApplicationContext

##### `ApplicationContext context = new ClassPathXmlApplicationContext(...)`

顾名思义，在 ClassPath 中寻找 xml 配置文件，根据 xml 文件内容来构建 ApplicationContext。
除了 ClassPathXmlApplicationContext 以外，还有其他构建 ApplicationContext 的方案可供选择，大体的继承结构如下图：

![1](E:\Markdown\JAVA\Spring.assets\c4e28c2eac192d768d62a27457cc4ad0.png)

可以看到，ClassPathXmlApplicationContext 嵌套了很多层才到 ApplicationContext 接口，同样的，我们也可以使用**FileSystemXmlApplicationContext** 和 **AnnotationConfigApplicationContext** 这两个类来构建 ApplicationContext。

**1、FileSystemXmlApplicationContext** 的构造函数需要一个 xml 配置文件在系统中的路径，其他和 ClassPathXmlApplicationContext 基本上一样。

**2、AnnotationConfigApplicationContext** 是基于注解来使用的，它不需要配置文件，采用 java 配置类和各种注解来配置，是比较简单的方式，也是大势所趋。

[Spring IOC容器 源码分析](https://blog.csdn.net/wjkwan/article/details/113368164)



## Spring配置

### 别名

[alias](https://www.dev996.com/spring-ioc-alias.html)

```xml
<bean id="hello" class="com.xxx.pojo.Hello" />
<!--别名，如果添加了别名，我们也可以使用别名获取到这个对象-->
<alias name="hello" alias="hellosss"></alias>
```

 ```java
Hello hello = (Hello) context.getBean("hellosss"); // bean的id
 ```

### Bean配置

```xml
<!--id : bean 的唯一-标识符， 也就是相当于我们学的对象名
class : bean 对象所对应的全限定名:包名+类型
name :也是别名,而且name可以同时取多个别名,user,u1,u2,u3都是别名-->
<bean id="hello" class="com.xxx.pojo.Hello" name="user u1,u2;u3">
	<property name="str" value="Spring"/>
</bean>
```

### applicationContext

applicationContext.xml  -- 就是之前的beans.xml的最终配置文件

>  *如果说BeanFactory是spring的心脏，那么ApplicationContext就是spring的身躯了。ApplicationContext由BeanFactory派生而来，提供了更多面向实际应用的的功能。BeanFactory是SpringIOC的基础容器，ApplicationContext是容器的高级接口*
>
> ​	ApplicationContext，即应用上下文。是Spring的接口的最终父接口是BeanFactory，而最终的实现类，是ClassPathXmlApplicationContext和FileSystemXmlApplicationContext两个类。
>
> ​	ClassPathXmlApplicationContext默认从类路径加载配置文件，而FileSystemXMLApplicationContext默认从文件系统中加载配置文件，其他方面没有很大区别。

applicationContext.xml 

```xml
<beans>
    <!--将之前的bean导入(配置文件)-->
	<import resource="beans.xml"/>
</beans>
```

注意

- 主配置有的bean与导入的bean冲突 即使不同名 也会用主配置的
- 当不同的xml配置导入时，同Id的bean后面的xml会覆盖前面的xml

## DI依赖注入*

**在Spring框架负责创建Bean对象时，动态的将依赖对象注入到Bean组件。**

判定存在component注解后，注解处理器会在spring容器框架中根据用户类的全限定名通过java的反射机制创建这个用户类的对象，并放到spring容器框架中进行管理。

> 获取所引用的实体类，通过实体类名或者类型直接赋值(Set)配对实体类。因为在创建一个对象时，Spring会对对象属性进行赋值，对于实体类，需要进行new一个对象，再进行传入。因此需要首先bean一个依赖的对象，先进行依赖属性赋值，再进行当前对象的属性注入。

### [源码解析](https://www.cnblogs.com/fnlingnzb-learner/p/10694592.html)

系列解析未看

### 构造器注入

[构造器注入](https://blog.csdn.net/qq_41737716/article/details/85596817)

### 通过set方式注入

依赖：bean对象的创建依赖于容器。

注入：bean对象的所有属性由容器注入。

```xml
<bean id="userss" class="com.xx.dao.Userss"/>
<bean id="UserServiceImpl" class="com.xx.service.UserServiceImpl" >
    <!-- 普通值注入，value ; private String user-->
	<property name="user" value= "mysql" />
    
	<!-- Bean注入，ref	; private Userss user-->
	<property name="user" ref= "userss" />
    
    <!-- 数组注入，private String[] user-->
    <property name="user"> 
    	<array>
        	<value>xxx</value>
        </array>
    </property>
    
    <!-- List注入，private List<String> user-->
    <property name="user">
    	<list>
        	<value>xxx</value>
        </list>
    </property>
    
    <!-- Map注入，private List<String, String> user-->
    <property name="user">
    	<map>
        	<entry key="xxx" value="111"/>
        </map>
    </property>
    
    <!-- Set注入，private Set<String> user-->
    <property name="user">
    	<set>
        	<value>xxx</value>
        </set>
    </property>
    
    <!-- null private String user-->
    <property name="user" value= "" />
    <property name="user">
   		<null/>
    </property>

    <!--Properties，Xxx.properties 为Java 语言常见的配置文件,properties底层是HashTable.
private Properties info-->
    <property name="info">
   		<props>
            <prop ket="driver">11111</prop>
            <prop ket="url">11111</prop>
            <prop ket="username">11111</prop>
            <prop ket="password">11111</prop>
        </props>
    </property>
    
</bean>
```



### 拓展方式注入

p-namespace +  c-namespace

```xml
<!--p-> properties  c->constructor-->
<beans xmIns: p="http://www.springframework.org/schema/p"
       xmIns: c="http://www.springframework.org/schema/c"
       >
<!--p命名空间(“命名空间是对作用域的一种特殊的抽象”，它其实包含了处于该作用域内的标识符，且它本身也用一个标识符来表示。它可作为附加信息来区分不同库中相同名称的函数、类、变量等。使用了命名空间即定义了上下文。本质上，命名空间就是定义了一个范围。)注入，可以直接注入属性的值: property-->
    <bean id="user" class=" com.xx.pojo.User" p:name="xxx" p:age="18"/>
    
    <!--c命名空间注入，通过构造器注入: construct-args-->
	<bean id="user1" class=" com.xx.pojo.User" c:name="xxx" c:age="18"/>
    
</beans>
```

注意点: p命名和c命名空间不能直接使用，需要导入xml约束!

### Bean-作用域

#### 单例模式singleton

Spring的默认机制，是指在一个JVM进程中仅有一个实例，只有一个共享的实例存在，所有对这个bean的请求都会返回这个唯一的实例（Spring默认为单例模式）。

Spring框架对单例的支持是采用单例注册表的方式进行实现的。

```xml
 <bean id="user" class=" com.xx.pojo.User" p:name="xxx" p:age="18" scope="singleton"/>
```

```java
@Test
public void test(){
    ApplicationContext context = new ClassPathXmlApplicationContext("userbeans.xm1");
	User user = context.getBean( s: "user", User.class);
	User user2 = context.getBean( s: "user", User.class);
    // 虽然拿出来的对象是多个，但是都是一个Bean单例出来的。
	System.out.print1n(user==user2);
    // 返回true
}
```

#### 原型模式prototpye

每次从容器中get的时候，都会产生一个新对象! Spring就会每次给客户端返回一个新的对象实例。

```xml
 <bean id="user" class=" com.xx.pojo.User" p:name="xxx" p:age="18" scope="prototpye"/>
```

```java
@Test
public void test(){
    ApplicationContext context = new ClassPathXmlApplicationContext("userbeans.xm1");
	User user = context.getBean( s: "user", User.class);
	User user2 = context.getBean( s: "user", User.class);
	System.out.print1n(user==user2);
    // 返回false
}
```

#### [源码](https://blog.csdn.net/cs408/article/details/48982085)

```java
//真正创建Bean实例  
sharedInstance=createBean(beanName,mergedBeanDefinition,args);  
//向单例注册表注册Bean实例  
addSingleton(beanName,sharedInstance);
```

## Bean的自动装配

- 自动装配是Spring满足bean依赖一种方式!

- Spring会在上下文中自动寻找，并自动给bean装配属性!

- 所谓自动装配，是自动获取容器里的实例并返回。

  

在Spring中有三种装配的方式

1. 在xml中显示的配置

2. 在java中显示配置

3. 隐式的自动装配bean [重要]

### @AutoWired让配置自动扫描包中的容器，使用@Autowired就能实现了

### ByName自动装配

```xml
<bean id="cat" class="com.xx.pojo.Cat"/>
<bean id="dog" class="com.xx.pojo.Dog"/>
<!--自动装配bean-->
<!-- byName:会自动在容器上下文中查找,和自己对象set方法后面的值对应的beanid, 满足自动装配(首字母一定为小写)-->
<!-- setDog(Dog dog),setCat(Cat cat)-->
<bean id="user" class="com.xx.pojo.User" autowire="byName">
    <property name="name" value= "xxx" />
    <!-- autowire="byName" 等价于
    <property name="cat" ref= "cat"/>
    <property name="dog" ref= "dog" />
	-->
</bean>

```

### ByType自动装配

```xml
<bean class="com.xx.pojo.Cat"/>
<bean id="dog111" class="com.xx.pojo.Dog"/>
<!--自动装配bean-->
<!-- byType:会自动在容器上下文中查找,和自己对象属性类型相同的bean, 出现两个相同类型的bean会报错,即类型全局唯一.
并且类型的id可以省略，byType根据class进行操作
-->
<!-- setDog(Dog dog),setCat(Cat cat)-->
<bean id="user" class="com.xx.pojo.User" autowire="byType">
    <property name="user" value= "mysql" />
    <!-- autowire="byType" 等价于
	<property name="cat" ref= "cat"/>
    <property name="dog" ref= "dog" />
	-->
</bean>
```

### 注意

byname：需要保证所有bean的id唯一(起别名貌似可以)，并且这个bean需要和自动导入的set方法名的后面的值一致，并且首字母一定为小写

bytype：需要保证所有bean的class唯一， 并且这个bean需要和自动注入的属性的类型一致!

## 使用注解进行自动装配*

基于注释的配置的引入提出了一个问题，即这种方法是否比XML“更好”。

使用注解需要导入context约束，直接输入<<context:annotation-config/>> 就会自动配置好约束=。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:context="http://www.springframework.org/schema/context"
xmlns:aop="http://www.springframework.org/schema/aop"
xsi:schemaLocation="http://www.springframework.org/schema/beans            					https://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/context
https://www.springframework.org/schema/context/spring-context.xsd
https://www.springframework.org/schema/aop
https://www.springframework.org/schema/aop/spring-aop.xsd     
">
    <!--开启注解支持-->
	<context:annotation-config/>
</beans>
```

### @Autowired

对类成员变量、方法及构造函数进行标注，完成自动装配的工作。

用来给指定的字段或方法注入所需的外部资源(bean/对象)。

直接在属性上使用即可! 也可以在set方式上使用，Autowired默认是bytype，resource默认是byname

注意：

> [Autowire](https://www.jb51.net/article/157767.htm)优先按类型Bytype，找不到就报错，如果匹配到类型的多个实例，再通过byName来确定Bean。若还是找不到，就报错。先type，type不唯一去找name，都无的话报错。
>
> 当注入在IoC容器中该类型只有一个时，就通过byType进行装配。
>
> 当注入容器存在多个同一类型的对象时，就是根据byName进行装配。

```xml
<bean id="cat" class="com.xx.pojo.Cat"/>
<bean id="dog" class="com.xx.pojo.Dog"/>
<!--在注解中@Component 代替bean的注入-->
<bean id="user" class="com.xx.pojo.User">
    <property name="user" value= "xxx" />
    <!--在注解中 @Autowired 代替自动装配
	<property name="cat" ref= "cat"/>-->
</bean>
```

User.java

使用@Autowired很简单，在需要注入的属性加入注解即可。

```java
public class User{
    // 等价于autowire="byType" 
    // 并且可以忽略set方法，不写set
    // 创建两个对象属性 cat 和 dog
    @Autowired
    private Cat cat;
    @Autowired
    private Dog dog; 
    private String name;
}
```

注意：

XML和注解都是反射！！！ 注解是反射直接给属性赋值，xml的是反射获得getset方法赋值

#### @Nullable

```java
private String name;
// 属性字段可以为空，字段标记了这个注解，说明这个字段可以为nu11;
public User(@Nullable String name){
    this.name = name;
}
```

#### required

```java
//如果显示定义了Autowired的required属性为false,说明这个对象可以为null，否则不允许为空
// required = false 代表如果找不到装配的 不抛出异常。正常情况找不到装配的对象,直接抛空指针异常。
@Autowired(required = false)
private Cat cat;
```

### @Qualifier

类型重复，使用qualifier来确定具体的哪一个

@Autowired查找候选者：按类型找->通过限定符@Qualifier过滤->@Primary->@Priority->根据名称找（字段名称或者方法名称）

使用@Qualifier(value="xx")去配置@Autowired的使用，指定一个唯一的bean对象注入!

```java
@Autowired
@Qualifier(value="cat11")
// 指定为cat11。
private Cat cat;
```

```xml
<bean id="cat" class="com.xx.pojo.Cat"/>
<bean id="cat11" class="com.xx.pojo.Cat"/> 
```

### @Resource

```java
@Resource(name="cat11")
// 指定为cat11。
private Cat cat;
@Resource
// 默认cat,@Resource是java的注解,默认是byName与byType
private Cat cat;
```

```xml
<bean id="cat" class="com.xx.pojo.Cat"/>
<bean id="cat11" class="com.xx.pojo.Cat"/> 
```

### 区别

@Autowire优先按类型byType，找不到就报错，找到多个则按名字ByName若还是找不到，就报错。

如果多个类型重复，使用qualifier来确定具体的哪一个。

@Resource先通过byName方式实现，没有找到则通过byType实现，找不到就报错

## 使用注解开发

通过扫描类路径来隐式检测候选组件的选项

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans >
    <context:component-scan base-package="org.example"/>
    <!--使用<context:component-scan>隐式启用的功能 <context:annotation-config>,即<context:component-scan>会自动开启注解支持-->
</beans>
```

`@Component`，`@Service`，和 `@Controller`。`@Component`是任何Spring托管组件的通用构造型。

### Bean的注入*

@Component是把类(组件)装载到spring容器中。

@Component 通常是通过类路径扫描来自动侦测以及自动装配到Spring容器中。

作用于类上，只有在我们的SpringBoot应用程序启用了组件扫描并且包含了被注解的类时才有效。

```java
// 等价于bean.xml里面的<bean id="user" class="com.xx.pojo.User"/> , bean.xml就可以不用写注入类信息
// @Component 组件
@Component
public class User{
    public String name = "xxxx";
}
```

```java
User user = context.getBean("user",User.class); // 默认类名首字母小写
// 获取配置文件的组件，即Spring创建的对象
user.name; // 结果为xxxx 
```

### 属性注入

@Value

```java
@Component
public class User{
    @Value("xxxx") // 相当于 <property name="name" value="xxxx"/>
    // 简单的用注解，复杂的用xml，例如数组，列表等等
    public String name;  // 结果为xxxx
    /*
    // 效果和上面一样
    @Value("xxxx") 
    public void setName(String name){
        this.name = name;
    } */
}
```

### 衍生注解

@Component注解表明一个类会作为组件类，并告知Spring要为这个类创建bean。

@Component有几个衍生注解，按照三处架构分层

- dao[ @Repository ]
- service[ @service ]
- controller[ @Controller]

功能都一样：将某个类注册到Spring中，装配Bean，标注在一个类上的，作用是将被标注的类注册在spring容器中，将类的实例化交给spring管理

@Component系列是为了创建对象，@autowried为了赋值

### 作用域

@Scope

@Scope("singleton")

@Scope("prototype")

### 小结

xml与注解最佳实践

- xml用来管理bean;
- 注解只负责完成属性的注入;
- 我们在使用的过程中，只需要注意- 一个问题:必须让注解生效,就需要开启注解的支持

扫描包包含了注解驱动，只要配扫描包就行了

## 使用java注解配置Spring

JavaConfig--配置类

### @Configuration

com.xxx.Config.java

```java
@Configuration
public class Config {
    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

上一`Config`类等效于Spring `<beans/>`XML，*Spring*配置信息 所有的*Spring* Bean信息都是定义在*Config*文件或者Configuration的配置类中的。

这个也会Spring容器托管，注册到容器中，因为他本来就是一个@**Component** , @Configuration代表这是一个配置类， 就和我们之前看的beans.xml

注意：

1. 如果开启包扫描，加载配置类以后就可以通过反射拿到配置类中的对象了
2. @Bean只写在方法上，返回的是一个对象，但一般不获取已经在容器中的对象
3. @Bean 可以用于通过方法获取数据库连接池Connection这种对象

### @ComponentScan

@ComponentScan value:指定要扫描的包

excludeFilters=Filter[]:指定扫描包的时候按照什么规则排除哪些组件

includeFilters=Filter[]:指定扫描包的时候要包含哪些组件,需将useDefaultFilters置false

```java
@Configuration
@ComponentScan("com.xx.pojo")  // 扫描包
@Import(XXXConfig.class) // 导入其他配置文件(xml,config)
public class Config {
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

### @Bean

@Bean注解告诉Spring这个方法将会返回一个对象，这个对象要注册为Spring应用上下文中的bean。通常方法体中包含了最终产生bean实例的逻辑。它可以独立加在方法或者类上。

```java
public class Config {
    //注册一个bean,就相当于我们之前写的一个bean标签
	//这个方法的名字，就相当于bean标签中的id属性
	//这个方法的返回值，就相当于bean标签中的class属性
    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```



总结：

@Configuration相当于把这个类变成了配置文件你在里面用

@Bean相当于写配置文件里的<bean>标签加上

@ComponentScan相当于你在配置文件中开启了注解扫描



**使用@Configuration声明配置类时有两种方法来生成[Bean](https://zhuanlan.zhihu.com/p/145852514)**

​		方法1:在配置类中定义一个方法，并使用@Bean注解声明，使用@Bean的话bean的id就是标注的方法名，可以在使用@Bean(name="")来设置id，**@Bean作用于方法。**

​		方法2:在类上使用@Component注解，表明一个类会作为组件类，并在配置类上声明@ComponentScan("User类的路径")，这样会自动扫描@Component并生成Bean。**@Component 作用于类**

> 使用第二种方法的话，因为已经是自动扫描，所以getUser那个方法可以不用写了，然后再把test的getBean()的内容改为小写的类名,比如user。
>
> ​    	两种都使用的话getBean("user")和getBean(getUser)获得的对象是同一个对象，打印了hashcode是一样的。
>

### 总结

@Configuration 注解，表名当前类是一个配置类

@ComponentScan 注解，替代 context:component-scan

@PropertySource，引入外部属性配置文件

@Import 引入其他配置类

@Value 对变量赋值，可以直接赋值，也可以使用 ${} 读取资源配置文件中的信息 

@Bean 将方法返回对象加入 SpringIOC 容器



### 实例化

```java
public class Test{
    public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(Config.class);
    MyService myService = ctx.getBean(MyService.class);
    // MyService myService = ctx.getBean("getmyService");
    myService.doStuff();
	}
}
```



## IOC[高级特性](https://www.yuque.com/barry-wjxgk/bw7hkp/fhxslh#e8Uat)

### lazy-Init 延迟加载

Bean的延迟加载(bean对象的延迟创建)

ApplicationContext 容器的默认行为是在启动服务器时将所有 ***singleton bean\*** 提前进行实例化。提前 实例化意味着作为初始化过程的一部分，ApplicationContext 实例会创建并配置所有的singleton bean。

```xml
<bean id="bean" calss="com.xxx.LazyBean" lazy-init="false" />
```

lazy-init="false"，立即加载，表示在spring启动时，立刻进行实例化。

对于被修饰为lazy-init的bean Spring 容器初始化阶段不会进行 init 并且依赖注入，当第一次进行getBean时候才进行初始化并依赖注入 

对于非懒加载的bean，getBean的时候会从缓存里头获取，因为容器初始化阶段 Bean 已经 初始化完成并缓存了起来

### springBean生命周期

![springBean生命周期](https://cdn.nlark.com/yuque/0/2020/png/1919001/1599838251522-f51284be-e325-43ce-b66c-a52f2cbff384.png?x-oss-process=image%2Fresize%2Cw_1612)



## AOP

[aspectj](https://www.baeldung.com/spring-aop-vs-aspectj)

​	AOP (Aspect Oriented Programming)意为:面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术

​	AOP是OOP的延续,是软件开发中的一个热点,也是Spring框架中的一个重要内容,是函数式编程的一种衍生范型。

​	利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

### AOP在Spring中作用

提供声明式事务;允许用户自定义切面

- 横切关注点：跨越应用程序多个模块的方法或**功能**。即是，与我们业务逻辑无关的，但是我们需要关注的部分，就是横切关注点。如日志,安全,缓存,事务等等....
- 切面(ASPECT) :横切关注点被模块化的特殊对象。即，它是一个**类**。
- 通知(Advice) :切面必须要完成的工作。即，它是类中的一**方法**。
- 目标(Target) :被通知对象。
- 代理(Proxy) :向目标对象应用通知之后创建的对象。
- 切入点(PointCut) :切面通知执行的“地点”的定义。
- 连接点(JointPoint) :与切入点匹配的执行点。

横切关注点就是我们想在当前类添加的一些业务功能；切面就是把这些业务抽象成一个类；通知就是类里面的方法

AOP在不改变原有代码的情况下，添加新的功能。

#### [图解](https://www.jianshu.com/p/bb1f7a410e2c)

![img](E:\Markdown\JAVA\Spring.assets\21013181-7ea93bab57c91a98.png)

target：目标类，需要被代理的类。也就是上图的UserService类。

JointPoint：连接点，指可能被拦截到的方法。例如：所有的方法。

PointCut：切入点，已经被增强的方法。例如：addUser()

> 怎么记连接点和切入点：
>
> 比如：洗手间所有的马桶就是连接点，正在被使用的马桶就是切入点

advice：通知/增强，增强的代码。例如：before()、after()

Weaving：织入，把增强advice应用到目标对象target来创建新的代理对象Proxy的过程。

Proxy：代理类

aspect：切面，切入点和通知的结合

### 实现AOP

#### @AspectJ支持

[重点]使用AOP织入，需要导入一个依赖包!

```xml
<dependencies>
<!-- https ://mvnrepository. com/artifact/org. aspectj/aspectjweaver -->
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.9.4</version>
    </dependency>
</dependencies>
```



#### 例子

##### 写入方法

service.UserService.java

```java
public interface UserService{
    public void add();
    public void delete();
    public void update();
    public void select();
}
```

service.UserServiceImpl.java

```java
public interface uUserServiceImpl implements UserService{
    public void add(){ xx };
    public void delete(){ xx };
    public void update(){ xx };
    public void select(){ xx };
}
```

##### 使用Spring的API接口

写一个日志增强类

log.BeforeLog.java

```java
import org.springframework.aop.MethodBeforeAdvice;

// 执行前
public class BeforeLog implements MethodBeforeAdvice{
    // Method:要执行的目标对象的方法
    // args：参数
    // target：目标对象
    public void before (Method method, Object[] args, Object target) throws Throwable{
     	System.out.println(target.getClass().getName() + "执行" + method.getName() + "方法"); 
    }
}
```

log.AfterLog.java

```java
import org.springframework.aop.AfterReturnAdvice;
// 执行后
public class AfterLog implements AfterReturnAdvice{
    // returnValue：返回值
	public void afterReturning (Object returnValue, Method method, object[] args, object target) throws Throwable {
        System.out.println(method.getName() + "方法返回了" + returnValue + "结果"); 
    }
}
```

将上述方法添加到Spring中

resource.applicationContext.xml

```xml
<beans>
	<!--注册bean-->
    <bean id="userService" class="com.xx.service.UserServiceImpl" />
    <bean id="beforeLog" class="com.xx.log.BeforeLog" />
    <bean id="afterLog" class="com.xx.log.AfterLog" />
    <!--配置AOP：需要导入AOP约束-->
    <aop:config>
        <!--execution 要执行的位置! (*(修饰词) *(返回值) *(类名) *(方法名) *(参数))-->
    	<aop:pointcut id="pointcut" expressio="execution(* com.xx.service.UserServiceImpl.*(..))"/>
       <!--执行环绕增加,将log切入到上面的方法中,切入点可以理解为被增强的方法 要扩展功能的那个方法-->
       <aop:advisor advice-ref="beforeLog" pointcut-ref="pointcut"/>
        <aop:advisor advice-ref="afterLog" pointcut-ref="pointcut"/>
    </aop:config>
</beans>
```

 spring AOP[配置中的正则表达式](https://blog.csdn.net/Z_Y_J_1_2_3/article/details/23599921)

[正则表达式的学习](https://deerchao.cn/tutorials/regex/regex.htm)

Test

```java
@Test
public void test(){
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml"); 
	// 动态代理代理的是接口
    // 相当于返回的是被创建的代理类，代理类与被代理类都实现的是UserService接口
    // 这里获得的是代理类，是spring创建的，是userservice的一个新的实现类，所以只能这样写。
    UserService userService = context.getBean("UserService",UserService.class);
    hello.getStr();
}
```

  

##### 自定义类实现AOP

其实aop就是将核心代码和非核心代码进行分离，在核心代码中切入非核心代码，主要项目中就是完成事物和日志的记录的。

diy.DiypointCut.java

```java
public class DiypointCut{
    public void before(){ }
    public void after(){ }
}
```

resource.applicationContext.xml

```xml
<beans>
	<!--注册bean-->
    <bean id="diypointCut" class="com.xx.diy.DiypointCut" />
    <bean id="beforeLog" class="com.xx.log.BeforeLog" />
    <bean id="afterLog" class="com.xx.log.AfterLog" />
    <!--配置AOP：需要导入AOP约束-->
    <aop:config>
        <!--切面是一个类-->
        <aop:aspect ref="diy">
            <!--切入点-->
            <!--service.*代表service下的所有类-->
            <aop:pointcut id="pointcut" expressio="execution(* com.xx.service.*.*(..))"/>
            <!--通知,自定义的方法-->
            <aop:before method="before" pointcut-ref="point" />
            <!--method直接指向了自定义的方法-->
            <aop:before method="after" pointcut-ref="point" />
        </aop:aspect>
    </aop:config>
</beans>
```



##### 总结

方式一：使用原生Spring API接口  配置文件里只说明用哪几个通知，根据类的定义（实现）去分配具体的功能（before/after)。

方式二：自定义类 配置文件里说明切面类的某个方法是做什么的，这样就不用限制类本身性质了。



### 注解AOP

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.After;
// 在这个切面类上 使用@Compent的方式，来创建bean,不能成功完成切面
@Aspect
// 标注这是一个切面
public class AnnotationPointCut {
    @Before("execution(* com.xx.service.UserServiceImpl.*(..))")
    public void before(){ }
    
    @After("execution(* com.xx.service.UserServiceImpl.*(..))")
    public void After(){ }
    
    // 在环绕增强中，我们可以给定一个参数,代表我们要获取处理切入的点:
    @Around("execution(* com.xx.service.UserServiceImpl.*(..))")
    // 新版的环绕在before之前执行,after之后执行
    // 环绕,ProcessingJoinPoint 连接点
    public void around(ProcessingJoinPoint jp){
        System.out.println("环绕前");
        //获得签名,执行方法的详细信息 void com.xx.service.UserService.xx()
        Signature signature = jp.getSignature();
        System.out.println("signature: " +signature);

        // 过滤方法，proceed相当于invoke方法，@Around=@Before+方法执行+@After
    	Object proceed = jp.proceed();
        
        System.out.println("环绕后");
        
    }
}
```

resource.applicationContext.xml

```xml
<beans>
	<!--注册bean-->
    <bean id="annotationPointCut" class="com.xx.diy.AnnotationPointCut" />
    <!--开启注解支持 JDK(默认)--false  cglib--true -->
    <!--如果目标类是不实现接口的，则必须用cglib-->
    <aop:aspectj-autoproxy proxy-target-class= "false"
/>
</beans>    
```

#### @[pointcut](https://cloud.tencent.com/developer/article/1655923)

有@pointcut(execution(  ) )注解修饰方法名，这样写其他通知的时候直接写这个方法名就可以了。

重复时，可以使用@Pointcut( )复用。

想了解如何使用 aop实现 权限控制 不是拦截器

### CGLIB（Code Generator Library）

[cglib](https://www.cnblogs.com/loveer/p/11588126.html)是一个强大的、高性能的代码生成库。 其被广泛应用于AOP框架（Spring）中，用以提供方法拦截操作。

CGLIB代理主要通过对字节码的操作，以控制对象的访问。

底层使用了ASM（一个短小精悍的字节码操作框架）来操作字节码生成新的类。

CGLIB相比于JDK动态代理更加强大：JDK动态代理虽然简单易用，但只能对接口进行代理。    如果要代理的类为一个普通类，没有接口，那么Java动态代理就没法使用了。

JDK动态代理使用Java原生的反射API进行操作（运行期），在生成类上比较高效。 CGLIB使用ASM框架直接对字节码进行操作（编译期），在类的执行过程中比较高效。

> 1. JDK动态代理– Spring AOP的首选方式。每当目标对象实现一个接口时，都将使用JDK动态代理
> 2. CGLIB代理–如果目标对象未实现接口，则可以使用CGLIB代理

![preview](E:\Markdown\JAVA\Spring.assets\v2-e777957e808c92fefcbcbec3945a2f91_r.jpg)



## AOP原理

### 为什么叫做面向切面编程

![image.png](E:\Markdown\JAVA\Spring.assets\1599284248591-d1a04573-50f4-41e1-a23b-3901575d7ff3.png)

「切」:指的是横切逻辑，原有业务逻辑代码我们不能动，只能操作横切逻辑代码，所以面向横切逻辑

「面」:横切逻辑代码往往要影响的是很多个方法，每一个方法都如同一个点，多个点构成面，有一个面的概念在里面。

### 源码解析

[Spring源码分析之AOP从解析到调用](https://www.jianshu.com/p/f14a9ef2382e)



## 整合Mybatis*

### 导入相关jar包

- junit
- mybatis
- mysq|数据库，spring-jdbc，spring-tx
- spring相关的
- aop织入
- mybatis-spring [new]

### My batis

- 编写实体类pojo
- 编写核心配置文件，注册<mapper class>
- 编写接口
- 编写xml，具体操作语句
- 测试

### [Mybatis-Spring](http://mybatis.org/spring/zh/factorybean.html)

mybatis-config.xml

```xml
<!--configuration核心配置文件-->
<configuration>
    <!--类型别名-->
    <typeAliases>
    	<package name="com.xxx.pojo"/>
    </typeAliases>
    <!--设置
	<settings>
	-->
    
    <!--被mapperLocations代替
    <mappers>
    	<mapper class=" com.xxx.mapper.UserMapper"/>
    </mappers>-->
</ configuration>
```

UserMapper.xml

```xml
<mapper namespace= "com.xxx.mapper.UserMapper">
    <select id="selectUser" resultType="user">
    	select * from mybatis.user;
    </select>
</mapper>
```

UserMapper.java

```java
public interface UserMapper{
    public List<User> selectUser;
}
```



需要在 Spring 应用上下文中定义至少两样东西：一个 `SqlSessionFactory` 和至少一个数据映射器类。

spring-dao.xml

```xml
<!--DataSource:使用Spring的数据源替换Mybatis的配置, resource.mybatis-config.xml, c3pθ dbcp druid -->
<!--datasource数据源就是mybatis连接数据库的操作，然后通过sqlsessionfactory来获取数据源的一些配置-->
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <!--Mysql8以上是 <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>-->
	<property name="driver" value="com.mysql.cj.jdbc.Driver"/>
	<property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
    <property name="username" value="root"/>
    <property name="password" value="123456"/>
</bean>

<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!--注入上面的配置的数据源-->
    <property name="dataSource" ref="dataSource" />
	<!--绑定Mybatis配置文件-->  
    <!--可以替代mybatis-config.xml配置文件中的信息-->
    <property name="configLocation" value="classpath:mybatis-config.xml"/>
    <!--写sql语句的地方,替换mybatis-config.xml中的<mappers>-->
    <property name="mapperLocations" 
value="classpath:com/xxx/mapper/*.xml"/>
</bean>

<!--sqlSessionTemplate就是SqlSession,这个步骤代替mybatis的MyBatisUtils工具类-->
<bean id="sqlSession" class="org.mybatis.spring .SqlSessionTemplate">
	<!--只能使用构造器注入sqlSessionFactory,因为它没有set方法-->
	<constructor-arg index= "0" ref="sqlSessionFactory"/>
</bean>

```

applicationContext.xml

```xml
<import resource="spring-dao.xml" />
<bean id="userMapper" class="com.xx.mapper.userMapperImpl">
	<property name="sqlSessionTemplate" ref="sqlSessionTemplate"/>
</bean>
```

Mapper要调用mapper的方法，但是mapper是个接口，没有实现类，这边要给他建一个实现类来让mapper去调用。spring无法直接注册接口只能注册实现类，Mapper层的实现类：

```java
public class UserMapperImpl implements UserMapper {
    //我们的所有操作,都使用sqlSession来执行,在原来,现在都使用sqlSessionTemplate;
    private SqlSession sqlSession;
    public void setSqlSession(sqlSession sqlSession){
        this.sqlSession = sqlSession;}
    
    public List<User> selectUser() {
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        return mapper.selectUser();
    }
}
```

可以不用写实现类，直接从容器中获取template，通过template获取UserMapper就行

把对象都交由spring管理，不再像之前test一样，需要先获取sqlsession，然后再获取usermapper，最后再调用。就是说不用手动创建SqlSession对象了，使用Spring简化了。

Test

```java
public class Test{
    @Test
    public void test01() throws IOException {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserMapper userMapper = context.getBean("userMapper", UserMapper.class);
        for (User user : userMapper.selectUser()){
            System.out.println(user);
        }
    }
}
```

### SqlSessionDaoSupport

**此方法在SpringBoot更简单，你可以跳过**

`SqlSessionDaoSupport` 是一个抽象的支持类，用来为你提供 `SqlSession`。调用 `getSqlSession()` 方法你会得到一个 `SqlSessionTemplate`，之后可以用于执行 SQL 方法。

```java
public class UserMapperImpl extends SqlSessionDaoSupport implements UserMapper {
  public List<User> selectUser() {
      UserMapper userMapper = getSqlSession().getmapper(UserMapper.class);
      return userMapper.selectUser();
  }
}
```

```xml
<bean id="userDao" class="org.mybatis.spring.sample.dao.UserDaoImpl">
      <property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>
```

之后使用MapperScannerConfigurer 在xml生成这个类，指定包名，自动会创建所有接口的实现类对象。

## 声明式事务

原子性，一致性，持久性，隔离性（幻读，脏读，不可重复读）

ACID分别是原子性，一致性，隔离性，持久性

隔离性 : 数据库允许多个并发事务同时对其数据进行读写和修改的能力, 但要排除其他事务对本次事务的影响。

### transaction

```xml
<!--开启 Spring 的声明式事务处理功能-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
      <constructor-arg ref="dataSource" />
</bean>

<!--结合AOP实现事务的织入-->
<!--配置事务通知-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
	<!--给哪些方法配置事务--> 
	<!--配置事务的传播特性: new Propagation-->
    <tx:attributes >
        <tx:method name="add" propagation="REQUIRED" />
        <tx:method name="delete" propagation="REQUIRED" />
        <tx:method name="update" propagation="REQUIRED" />
        <!--read-only 只读-->
        <tx:method name="query" read-only="true"/>
		<!--给那些所有方法配置事务--> 
        <tx:method name="*" propagation="REQUIRED" />
    </tx:attributes>
</tx:advice>
<tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true"/>


<!--配置事务切入-->
<aop:config>
    <aop:pointcut id= "txPointCut" expression="execution(* com.xxx.mapper.*.*(..))"/>
    <aop:advisor advice-ref= "txAdvice" pointcut-ref= "txPointCut"/>
</aop:config>
```

注意：

- 传播很重要，面试要考
- MyISAM不支持事务，InnoDB可以
- mysql中引擎默认为MyISAM，是不支持事务操作的。
- 事务使用必须导入aop织入包 

为什么需要事务?
- 如果不配置事务，可能存在数据提交不一致的情况下;
- 如果我们不在SPRING中去配置声明式事务，我们就需要在代码中手动配置事务!
- 事务在项目的开发中十分重要，设计到数据的一致性和完整性问题，不容马虎!



### Propagation 传播

事务的传播propagation属性

- required(默认) 有事务则用,没事务则创建事务
- not supported 不论有无事务,都不开启
- requeres_new 不论有无事务,都创建启动新事务
- mandatory 有事务则开启事务,没事务抛出异常
- supports 有事务则开启事务,没事务就不用事务
- never 没事务正常运行,有事务则抛出异常
- nested(嵌套) 内部事务嵌套在外部事务中,如果没有则创建事务,只对DataSourceTransactionManager事务管理器有效

[spring 事务传播行为Propagation详解](https://www.jianshu.com/p/34bc1c5be703)

requeres_new：

1.标志REQUIRES_NEW会新开启事务，外层事务不会影响内部事务的提交/回滚
2.标志REQUIRES_NEW的内部事务的异常，会影响外部事务的回滚





## [元注解](https://blog.csdn.net/zhou920786312/article/details/81133284)

### 元注解meta-annotation

#### @Target

　　**作用：用于描述注解的使用范围（即：被描述的注解可以用在什么地方）**

　　**取值(ElementType)有：**

　　　　1.CONSTRUCTOR:用于描述构造器
　　　　2.FIELD:用于描述域，成员变量
　　　　3.LOCAL_VARIABLE:用于描述局部变量
　　　　4.METHOD:用于描述方法
　　　　5.PACKAGE:用于描述包
　　　　6.PARAMETER:用于描述参数
　　　　7.TYPE:用于描述类、接口(包括注解类型) 或enum声明

使用： @Target(ElementType.TYPE)


#### @Retention

​	**作用：表示需要在什么级别保存该注释信息，用于描述注解的生命周期（即：被描述的注解在什么范围内有效）**

　　**取值（RetentionPoicy）有：**

　　　　1.SOURCE:在源文件中有效（即源文件保留）
　　　　2.CLASS:在class文件中有效（即class保留）
　　　　3.RUNTIME:在运行时有效（即运行时保留）

使用：
@Retention(RetentionPolicy.RUNTIME)

#### @Documented

@Documented用于描述其它类型的annotation应该被作为被标注的程序成员的公共API，因此可以被例如javadoc此类的工具文档化。Documented是一个标记注解，没有成员。

#### @Inherited

　@Inherited 元注解是一个标记注解，@Inherited阐述了某个被标注的类型是被继承的。如果一个使用了@Inherited修饰的annotation类型被用于一个class，则这个annotation将被用于该class的子类。



### 自定义注解

public @interface 注解名 {定义体}



```java
@Target(ElementType.TYPE) // 标注在类的声明上
@Retention(RetentionPolicy.RUNTIME) // 保存在生成的字节码文件中
@Documented // 将注解的信息包含在javadoc生成的文档中
@Configuration // 配置类
@ComponentScan // 扫描包
public @interface MyConfiguraton {
    /**
     * 传入的value参数，会覆盖元注解@ComponentScan注解的value上
     * @return
     */
    String[] value() default {};
}
```

```java
@MyConfiguraton()
public class MyConfig {
}
```






















































































































