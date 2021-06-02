# Mybatis-4.26

Mybatis持久层框架：持久层： dao层  主要与数据库进行交互

mybatis避免了几乎所有的jdbc代码和手动设置参数以及获取结果集

1. 优化了JDBC存在硬编码的问题
2. 解决频繁的释放数据库连接，耗费内存资源的问题
3. 优化sql语句参数存在硬编码的问题
4. 手动封装结构集，比较繁琐

**hibernate**

**mybatis-plus**不用自己打，mapper层接口写好后会提示自动补充xml

## 简介

### 持久化

持久化就是将数据的数据在持久状态(数据库)和瞬时状态(每次)转换的过程。

### 持久层

完成持久化的代码，层界限十分明显，高内聚低耦合

### Mybatis

sql与代码分离，通过Dao层，将业务逻辑与数据访问逻辑进行继续分离。

## Mybatis

### 搭建步骤

创建Maven项目，删除src目录，搭建父工程，导入依赖：mysql+mybatis+junit

创建Module，Maven项目

在Maven中的在build中配置resources，才能将Mapper.xml文件导出到服务器。在Mybatis核心配置文件中注册才能生效。查看Maven笔记，静态资源过滤问题。

在src下的**resource**创建mybatis-config.xml：核心配置文件<configuration>，<transcaionManager type="JDBC"/>事务管理。

resource.mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
  <!--每一个Mapper.xml都要在Mybatis核心配置文件中注册，即映射器（mappers），这些配置会告诉 MyBatis 去哪里找映射文件，注意mybatis-config.xml文件里不能写注释-->
  <mappers>
    <mapper resource="com/xx/dao/UserMappper.xml"/>
  </mappers>
</configuration>
```

使用IDEA进行与数据库连接

> 注意：连不上的在url后边加上?serverTimezone=GMT%2B8。
>
> 我的也是，用useSSL=true就报错，数据库拒绝连接，改为false就可以。
>
> ```xml
> <property name="url" value="jdbc.mysql://localhost:3306/数据库名?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=GMT%2B8"/>
> <!--或者useSSL=false&amp;serverTimezone=UTC-->
> ```

SqlSessionFactory

每一个Mybatis都是以一个SqlSessionFactory的实例为核心，SqlSessionFactory实例可以通过SqlSessionFactoryBuilder获得，SqlSessionFactoryBuilder可以从XML配置文件的Configuration的实例构建SqlSessionFactory实例。

```javascript
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource); // 读取配置文件
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream); // build加载流文件构建工厂
```

从SqlSessionFactory中获取获取SqlSession实例，SqlSession完全包含面向数据库执行的SQL命令所需的所有方法，从SqlSession实例直接执行已映射的SQL语句

编写工具类com.xxx.utils.MyBatisUtils.java

```java
// sqlSessionFactory--> sqlSession
public class MyBatisUtils{
    public static SqlSessionFactory sqlSessionFactory;
    
    // 加载工厂类
    static{
        try{
            // 获取SqlSessionFactory对象，通过工厂创建。
            String resource = "mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource); // 读取配置文件
sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOExcaption e){
            e.printStackTrace();}
    } 
    
    // 创建一个可以执行sql的对象
    public static SqlSession getSqlSession(){
        // SqlSession sqlSession = sqlSessionFactory.openSession();
        // return sqlSession;
        return sqlSessionFactory.openSession();
    }
    
}
```

### 编写代码

原来的步骤：

- 实体类--POJO--User接口

- Dao接口--操作数据库对象实体--UserDao--Mapper

- > 推荐一个插件eastcode,一键生成dao
  >
  > 下载个lombok，一个@Date写完，但是有问题

- 接口实现类--impl

现在的步骤：

Maven

在utils里编写工具类MyBatisUtils.java

sqlSession执行sql语句，原来是使用JDBC写实现类，现在是通过对象调用sqlSession，调用**XML**或者注解。mapper.xml就是impl

接口实现类由原来的UserDaolmpl转化为一个Mapper配置文件

com.xxx.dao.UserMapper.xml (在Mybatis核心文件中注册)

```xml
<!-- namespace = 绑定一个对于Dao/Mapper接口，原理就是实现UserDao接口，相当于java反射中class中的forname，知道怎么去哪里找对应的接口-->
<mapper namespace="com.xx.dao.UserDao">
    <!--id对应的方法名字-->
    <!--resultMap返回集合，resultType返回一个，resultType会对结果集封装-->
	<select id="getUserList" resultType="com.xxx.pojo.User">
        select * from School.Student
    </select>    
</mapper>
```

com.xxx.dao.UserDao.java

```java
public interface UserDao{
    List<User> getUserLsist(); // 集合写泛型，全限定类名
}
```

com.xxx.pojo.User.java  -- 实体类，get/set方法

编写测试类

一个快捷写测试类的方法，Dao接口里面右键，选择go to，再点test，然后create new test

test.com.xxx.dao.UserDaoTest.java

[JDK的动态代理](https://blog.csdn.net/jiankunking/article/details/52143504)，就是在程序运行的过程中，根据被代理的接口来动态生成代理类的class文件，并加载运行的过程。

```java
public class UserDaoTest{
    // 获取SqlSession对象，工具类
    try (SqlSession sqlsession = MybatisUtils.getSqlSession()){
        // 方式1    
        // 执行sql
        // 获取sql语句，即Dao的实现类--Mapper.xml
        // 使用接口反射得到具体的实现类方法--Mapper.xml
        // 因为java文件和xlm文件已经绑定了 可以算是一个东西
        // 类似多态，面向接口编程  接口名称  对象名  =  new  实现类名称（）
        // 返回一个自动生成的实现类--动态代理类
        UserDao userDao = sqlSession.getMapper(UserDao.class);
        // 使用userDao可以进行调用方法
        List<user> userList = userDao.getUserLsist();


        // 方式2 <返回Object进行强转>
        // List<User> userList = sqlsession.selectList("com.xxx.dao.UserDao.getUserList");  // 根据方法的返回值进行限制，例如：selectOne，select返回一个。

        for (User user:userList){
            Systrm.out.println(user)
        }
        
    }catch (Exception e) {
        e.printStackTrace();
    }
}
```

### try-with-resource

**[try-with-resource](https://blog.csdn.net/weixin_42447959/article/details/81192098)**  : 语法糖，之前讲过，可以直接进行执行close不用写finally，其底层仍然是try-catch-finally，不过在catch(){}代码块中有一个addSuppressed()方法，即异常抑制方法。



### 原理

![在这里插入图片描述](E:\Markdown\JAVA\Mybatis.assets\8af08d678be8e81b6d33a9f42ef8d115.png)

1. DefaultSqlSessionF调用getMapper方法，其中为configuration下的getMapper方法
2. configuration下的getMapper方法，其中为mapperRegistry下的getMapper方法
3. mapperRegistry下的getMapper方法下根据接口类型获取MapperProxyFactory
4. MapperProxyFactory调用newInstance生成MapperProxy
5. 创建MapperProxy的代理对象，一路返回到最初调用的DefaultSqlSession

[运行原理](https://www.cnblogs.com/jian0110/p/9452592.html)



## CRUD

###  namespace

namespace中的包名要和Dao/mapper接口的包名一致

### select

id：就是对应的namespace中的方法名

我个人理解为namespace为实现的接口，id就是重写接口的方法

resultType：sql执行的返回值类型，一般在实体类里面

parameterType：参数类型

com.xxx.dao.UserMapper(Dao).java

```java
public interface UserMapper{
    // 根据ID查询用户
    User getUserById(int id);
    User addUser(User user);
}
```

com.xxx.dao.UserMapper(Dao).xml

```xml
<mapper namespace="com.xxx.dao.UserMapper">
    <select id="getUserById" parameterType="int" resultType="com.xxx.pojo.User">
    	select * from 数据库.表名 where id = #{id}
    </select>
    <insert id="addUser" parameterType="com.xxx.pojo.User">
    	insert into 数据库.表名 (id, name, pwd) values (#{id}, #{name}, #{pwd})
        <!-- pojo里面的变量名 = #{ 写pojo里面的变量名 } ,前面也一致 -->
    </insert>
</mapper>
```

再写测试类就行，注意增删改查必须要提交事务sqlSession.commit();

*MySQL8.0可以不用commit*

**[mybatisplus](https://www.cnblogs.com/leeego-123/p/10734000.html)** 啥都不用写

### Map

 使用 

```java
int add(Map<String, object> map);
```

```xml
<insert id="add" parameterType="map" >
	insert into 数据库.表名 (id, name, pwd) values (#{xxid}, #{namxxe}, #{pwxxd}) 
    <!-- pojo里面的变量名 = #{ 自定义的变量名，与测试类一致 } -->
</insert>
```

```java
Map<String, Object> map = new HashMap<String, Object>();
map.put("xxid", 5);
map.put("namxxe", "xx");
map.put("pwxxd", "xx");
// 这样可以不用一次修改全部属性，可以针对不同属性进行学习，没有map前，因为User类中构造器设置了传入参数的个数，必须全部传参，也可以设置个别参数，但不够灵活
// 测试类给dao层传参数的时候new的user所有的参数必须有，而map不需要
// 阿里的JAVA开发建议是强制不使用map
// 而且用map的话如果传输数据非法会报错的
mapper.add(map);
```

### 模糊查询

where like"李%"

#### 传递通配符

where like #{value}

List<User> userList = mapper.getUserLiske("%李%");

#### 使用通配符代替上面的操作

where like concat('%', #{value}, '%')     // 用#防止sql注入

List<User> userList = mapper.getUserLiske("李");

## 配置解析

### 核心配置文件

myBatis-config.xml，在xml配置都有顺序

### 环境配置（environments）

MyBatis 可以配置成适应多种环境

**尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。**

- 默认使用的环境 ID（比如：default="development"）。
- 每个 environment 元素定义的环境 ID（比如：id="development"）。
- 事务管理器的配置（比如：type="JDBC"）。
- 数据源的配置（比如：type="POOLED"）。

> 在 MyBatis 中有两种类型的事务管理器（也就是 type="[JDBC|MANAGED]"）
>
> 如果你正在使用 Spring + MyBatis，则没有必要配置事务管理器，因为 Spring 模块会使用自带的管理器来覆盖前面的配置。

> 三种内建的数据源类型（也就是 type="[UNPOOLED|POOLED|JNDI]" ，默认POOLED

### 属性（properties）

通过properties属性来实现引用配置文件

这些属性可以在外部进行配置，并可以进行动态替换。你既可以在典型的 Java 属性文件中配置这些属性，也可以在 properties 元素的子元素中设置。[db.properties]

​		首先读取在 properties 元素体内指定的属性 .最后读取作为方法参数传递的属性，并覆盖之前读取过的同名属性。然后根据 properties 元素中的 resource 属性读取类路径下属性文件，或根据 url 属性指定的路径读取属性文件，并覆盖之前读取过的同名属性。

编写db.properties

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<!--引入外部配置文件-->
    <properties resource="db.properties">
    	<property name="username" value="dev_user"/>
  		<property name="password" value="F2Fa3!33TYyg"/>
    </properties>
    
    <!--property元素体内的属性优先，在resource导入文件中遇到同名属性会覆盖-->

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                  <property name="driver" value="${driver}"/>
                  <property name="url" value="${url}"/>
                  <property name="username" value="${username}"/>
                  <property name="password" value="${password}"/>
    </dataSource>   
        </environment>
    </environments>
</configuration>
```

> 1db.properties 里面的配置不能有引号和空格,
>
> 2有的版本url链接需要用& amp;,
>
> 3引用的时候键要填写正确

### 类型别名（typeAliases）

[1.类型别名](https://blog.csdn.net/fageweiketang/article/details/80776883)可为 Java 类型设置一个缩写名字。 建议使用使用全类名。

```xml
<!--使用前-->
resultType="cn.com.mybatis.pojo.User"
<typeAlias alias="user" type="cn.com.mybatis.pojo.User"/>
<!--使用后-->
resultType="user"
```

```xml
<typeAliases>
      <typeAlias alias="Author" type="domain.blog.Author"/>
</typeAliases>
```

2.也可以指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean，扫描实体类的包，为每一个实体类添加一个别名，默认别名为类的类名，首字母小写

```xml
<typeAliases>
      <package name="com.xxx.pojo"/>
</typeAliases>
```

3.若有注解，则别名为其注解值。见下面的例子：

```java
import org.apache.ibatis.type.Alias;
@Alias("author")
public class Author {     ...  }
```

### 设置

![image-20210429155125017](E:\Markdown\JAVA\Mybatis.assets\image-20210429155125017.png)

![image-20210429154653511](E:\Markdown\JAVA\Mybatis.assets\image-20210429154653511.png)

### 懒加载

[懒加载](https://blog.csdn.net/shfqbluestone/article/details/52853460)功能使用了代理对象，所以在调用懒加载属性的 get/set 方法（或者是其他触发懒加载操作的方法）时 mybatis 才能知道这时候应该去加载懒加载属性。

懒加载通俗的讲就是按需加载，我们需要什么的时候再去进行什么操作。而且先从单表查询，需要时再从关联表去关联查询，能大大提高数据库性能，因为查询单表要比关联查询多张表速度要快。

mybatis 框架用 javassist 为懒加载对象创建了代理对象。

```xml
<!-- 开启懒加载配置 -->
<settings>
    <!-- 全局性设置懒加载。如果设为‘false'，则所有相关联的都会被初始化加载。 -->
    <setting name="lazyLoadingEnabled" value="true"/>
    <!-- 当设置为‘true'的时候，懒加载的对象可能被任何懒属性全部加载。否则，每个属性都按需加载。 -->
    <setting name="aggressiveLazyLoading" value="false"/>
</settings>
```

### 其他配置

插件

### 映射器（mappers）

​	映射器其实就是一个动态代理对象，进入到MapperMethod的execute方法就能简单找到SqlSession的删除、更新、查询、选择方法，从底层实现来说：通过动态代理技术，让接口跑起来，之后采用命令模式，最后还是采用了SqlSession的接口方法（getMapper()方法等到Mapper）执行SQL查询（也就是说Mapper接口方法的实现底层还是采用SqlSession接口方法实现的）

#### 使用class文件绑定注册：

接口和Mapper配置文件必须同名

接口和Mapper配置文件可以不同包的，放到resources下面的同名包mapper或者dao。如果想分离，只需要在resource下建和接口所在相同的包。因为这样编译后class文件就会和xml在同一个包下。

#### 扫描包

```xml
<!-- 将包内的映射器接口实现全部注册为映射器 -->
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```

### 生命周期

不同作用域和生命周期类别是至关重要的，因为错误的使用会导致非常严重的并发问题。

![image-20210429161205862](E:\Markdown\JAVA\Mybatis.assets\image-20210429161205862.png)

#### SqlSessionFactoryBuilder

一旦创建了 SqlSessionFactory，就不再需要它了。 因此 SqlSessionFactoryBuilder 实例的最佳作用域是方法作用域（也就是局部方法变量）。 

#### SqlSessionFactory

- 可以理解为数据库连接池
- SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，**没有任何理由丢弃它或重新创建另一个实例**。
- 多次重建 SqlSessionFactory 被视为一种代码“坏习惯”。
- 最简单的就是使用单例模式或者静态单例模式。

#### SqlSession

- sqlsession也就是一种类型的session
- sqlsession就是连接池的一个连接请求。
- 最佳的作用域是请求或方法作用域。用完就关闭，否则资源被占用。
- 一个sqlsession可以多次使用它的getMapper方法,获取到多个mapper接口实例，一个mapper代表一个具体的业务。

#### [SqlSession原理](https://www.cnblogs.com/jian0110/p/9452592.html)

## Resultmap结果映射集

解决属性名和字段名不一致的问题：

起别名：pwd ad password

### Resultmap

结果集映射：找到自己实现的标签去映射

`ResultMap` 的设计思想是，对简单的语句做到零配置，对于复杂一点的语句，只需要描述语句之间的关系就行了。

```xml
<!--将User实体类指定为UserMap-->
< resultMap id="UserMap" type="User">
    <!--column数据库字段，property实体类属性-->
    <result column="id" property="id"/>
    <result column="name" property="name"/>
    <result column="pwd" property="password"/>
</resultMap>
<select id="getUserId" resultMap="UserMap">
	select * from xxx.xx where id = #{id}
</select>
```

 上述语句只是简单地将所有的列映射到 `HashMap` 的键上，这由 `resultType` 属性指定。

## 日志 5.7

### 日志工厂 

异常，排错

日志实现和日志门面。而且推荐使用SLF4J +LOG4J2

#### STDOUT_LOGGING

配置mybatis-config.xml

```xml
<settings>
	<setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
```

#### log4j

导包--Maven--pom.xml

进行log4j-properties配置

配置实现

```xml
<settings>
	<setting name="logImpl" value="LOG4J"/>
</settings>
```

#####  使用

```java
// import org.apache.log4j.Logger;

public void testLog4j{
    // 定义属性,UserDaoTest类
    Logger logger = Logger.getLogger(UserDaoTest.class);
    logger.info(); // 信息
    logger.error(); // 
}
```

## 分页

### Limit

```java
HashMap<String, Integer> map = new HashMap<String, Integer>();
// 分页数据
map.put("startIndex", 1);
map.put("pageSize", 2);
// 查询分页接口
List<User> userList = mapper.getUserByLimit(map);
```

Mapper.xml

```xml
<select id="getUserByLimit" parameterType="map" resultMap="UserMap">
	select # from xx.xxx limit #{startIndex}, #{pageSize}
</select>
```

实际中分页都是pageHelper

### pageHelper

**在 pom.xml 中添加如下依赖：**

```xml
<dependency>
<groupId>com.github.pagehelper</groupId>
<artifactId>pagehelper</artifactId>
<version>最新版本</version>
</dependency>
```
**Mapper接口方式的调用，推荐这种使用方式。**
**PageHelper.startPage(1, 10);**
**List<Country> list = countryMapper.selectIf(1);**

使用pagehelper非常的简单，只需要在你想要分页的查询逻辑前添加一行代码即可，代码：

PageHelper.startPage(pageNum, pageSize);

pageNum：页数（第几页）

pageSize：每页的数据行数



## 使用注解进行开发

面向接口编程，解耦。

接口：

```java
public interface UserMapper{
    @Select("select * from user")
    List<User> getUsers();
}
```

配置文件：mybatis-config.xml

```xml
<!--绑定接口-->
<mappers>
	<mapper class = "com.xx.dao.UserMapper"/>
</mappers>
```

底层核心使用反射+动态代理。

MapperProxyFactory、newInstance、mapperProxy、invoke() 动态代理

JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；这种动态获取信息以及动态调用对象方法的功能称为java语言的反射机制。

### @param

```java
public interface UserMapper{
    // 方法存在多个参数，每一个参数必须加上@param("id")
    // 引用类型不加
    @Select("select * from user where id=#{id}")
    List<User> getUserById(@param("id" int id)  );
}
```

#{} ${} 区别：

${}代表statement，#{}代表preparestarement

${}方式无法防止SQL注入。传入数据会直接显示在生成的sql中。



## Mybatis详细过程

- Resource获取全局配置文件

- 实例化SqlSessionFactoryBuilder构造器

- 解析配置文件流XMLConfigBuilder

- Configuration所有的配置信息，解析的所有信息

- SqlSessionFactory实例化

- transactional事务管理

- 创建executor执行器，

  - 简单执行器SimpleExecutor：执行事务，缓存，查询栈，close

  - 事务缓存TransactionalCacheManager

- 创建sqlSession----包括Configuration+executor+事务自动提交+dirty+cursorList

- 实现CRUD

- 成功进行提交事务，关闭

执行器去执行mapper，mapper---sqlSession+反射加载的类信息(sql，由接口(CRUD)读出)

## lombok

[为什么不用BeanUtils](https://mp.weixin.qq.com/s/KQzwujnVCLf62gwIPZRpwA)

lombok工程是一个java库,这个库可以自动的插入你的编辑器和构建工具,简化释放你的java。

不需要另外写getter和equals方法,只需要在你的类上有一个注解就可以完全构建  自动记录变量。

安装插件

使用Maven导入

在实体类上加注解，@Data。

## 多对一*

###  sql约束

![image-20210510150108966](E:\Markdown\JAVA\Mybatis.assets\image-20210510150108966.png)

多个学生一个年级

```mysql
CREATE TABLE IF NOT EXISTS `student`(
	`id` INT(4) NOT NULL AUTO_INCREMENT COMMENT '学号',
    `name` VARCHAR(30) NOT NULL DEFAULT '匿名' COMMENT '学号',
	`gradeid` INT(4) NOT NULL  COMMENT '学生年级',
    PRIMARY KEY(`id`),  
    KEY `FK_gradeid` (`gradeid`), #外键--grade的主键
    CONSTRAINT `FK_gradeid` FOREIGN KEY (`gradeid`) REFERENCES `grade`(`gradeid`)# 约束
    # KEY 约束名 (引用列) 
    # CONSTRAINT 约束名 FOREIGN KEY (当前引用列) REFERENCES 引用表(引用列)
)ENGINE=INNODB DEFAULT CHARSET=utf8
```

一般一对多关系都由多端来负责维护关系，在写pojo时，外键关联应该写关联的实体类。

pojo.Student.java

```java
public class Student{
    private int id;
    private String name;
    private Grade grade;  // 关联年级，年级实体类
}
```

一个表对应一个pojo(JavaBean )类，一个pojo对应一个Mapper，一个Mapper对应一个xml文件

dao(mapper).StudentMapper.java

```java
public interface StudentMapper{
    
}
```

rescource.dao(mapper).StudentMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<mapper namespace="com.xx.dao.StudentMapper">
	<select id="getUserList" resultType="com.xxx.pojo.User">
        select * from School.Student
    </select>    
</mapper>
```

配置mybatis-config.xml

做项目orm映射的实体bean一般都是和结果集对应不是和表对应的。不关心你来自哪个表只关心你查出来的数据。

### 关联的嵌套 Select 查询

对于查询多对一关系表，查询外键学习，首先获取学生信息，通过查询的学生信息的gradeid，查询年级信息。但是由于java中年级是一个对象，因此涉及到复杂查询，通过对查询出来的学生信息进行重新赋值，使用resultMap，在查询复杂属性时，将属性和其他查询语句进行连接。获取到年级类的查询之后的信息，添加到学生查询之后的信息里面。

即：通过学生信息查出来的年级ID，再去数据库查该年级ID的信息，然后放到学生对象的属性里

**复杂查询：对象：association；集合：collection**

StudentMapper.xml

```xml
<select id="getStudent" resultMap="StudentGrade">
    <!--子查询-->
	select * from stduent
</select>
<resultMap id="StudentGrade" type="Student">
    <result property="id" column="id" />
	<result property="name" column="name"/>
    <association property="grade" column="gradeid" javaType="Grade" select="getGrade">
        <!--gradeid被设置为对应嵌套 Select 语句的参数。-->
    </association>
</resultMap>

<select id="getGrade" resultMap="Grade">
	select * from grade where id = # {gradeid}
</select>
```

这种方式虽然很简单，但在大型数据集或大型数据表上表现不佳。这个问题被称为“N+1 查询问题”。 概括地讲，N+1 查询问题是这样子的：

- 你执行了一个单独的 SQL 语句来获取结果的一个列表（就是“+1”）。
- 对列表返回的每条记录，你执行一个 select 查询语句来为每条记录加载详细信息（就是“N”）。

虽然，MyBatis 能够对这样的查询进行延迟加载，因此可以将大量语句同时运行的开销分散开来。 然而，如果你加载记录列表之后立刻就遍历列表以获取嵌套的数据，就会触发所有的延迟加载查询，性能可能会变得很糟糕。

### 关联的嵌套结果映射

```xml
<select id="getStudent" resultMap="StudentGrade">
    <!--联表查询-->
	select s.id sid, s.name sname, g.name gname 
    from stduent s, grade g 
    where s.gradeid = g.id
</select>
<resultMap id="StudentGrade" type="Student">
    <result property="id" column="sid" />
	<result property="name" column="sname"/>
    <association property="grade" javaType="Grade">
        <result property="name" column="gname"/>
    </association>
</resultMap>
```

注意：结果查询不能懒加载。

## 一对多

```xml
<resultMap id="blogResult" type="Blog">
  <id property="id" column="blog_id" />
  <result property="title" column="blog_title"/>
  <collection property="posts" ofType="Post">
    <id property="id" column="post_id"/>
    <result property="subject" column="post_subject"/>
    <result property="body" column="post_body"/>
  </collection>
</resultMap>
```



### 注意

> javaType：指定实体类中的属性类型
>
> ofType：指定映射到List或者集合中的pojo类型，泛型中的约束类型。

## 动态sql*

==根据不同条件生成不同的SQL语句==

### 小技巧

驼峰命名转换

mybatis-config.xml

```xml

<settings>
	<setting name="logImpl" value="STDOUT_LOGGING"/>
    <!--开启驼峰命名转换-->
    <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```

ID随机

IDutils.java

```java
@SuppressWarnings("all") //抑制警告
public class IDutils{
    public static Sting getId(){
        return UUID.randomUUID().toString().replaceAll("-",""); 
    }
    @Test
    public void test(){
        System.out.println(IDutils.getId());
    }
}
```

### IF

```xml
<select id="BlogWithTitle"
     parameterType="map" resultType="Blog">
  SELECT * FROM mybatis.blog
  WHERE 1=1 
    <!--1=1是为了where后所有判断条件都不符时，依然可以执行查询而不报错.
1=1是为了条件中and关键字能够进行作用为拼接条件-->
  <if test="title != null">
    AND title = #{title}
  </if>
   <if test="author != null">
    AND author = #{author}
  </if>
</select>
```

BlogMapper.java

```java
public interface BlogMapper{
    List<Blog> BlogWithTitle(Map map);
}
```

后台测试：

```java
SqlSession sqlSession = MybatisUtils.getSqlSession();
BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);
HashMap map = new HashMap();
map.put("author","xxx");
List<Blog> blogs = mapper.BlogWithTitle(map);
for (Blog blog:blogs){
    System.out.println(blog);
}
```

### choose、when、otherwise



```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM  mybatis.blog
  <WHERE >
      <choose>
          <!--满足第一个就会返回，和switch中的break一样-->
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
   </WHERE>
</select>
```

### trim、where、set

*<where>* 元素只会在子元素有返回内容的情况下才插入 “WHERE” 子句。而且，若子句的开头为 “AND” 或 “OR”，**<where>* * 元素也会将它们去除。

用于动态更新语句的类似解决方案叫做 *set*。*set* 元素可以用于动态包含需要更新的列，忽略其它不更新的列。即去掉连接的最后一个逗号。 

```xml
<update id="updateAuthorIfNecessary">
  update Author
    <set>
      <if test="username != null">username=#{username},</if>
        <!--当map.username("bio","xxx");时，bio，则自动去掉连接的最后一个逗号-->
        <!--当map.put("bio","xxx");时，没有更新username，则跳过这条语句-->
      <if test="bio != null">bio=#{bio}</if>
    </set>
  where id=#{id}
</update>
```

**trim**

通过自定义 trim 元素来定制 *where* 元素的功能。

### foreach

遍历collection，将元素拼接成一个条件串

声明可以在元素体内使用的集合项（item）和索引（index）变量。它也允许你指定开头与结尾的字符串以及集合项迭代之间的分隔符。

```xml
<!--select * from mybatis.blog where 1=1 and (id=1 or id=2 or id=3)
select * from mybatis.blog where 1=1 and id in (1,2,3)
-->
<select id="selectPostIn" parameterType="map" resultType="domain.blog.Post">
  SELECT * from mybatis.blog
<where>
    <!--传递map集合-->
    <!--sql语句不区分空格-->
  <foreach item="id"  collection="list"
      open="and (" separator="or" close=")">
        id = #{id}
  </foreach>
</where>
</select>
```

```java
SqlSession sqlSession = MybatisUtils.getSqlSession();
BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);
HashMap map = new HashMap();

ArrayList<Integer> list = new ArrayList<Integer>();
list.add(1);
list.add(2);
map.put("list",list);
List<Blog> blogs = mapper.BlogWithTitle(map);
for (Blog blog:blogs){
    System.out.println(blog);
}
```

### 总结

==动态SQL本质还是SQL，只是再SQL层面，执行一个逻辑代码。==

==动态SQL就是不同的条件生成不同的语句，拼接SQL语句==

   

## SQL片段

```xml
<sql id="username-bio">
	<if test="username != null">username=#{username},</if>
     <if test="bio != null">bio=#{bio}</if>
</sql>
<!--将一些复用的语句进行封装，提高复用性-->
<select id="Author">
  select * from blog
    <where>
        <include refid="username-bio"></include>
    </where>
</update>
```

使用SQL标签抽取公共部分，使用include进行引用。

最好基于单表来定义SQL片段，不要存在where标签。

## 缓存 5.11

读写分离，主从复制

减少和数据库交互次数，减少系统开销。

### 一级缓存

默认，SqlSession级别的缓存，称为本地缓存，只在一次SqlSession有效，源码里就是一个Map。

与数据库同一次会话期间查询的数据会放在本地缓存中，之后获取相同的数据之间从缓存中取。

开启日志

```xml
<settings>
	<setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
```

缓存失效：

1.查询不同的东西
2.增删改操作，可能会改变原来的数据，所以必定会刷新缓存
3.查询不同的Mapper.xml
4.手动清理缓存

```java
sqlSession.clearCache(); // 手动清除缓存
```



### 二级缓存

手动开启，namespace级别的缓存。二级缓存也叫全局缓存

SQL 映射文件中添加一行：

```xml
<cache/>
```

二级缓存是跨SqlSession的。基于namespace级别的缓存。

==namespace==

- Liunx：这个*namespace*里面,用户是具有root权限的。
- 在关系数据库系统中，命名空间namespace指的是一个表的逻辑分组，同一组中的表有类似的用途。
- K8S：资源隔离+权限控制

，一个名称空间，对应一个二级缓存;

工作机制

- 一个会话查询一条数据，这个数据就会被放在当前会话的一-级缓存中;

- 如果当前会话关闭了，这个会话对应的一级缓存就没了;但是我们想要的是，**会话关闭了，一级缓存中的数据被保存到二级缓存中;**

- 新的会话查询信息，就可以从二级缓存中获取内容;

- 不同的mapper查出的数据会放在自己对应的缓存(map)中;
  

FIFO缓存：先进先出策略

可用的清除策略有：

- `LRU` – 最近最少使用：移除最长时间不被使用的对象。
- `FIFO` – 先进先出：按对象进入缓存的顺序来移除它们。
- `SOFT` – 软引用：基于垃圾回收器状态和软引用规则移除对象。
- `WEAK` – 弱引用：更积极地基于垃圾收集器状态和弱引用规则移除对象。



```xml
<!--显示开启的全局缓存-->
<setting name="cacheEnable" value="true"/>
```

```xml
<!--在当前Mapper. xmL中使用二级缓存-->
<cache
      eviction="FIFO"
      flushInterval="60000"
      size="512"
      readOnly="true"/>
<!--readOnly为True时是读相同实例，false时是读拷贝值，相对安全，所以会出现比较时出现false-->
```

#### 技巧

```xml
<select id="queryUserById" resultType="user" useCache="false"> 
    <!--更新比较频繁，关闭缓存-->
    select * from user where id = #{id}
</select>
<update id="updateUser" parameterType="user" flushCache="false">
    <!--更新时，不刷新缓存-->
    update mybatis .user set name=#{ name}, pwd=#{pwd} where id = #{id};
</update>
</mapper>

```



注意：

> 我们需要将实体类序列化!否则就会报错!
> Caused by: java. io. NotSerializableException: com. kuang. pojo.user
>
> 序列化是因为实体在二级缓存中，所以需要序列化的处理才能方便查找。
>
> 将对象转为字节流存储到硬盘上，当JVM停机的话，字节流还会在硬盘上默默等待，等待下一次JVM的启动，把序列化的对象，通过反序列化为原来的对象，并且序列化的二进制序列能够减少存储空间（永久性保存对象）。

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User implements Serializable {
    private int id;
    private String name;
    private String pwd;
}
```

### 小结:

- 只要开启了二级缓存,在同一个Mapper下就有效
- 所有的数据都会先放在一-级缓存中;
- 只有当会话提交，或者关闭的时候，才会提交到二级缓冲中!

### 原理

用户→二级缓存→一级缓存→数据库

二级缓存作用域：一个mapper.xml文件，就是namespace作用域

![image-20210511173925382](E:\Markdown\JAVA\Mybatis.assets\image-20210511173925382.png)

### 自定义缓存-ehcache

java分布式缓存，面向通用缓存





























































