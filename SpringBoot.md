# SpringBoot

6/1

**约定大于配置**

dubbo是个RPC框架，是服务间通信的一种解决方案

微服务：高内聚，低耦合

## 第一个SpringBoot

自动装配，配置多个启动器

打包：在maven工程里，点击项目下的Lifecycle，直接点击package

设置启动图像

## 自动装配原理

狂

pom.xml

- spring-boot-dependencies :核心依赖在父工程中!
- 我们在写或者引入一些Springboot依赖的时候，不需要指定版本，就因为有这些版本仓库。

启动器：spring-boot-starter，springboot启动创建，自动导入环境依赖，将所有的功能场景变为启动器。

### 主程序注解

SpringBootApplication：标注为springboot的应用

SpringApplication.run(xxx.class, args); ：将springboot应用启动

#### SpringBootApplication

```java
@SpringBootConfiguration  // 表示这是一个SpringBoot的配置类
@EnableAutoConfiguration // 开启自动配置功能
@ComponentScan // 自动扫描并加载符合条件的组件或者bean ， 将这个bean定义加载到IOC容器中
public @interface SpringBootApplication {}
```

#### SpringApplication

1、推断应用的类型是普通的项目还是Web项目

2、查找并加载所有可用初始化器 ， 设置到initializers属性中

3、找出所有的应用程序监听器，设置到listeners属性中

4、推断并设置main方法的定义类，找到运行的主类

结论: 

springboot所有自动配置都是在启动的时候描并加载: spring. factories所有的自动配置类都在这里面，但是不一定生效，要判断条件是否成立，只要导入了对应的start，就有对应的启动器了，有了启动器，我们自动装配就会生效,然后就配置成功!

## 自动装配原理

[雷](https://www.yuque.com/atguigu/springboot/qb7hy2)

### 依赖管理

父项目声明了常用的依赖版本号。无需关注版本号，自动版本仲裁

修改默认版本号：

```xml
1、查看spring-boot-dependencies里面规定当前依赖的版本 用的 key。
2、在当前项目里面重写配置
    <properties>
        <mysql.version>5.1.43</mysql.version>
    </properties>
```

启动器：starter场景启动器，SpringBoot所有支持的场景

```xml
5、所有场景启动器最底层的依赖
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
  <version>2.3.4.RELEASE</version>
  <scope>compile</scope>
</dependency>
```

### 自动配置

#### 配置项

##### 配置SSM

自动配好SpringMVC

自动配置，前置控制器DispatcherServlet，字符编码拦截器characterEncodingFilter，视图解析器

##### 扫描包

主程序;

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan("com.xxx.boot")
// 以上三个等同于SpringBootApplication
public class MainApplication {
    public static void main(String[] args) {
        //1、返回我们IOC容器
        ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class, args);

        //2、查看容器里面的组件
        String[] names = run.getBeanDefinitionNames();
        for (String name : names) {
            System.out.println(name);
        }
```



主程序所在包(com.xxx.hello)及其下面的所有子包里面的组件都会被默认扫描bean进来

但是如果不在主程序所在包(com.xxx.hello)的目录里面，运行则需要改变扫描路径，

```java
@SpringBootApplication(scanBasePackages="com.xxx") //即(com.xxx.hello)的父目录
```

或者@ComponentScan 指定扫描路径

##### 各种配置拥有默认值

默认配置最终都是映射到某个类上，如：MultipartProperties

配置文件的值最终会绑定每个类上，这个类会在容器中创建对象

修改默认配置，更改application.properties文件

```properties
访问路径
server.port = 8888 
```

##### 按照需求加载包，只有用到才会去加载

### 容器

### 组件添加

组件可以理解为可拆分的小工具实例，比如你一个维修工具套装里的小锤子，id为它的方法名，返回类型就是锤子。

组件(使用Spring来创建对象，在Spring这些都称为**Bean**)就是生成的具体的实体对象，里面封装了方法和数据，它们都放在spring容器中统一管理。使用getBean可以获取组件。

因为是从ioc容器里面得到的组件  这个组件在容器里面只有一个，是单例模式

#### @Configuration

configuration 是 spring注解 它标注的类会使用cglib

CJlib动态代理的实现方法之一，只可以代理类，不可以代理接口。

代理设计模式，即通过一个对象来调用真实对象的方法，目的是保护真实对象

```java
/*
proxyBeanMethods：代理bean的方法
    Full(proxyBeanMethods = true)、【保证每个@Bean方法被调用多少次返回的组件都是单实例的】。单例模式
    Lite(proxyBeanMethods = false)【每个@Bean方法被调用多少次返回的组件都是新创建的】轻量级，不会去检测。原型模式。
    组件依赖必须使用Full模式默认，使用Full模式来保证取得的组件为ioc中的同一组件，使用其他组件的方法，得到的组件方法hashcode一致。其他默认是否Lite模式
    应用场景就是：以前使用spring在xml中配置嵌套对象，如mybatis，你会配置sql工厂和sqlSession，然后嵌套Session到工厂，这个Session就是容器中的
    */
@Configuration(proxyBeanMethods = false) //告诉SpringBoot这是一个配置类 == 配置文件
// 配置类里面使用@Bean标注在方法上给容器注册组件，默认也是单实例的
// 配置类本身也是组件
```

#### @Bean

配置类中使用

@Bean //给容器中添加组件。以方法名作为组件的id（组件名）。返回类型就是组件类型。返回的值，就是组件在容器中的实例。@Bean作用于方法。

#### @Component

在bean(pojo)等类上使用@Component注解，表明一个类会作为组件类，这样会自动扫描@Component并生成Bean。@Component 作用于类。

#### @Import

给容器中自动创建出这两个类型的组件、默认组件的名字就是全类名

```java
@Import({User.class, DBHelper.class})
```

####  @Conditional

条件装配：满足Conditional指定的条件，则进行组件注入

项目的类路径中存在某些资源的时候才会XXX

##### 组件依赖

组件依赖时，依赖组件不存在，则当前组件不注入

```java
@ConditionalOnBean(name = "xxx")
```

同时，如果依赖组件的代码在当前组件的下方时，当前组件也未加载。即**组件有先后注册的问题**

@Conditional注解写在类上时，只有当容器中存在了才会加载该内中的容器。

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan("com.xxx.boot")
public class MainApplication {

    public static void main(String[] args) {
        //1、返回我们IOC容器
        ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class, args);

        //2、查看容器里面的组件
        String[] names = run.getBeanDefinitionNames();
        for (String name : names) {
            System.out.println(name);
        }

        //3、从容器中获取组件
        Pet tom01 = run.getBean("tom", Pet.class);

        //3、从容器中组件(bean)是否存在
        boolean tom = run.containsBean("tom"); // 返回true/false
```



### 原生配置文件引入

#### @ImportResource

```java
// 引入xml文件的组件
@ImportResource("classpath:beans.xml")
```

### 配置绑定

以前的操作：配置数据库配置文件，解析配置文件到JavaBean(可重用组件，符合一定规范编写的Java类，是一种规范)中。

使用Properties类，加载(FileInputStream)配置文件，遍历配置文件中的每一个K-V值，即配置key和value

SpringBoot的方法

#### @Component + @ConfigurationProperties

```java
/**
 * @Component组件放入容器中，只有在容器中的组件，才会拥有SpringBoot提供的强大功能
 */
@Component
// 与application.properties中的mycar属性对应
@ConfigurationProperties(prefix = "mycar")
public class Car {
    private String brand;
    public void setBrand(String brand) {
        this.brand = brand;
    }
}
```

application.properties

```properties
mycar.brand = BYD
```

注意：

1. 用@ConfigurationProperties注解注意要给类中属性加上惯用的setter方法，不然配置中的属性不能赋值到容器管理的对象中
2. mycar要小写

####  @EnableConfigurationProperties + @ConfigurationProperties

在配置类中写，开启属性配置功能。

EnableConfigurationProperties 来代替上面的@Component，一般是引用第三方包，第三方包没有@Component，则自己使用这样方法配置绑定

```java
@EnableConfigurationProperties(Car.class)
//1、开启Car配置绑定功能
// 就是用配置文件开启对应类的属性绑定功能
//2、把这个Car这个组件自动注册到容器中
public class MyConfig {
}
```



### 自动配置原理入门

#### 3.1、引导加载自动配置类

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication{}
```

##### @SpringBootConfiguration

底层是@Configuration。代表当前是一个配置类。

##### @ComponentScan

指定扫描哪些包，Spring注解；

##### @EnableAutoConfiguration

自动配置包下的所有组件，主类所在的包名封装成数组，进行配置

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {}
```

###### @AutoConfigurationPackage

将指定的一个包下的所有组件导入进来，MainApplication 所在包下。

###### @Import(AutoConfigurationImportSelector.class)

给容器中批量导入一些组件，从META-INF/spring.factories位置来加载一个文件。







































