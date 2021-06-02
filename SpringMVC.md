# SpringMVC

5/22

## 简介

MVC：模型 dao+service，视图  jsp/html，控制器 Servlet

model处理数据（dao和service），view（展示数据，html/jsp等），controller（servlet，获取请求和返回响应，转发和重定向(转发：你帮我做一下  重定向：这事之后就扔给你了)）

MVC：模型Model，视图View，控制器Controller

###  MVC

传统Model1

![image-20210522103843255](E:\Markdown\JAVA\SpringMVC.assets\image-20210522103843255.png)

具体细节

![img](E:\Markdown\JAVA\SpringMVC.assets\7896890-7b3f9cd59394b017.png)

弊端：

JSP 和 Java Bean 之间严重耦合，前端和后端相互依赖，代码难以复用。JSP 职责不单一。

**Model2**

![image-20210522104647259](E:\Markdown\JAVA\SpringMVC.assets\image-20210522104647259.png)

具体细节

![image-20210522105358368](E:\Markdown\JAVA\SpringMVC.assets\image-20210522105358368.png)

模型（Model）
数据模型，提供要展示的数据，包含数据和行为，就是 dao,bean，也就是模型提供了模型数据查询和模型数据的状态更新等功能，包括数据和业务。

视图（View）
网页, JSP，用来展示模型中的数据

控制器（controller)
接收用户请求，委托给模型进行处理(状态改变)， 处理完毕后把返回的模型数据返回给视图，由视图负责展示。也就是说控制器做了个调度员的工作。

MVC框架要做哪些事情

1.将url映射到java类或java类的方法。

2.封装用户提交的数据。

3.处理请求--调用相关的业务处理--封装响应数据。

4.将响应的数据进行渲染. jsp / html等表示层数据。

## SpringMVC*

Spring MVC是Spring Framework的一部分，是基于Java实现MVC的轻量级Web框架。底层也是Servlet--doService

### 例子

1.导入SpringMVC的依赖，配置web.xml，注册DispatcherServlet，匹配所有请求。

web.xml

```xml
<!--配置DispatchServlet:这个是SpringMVC的核心:请求分发器，前端控制器-->
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServ1et</servlet-class>
    <!--DispatcherServlet要绑定Spring的配置文件-->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        < param-value>classpath:springmvc-servlet.xml</param-value>
    </init-param>
    <!--启动级别：服务器一启动它就启动-->
    < load-on-startup>1</load-on-startup>
</servlet>

<!--
在SpringMVC中，/ /*
/：只四配所有的请求，不会 去匹配jsp页面
/* ：匹配所有的请求，包括jsp页面-->
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</ur1-pattern>
</servlet-mapping>
```



2.关联SpringMVC配置文件，springmvc-servlet.xml：处理映射器+适配器+视图解析器

```xml
<!--处理器映射器-->
<!--根据bean的名字找url，所以要写一个bean-->
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>

<!--处理器适配器-->
<bean class=" org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>

<!--视图解析器:DispatcherServlet给他的ModeLAndView
1.获取了ModeLAndView的数据
2.解析ModeLAnaView的视图名字
3.拼接视图名字，找到对应的视图 /WEB-INF/jsp/hello.jsp
4.将数据渲染到这个视图上-->
<!--可以自定义模板引擎Thymeleaf ，Freemarker... -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="InternalResourceViewResolver">
    <!--类似于 sql trim，<trim>是动态sql mybatis的一个标签 是where 和set的原生标签-->
    <!--前缀-->
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <!--后缀-->
    <property name="suffix" value=".jsp"/>
</bean>

<!--HandLer-->
<!--BeanNameUrLHandLerMapping: bean
BeanNameUrlHandlerMapping这个映射器会去根据输入(http://localhost:8080/hello)的url(/hello) 去匹配bean的id寻找controller
-->
<bean id="/hel1o" class="com.xxx.controller.HelloController"/>
```

3.写一个操作业务的Controller，返回ModelAndView，装数据，封视图

```java
import org.springframework.web.servlet.mvc.Controller;
//注意:这里我们先导入Controller接口
public class HelloController implements Controller {
    public ModelAndView handleRequest (HttpServletRequest request, HttpServletResponse response) throws Exception {
        //ModeLAndView模型和视图
        ModelAndView mv = new ModelAndView();
        
        //调用业务层
        String result = "HelloSpringMVC!";
        /*封装对象，放在:ModeLAndView中。Model
        // mv.addobject("msg", "HelloSpringMVC!");
         addobject(String, @Nullable Object), 第二个参数可以为空	
         */
        mv.addobject("msg", result);
        
        //封装要跳转的视图，放在ModeLAndView中，视图解析器会进行设置前缀和后缀
        mv.setViewName("hello"); //: /WEB- INF/jsp/hello.jsp
        
        return mv;
	}
}
```

```xml
<!--Handler，一个术语,它既不是类也不是接口.它负责执行映射.-->
<!--将自己的类交给SpringIOC容器，注册bean-->
<!--根据web.xml中设置的请求路径，匹配到hello，找到处理器-->
<bean id="/hello" class="com.xxx.controller.HelloController"/>
```

4.写一个hello.jsp跳转页面，显示ModelandView存放的数据。

> 注意：添加lib包，负责可能导致项目无法启动，ctrl+alt+shift+s，进入Project Structure，点击Artifacts，在WEB-INF下新建lib文件夹，添加依赖包



​	

### 原理

Spring MVC框架像许多其他MVC框架一样，以请求为驱动，围绕一个中心Servlet分派请求及提供其他功能，DispatcherServlet是一个实际的Servlet (它继承自HttpServlet基类)。

和其它web框架一样,Spring的web框架是一个请求驱动的web框架,其设计围绕一个中心的*servlet*进行,它能将请求分发给控制器,并提供其它功能帮助web应用开发。

#### SpringMVC原理

![image-20210522172519658](E:\Markdown\JAVA\SpringMVC.assets\image-20210522172519658.png)

#### 执行原理

##### [简单版本](https://www.jianshu.com/p/91a2d0a1e45a)

![image-20210522163153339](E:\Markdown\JAVA\SpringMVC.assets\image-20210522163153339.png)



##### 复杂版本

![image-20210522170920371](E:\Markdown\JAVA\SpringMVC.assets\image-20210522170920371.png)

1. DispatcherServlet表示前置控制器，是整个SpringMVC的控制中心。用户发出请求，DispatcherServlet接收请求并拦截请求。
2. 处理器映射：适配请求
   1. HandlerMapping为处理器映射。DispatcherServlet调用HandlerMapping, HandlerMapping根据请求url查找Handler。
   2. HandlerExecution表示具体的Handler,其主要作用是根据url查找控制器，如.上url被查找控制器为：hello。
   3. HandlerExecution将解析后的信息传递给DispatcherServlet，如解析控制器映射等。
3. 处理请求
   1. HandlerAdapter表示处理器适配器，其按照特定的规则去执行Handler(后端控制器)。
   2. Handler让具体的Controller执行。
   3. Controller将具体的执行信息返回给HandlerAdapter，如ModelAndView。
   4. HandlerAdapter将视图逻辑名或模型传递给DispatcherServlet.
4. 视图解析
   1. DispatcherServlet调用视图解析器(ViewResolver)来解析HandlerAdapter传递的逻辑视图名。
   2. 视图解析器将解析的逻辑视图名传给DispatcherServlet.
   3. DispatcherServlet根据视图解析器解析的视图结果，调用具体的视图(View)。

5. 最终视图呈现给用户。

注意：虚线是需要用户写的，实线是SpringMVC提供的。看完原理可以再看一边代码，将代码和步骤进行对应。<img src="E:\Markdown\JAVA\SpringMVC.assets\image-20210522180201017.png" alt="image-20210522180201017" style="zoom:25%;" />

## 注解

使用注解就可以不用配置bean文件。

@Controller注解类型用于声明Spring类的实例是一个控制器(在讲IOC时还提到了另外3个注解) (@Component @Service @Repository)

Spring可以使用扫描机 制来找到应用程序中所有基于注解的控制器类,为了保证Spring能找到你的控制器，需要在配置文件中声明组件扫描。

编写web.xml

```xml
<!--配置DispatchServlet:这个是SpringMVC的核心:请求分发器，前端控制器-->
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServ1et</servlet-class>
    <!--DispatcherServlet要绑定Spring的配置文件-->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        < param-value>classpath:springmvc-servlet.xml</param-value>
    </init-param>
    <!--启动级别：服务器一启动它就启动-->
    < load-on-startup>1</load-on-startup>
</servlet>


<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</ur1-pattern>
</servlet-mapping>
```

springmvc-servlet.xml

```xml
<!-- 自动扫描包，让指定包下的注解生效,由IOC容器统- 管理-->
<context:comiponent-scan base-package= "com.xxx.controller"/>
<!-- 让Spring MVC不处理静态资源  .css .js .html-->
<mvc:default-servlet-handler />
<!--
支持mvc注解驱动
    在spring中-般彩用@RequestMapping注解来完成映射关系
    要想使@RequestMapping注解生效
    必须向上下文中注册DefaultAnnotationHandlerMapping
    和一个AnnotationMethodHandlerAdapter实例
    这两个实例分别在类级别和方法级别处理。
    而annotation-driven配置帮助我们自动完成上述两个实例的注入。
-->
<mvc:annotation-driven />

<!--视图解析器-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="InternalResourceViewResolver">
    <!--类似于 sql trim，<trim>是动态sql mybatis的一个标签 是where 和set的原生标签-->
    <!--前缀-->
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <!--后缀-->
    <property name="suffix" value=".jsp"/>
</bean>
```

Controller

```java
// 不用配置bean文件，IOC注入
@Controller
// 类/方法前面使用
@RequestMapping("/HelloController")
public class HelloController{
    
    // url请求,项目名/HelloController/hello
    @RequestMapping("/hello")
    public String hello(Model model){
        // 封装数据，直接在jsp页面取出并渲染
        String result = "HelloSpringMVC!";
        model.addAttribute("msg", result)
            // 组成/WEB-INF/jsp/hello.jsp
        return "hello"; //会被视图解析器处理; 访问/WEB-INF/jsp/hello.jsp 文件
    }
    
    // 视图是被复用的，而控制器与视图之间是弱偶合关系。访问页面一致，但访问内容不一致
	@RequestMapping("/hello1")
    public String hello(Model model){
        // 封装数据，直接在jsp页面取出并渲染
        String result = "Hello!";
        model.addAttribute("msg", result)
            // 组成/WEB-INF/jsp/hello.jsp
        return "hello"; //会被视图解析器处理; 访问/WEB-INF/jsp/hello.jsp 文件,但访问内容不一致
    }
}
```

@Controller

代表这个类会被Spring接管。
被这个注解的类，中的所有方法，如果返回值是String，并且有具体页面可以跳转，那么就会被视图解析器解析。

@RequestMapping

@RequestMapping注解用于映射url到控制器类或一个特定的处理程序方法。可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。



### @Controller @Component @Service @Repository

#### @Component

@Component注解表明一个类会作为组件类，并告知Spring要为这个类创建bean。

@Component有几个衍生注解，按照三处架构分层

- dao[ @Repository ]
- service[ @service ]
- controller[ @Controller]

因此，当你的一个类被`@Component`所注解，那么就意味着同样可以用`@Repository`, `@Service`, `@Controller`来替代它，同时这些注解会具备有更多的功能，而且功能各异。

#### @Controller控制层

用于标记在一个类上，使用它标记的类就是一个SpringMVC Controller对象，分发处理器会扫描使用该注解的类的方法，并检测该方法是否使用了@RequestMapping注解。@Controller只是定义了一个控制器类，而使用@RequestMapping注解的方法才是处理请求的处理器。

#### @RequestMapping

它可以注解类也可以注解方法，注解类时标注请求的路径，标注方法时表示将特定的URL映射到指定的方法。

#### @Service业务逻辑层

应用于业务层，用于标注业务层组件,表示定义一个bean，自动根据bean的类名实例化一个首写字母为小写的bean。

#### @Repository持久层

用于标注数据访问组件，即DAO组件，表示将Dao类声明为bean

@Repository 只能标注在 DAO 类，因为该注解的作用不只是将类识别为Bean，同时它还能将所标注的类中抛出的数据访问异常封装为 Spring 的数据访问异常类型。

[不使用@Repository](https://blog.csdn.net/wqh0830/article/details/96109587)

#### **总结**

`@Component`, `@Service`, `@Controller`, `@Repository`是spring注解，注解后可以被spring框架所扫描并注入到spring容器来进行管理。虽然你可以全部使用`@Component`注解，但使用其他注解，则你的类更适合于通过工具进行处理或与其他方面相关联。

- `@Component`是通用注解，其他三个注解是这个注解的衍生注解，并且具有了特定的功能。
- `@Controller`层是spring-mvc的注解，用于标注控制层组件，并查看是否处理请求转发，重定向。
- `@Service`层是业务逻辑层注解，这个注解只是标注该类处于业务逻辑层。
- `@Repository`注解在持久层中，标注 DAO 类，具有将数据库操作抛出的数据访问异常自动转换(封装)为spring的持久层异常的功能。

用这些注解对应用进行分层之后，就能将请求处理，义务逻辑处理，数据库操作处理分离出来，为代码解耦，也方便了以后项目的维护和开发。

参考

[@Component, @Repository, @Service的区别](https://blog.csdn.net/fansili/article/details/78740877)



### @Controller + @ResponseBody = @RestController

简单的理解：

> 如果类上标注@Controller 且方法上没有标注@ResponseBody的话 return的值会被视图解析器处理
>
> 如果类上标注的是@RestController的话，或者在方法上标注了@ResponseBody的话，返回值就不会被视图解析器处理

复杂理解：

#### @Controller

使用@Controller 注解，注解标注类的方法，return时会被视图处理器识别成静态文件的路径。

#### @ResponseBody

[@ResponseBody](https://blog.csdn.net/jiahao1186/article/details/91980316) 将方法的返回值，以特定的格式写入到response的body区域，进而将数据返回给客户端。当方法上面**没有**写ResponseBody,底层会将方法的返回值封装为ModelAndView对象。当它标注类时，类中所有方法的返回值都将直接返回值到页面，相当于给所有的类都加上@ResponseBody注解。

如果返回值是字符串，那么直接将字符串写到客户端；如果是一个对象，会将对象转化为json串，然后写到客户端。

**使用位置**：一般在异步获取数据时使用[也就是AJAX]，在使用 @RequestMapping后，返回值通常解析为跳转路径，但是加上 @ResponseBody 后返回结果不会被解析为跳转路径，而是直接写入HTTP response body 中。

**举例**：比如异步获取 json 数据，加上 @ResponseBody 后，会直接返回 json 数据。

[好处](https://zhidao.baidu.com/question/449950170.html)是：

1. GET模式下，这里使用了@PathVariable绑定输入参数，非常适合**Restful**风格。因为隐藏了参数与路径的关系，可以提升网站的安全性，静态化页面，降低恶意攻击风险。
2. POST模式下，使用@RequestBody绑定请求对象，Spring会帮你进行协议转换，将Json、Xml协议转换成你需要的对象。
   @ResponseBody可以标注任何对象，由Spring完成对象——协议的转换。

[坏处](https://zhidao.baidu.com/question/449950170.html)是：返回之前，若前端编码格式不一致，很容易导致乱码。

#### @RestController

@RestController注解相当于@ResponseBody ＋ @Controller合在一起的作用，只能注解类。是对RESTful风格的实现

使用@RestController这个注解，直接写入HTTP相应体返回到页面中。此时return "xx"，只会返回"xx"的内容。因为@RestController中相当于已经有了@RessponseBody的注解效果，它会以Json字符串的形式返回给客户端。



### 常用注解

- `@Controller`：用于标识是处理器类

- `@RequestMapping`：请求到处理器功能方法的映射规则

- `@RequestParam`：请求参数到处理器功能处理方法的方法参数上的绑定

- `@ModelAttribute`：请求参数到命令对象的绑定

- `@SessionAttributes`：用于声明 session 级别存储的属性，放置在处理器类上，通常列出模型属性（如 @ModelAttribute）对应的名称，则这些属性会透明地保存在 session 中

- `@InitBinder`：自定义数据绑定注册支持，用于将请求参数转换到命令对象属性的对应类型

- `@CookieValue`：cookie 数据到处理器功能处理方法的方法参数上的绑定

- `@RequestHeader`：请求头（header）数据到处理器功能处理方法参数上的绑定

- `@RequestBody`：请求的 body 体的绑定（通过 HttpMessageConverter 进行类型转换）

- `@RequestStatus`：定义处理器功能处理方法/异常处理器返回的状态码和原因

- `@ExceptionHandler`：注解式声明异常处理器

- `@PathVariable`：请求 URI 中的模版变量部分到处理器功能处理方法的方法参数上的绑定，从而支持 RESTful 架构风格的 URL

- `@profile` 不同环境下使用不同的配置（开发环境，测试环境，生产环境（上线））

- `@EnableScheduling` 计划任务，实际上就相当于定时执行

- `@Condition` 根据满足某一个特定条件创建一个特定的Bean

  

## RestFul*

网页格式风格：localhost/xx/xx/xx



Restful就是一个资源定位及资源操作的风格。不是标准也不是协议，只是一种风格。基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存等机制。

使用不太的请求方式实现不同的效果。改变之前的使用路径进行实现不同的效果。

@RequestMapping("/add/{a}/{b}")

### @GetMapping

在Spring MVC中可以使用@PathVariable注解，让方法参数的值对应绑定到一个URI模板变量上。

```java
@Controller
public class RestfulController{
    //原来的:  http://locaLhost:8080/add?a=1&b=2
 	//RestFuL  http://localhost:8080/add/a/b
    // http://localhost:8080/add/1/2

    // @RequestMapping("/add/{a}/{b}")
    // @RequestMapping(value="/add/{a}/{b}", mathod=RequestMethod.GET) //指定具体的方法为Get[Delete，post...]
    @GetMapping("/add/{a}/{b}")
    // 使用Get方式提交数据
    // @PostMapping("/add/{a}/{b}")
    // @GetMapping，@PostMapping，@PutMapping，@DeleteMapping，@PatchMapping
    public String hello(@PathVariable int a, @PathVariable int b, Model model){
        int result = a + b;
        model.addAttribute("msg", result)
        return "test"; 
    }
    // 注意，多个相同get方法，出现Ambiguous错误，不明确 
```

### POST和GET区别

- GET把参数包含在URL中，POST通过request body传递参数。
- GET/POST都是TCP链接。GET产生一个TCP数据包；POST产生两个TCP数据包。
- 并不是所有浏览器都会在POST中发送两次包，Firefox就只发送一次。

### @RequestMapping

源码：

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Mapping
public @interface RequestMapping {
    // 此处name属性，相当于方法的注释，使方法更易理解 
    String name() default "";
    //  path属性，和value属性使用一致，两者都是用来作为映射使用的。
    // @AliasFor，互为别名基本的限制，value和path的值不能冲突，比如任意设置value或者设置path属性的值，都能够通过另一个属性来获取值等等。
    @AliasFor("path")
    String[] value() default {}; 
    // 标准URL映射
    @AliasFor("value")
    String[] path() default {}; 
    // 指定请求的 method 类型。例如 GET、POST、PUT、DELETE 等
    RequestMethod[] method() default {};
    // 指定 request 中必须包含某些参数值，才让该方法处理请求。
    String[] params() default {};
    // 请求 Headers 映射
    String[] headers() default {};
    // 请求类型限定
    String[] consumes() default {};
    // 设置返回类型
    String[] produces() default {};
}
```

RequestMapping是一个用来处理请求地址映射的注解，可用于类、接口货方法上，用于类或者接口上的时候，表示类中的所有相应请求的方法都是以该地址作为父路径，基本字段有name，path(value)（path 和 value 互相引用），method，params，headers，consumes，produces


#### value

@RequestMapping(value="xxx")

@RequestMapping("xxx")  // 只有应该属性时，value=可以省略

此处的value是name的别名

value的语法:

- @RequestMapping("/默认value值")
- @RequestMapping(value={"/hello1","/hello2"}) 可以根据请求响应对应的。
- 支持通配符匹配@RequestMapping(value="getUser/*")

@RequestMapping(“/login/{xxx}”)：通过@PathVariable 可以提取 URI 模版模式中的 {xxx} 中的 xx 变量的值。

URI：统一资源标识符。 URL：统一资源定位符。

[URI---URL](https://blog.csdn.net/wnx_52055/article/details/82178072) 



## 转发-重定向

### ServletAPI

通过设置ServletAPI，不需要视图解析器。

通过HttpServletResponse实现重定向

通过HttpServletRequest实现转发

### SpringMVC

无需视图解析器。

```java
@Controller
public class ResultSpringMVC {
    @RequestMapping("/rsm/t1")
        public string test1(){
        //转发
        return "/WEB-INF/jsp/test.jsp;
    }
    @RequestMapping("/rsm/t2")
        public string test2(){
        //转发二，一般使用这个
        return "forward:/WEB-INF/jsp/test.jsp ";
    }
    @RequestMapping("/rsm/t3")
        public String test3( ){
        //重定向，重定向是request, 不能访问WEB-INF目录
        return "gedirect:/index.jsp";
    }
}
```

###  [重定向和转发](https://blog.csdn.net/liubin5620/article/details/79922692)

在Servlet中实现页面的跳转有两种方式：转发和重定向

  1、转发使用的是getRequestDispatcher()方法;重定向使用的是sendRedirect();

  2、转发：浏览器URL的地址栏不变。重定向：浏览器URL的地址栏改变；

  3、转发是服务器行为，重定向是客户端行为；

  4、转发是浏览器只做了一次访问请求。重定向是浏览器做了至少两次的访问请求；

  5、转发2次跳转之间传输的信息不会丢失，重定向2次跳转之间传输的信息会丢失（request范围）。

**选择**

 1、重定向的速度比转发慢，因为浏览器还得发出一个新的请求，如果在使用转发和重定向都无所谓的时候建议使用转发。

   2、因为转发只能访问当前WEB的应用程序，所以不同WEB应用程序之间的访问，特别是要访问到另外一个WEB站点上的资源的情况，这个时候就只能使用重定向了。



5/26

### 前端接收参数

 @RequestParam

 ```java
@Controller
public class RestfulController{
    @GetMapping("/test")
    public String hello(@RequestParam(username) String name, Model model){
        model.addAttribute("msg", name)
        return "test"; 
	}
 ```

### 前端接收的是一个对象

要求提交的表单域和对象的属性名(pojo)一致，参数使用对象即可。

如果使用对象的话，前端传递的参数名和对象名必须一致，否则就是null

```java
@Controller
public class RestfulController{
    @GetMapping("/test")
    public String test1(User user, Model model){
        model.addAttribute("msg", user)
        return "test"; 
	}
```



### ModalMap前端返回

ModalMap  extends LinkedHashMap 

Modal，ModalMap，LinkedHashMap 

Model只有寥寥几个方法只适合用于储存数据，简化了新手对于Model对象的操作和理解;

ModelMap继承了LinkedMap ，除了实现了自身的一些方法，同样的继承LinkedMap的方法和特性;

ModelAndview可以在储存数据的同时，可以进行设置返回的逻辑视图，进行控制展示层的跳转。

### 乱码

get不会乱码.post才会乱码,要加一个过滤器。

get是从请求头取的，post是从请求体取的。

过滤器或者拦截器解决

SpringMVC自己提供一个过滤器，在web.xml中配置

```xml
<!--2.配置SpringMVC的乱码过滤-->
<filter>
	<filter-name>encoding</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    < init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>
</filter>

<filter-mapping>
    <filter-name>encoding</filter-name>
    <!--/* 会包含jsp页面，/ 不包含-->
    <ur1-pattern>/*</ur1-pattern>
</filter-mapping>
```



## JSON

- JSON(JavaScript Object Notation, JS对象标记)是一种轻量级的数据交换格式，目前使用特别广泛。

- 采用完全独立于编程语言的文本格式来存储和表示数据。

- 简洁和清晰的层次结构使得JSON成为理想的数据交换语言。

- 易于人阅读和编写，同时也易于机器解析和生成，并有效地提升网络传输效率。

JSON是JavaScript对象的字符串表示法，它使用文本表示一个JS对象的信息，本质是一个字符串。
```js
var obj = {a:'He1lo', b:'World'}; 
//这是一个对象，注意键名也是可以使用引号包裹的
var json = '{"a": "Hello","b":"World"}'; 
//这是一个JSON 字符串，本质是一个字符串
```

### 对象和JSON转换


```html
<script type= "text/javascript">
//编写一个JavaScript对象  ES6
    var user = {
        name:"秦疆",
        age:3,
        sex:"男"
    };
    
    //将js对象转换为json对象;
    var json = JSON.stringify(user);

	//将JSON对象转换为JavaScript对象
	var obj = JSON.parse(json);
</script>
```

### jackson

jackson是springboot自带解析器。

导入Maven包，导入依赖

Controller

```java
@Controller
public class UserController {
    @RequestMapping("/j1")
    @ResponseBody //它就不会走视图解析器，会直接返回一一个字符串
    public String json1(){
        // jackson, objec tMapper
        objectMapper mapper = new ObjectMapper(); 
        // 创建一个对象
        User user = new User( name:"xx", age:3, sex:"男");
        // writeValueAsString
        String str = mapper.writeValueAsString(user);
        return str; // 返回一个json字符串
    }
}
```

#### @ResponseBody和@RequestBody

##### `@Responsebody`

该注解表示该方法的返回的结果直接写入 HTTP 响应正文（ResponseBody）中，一般在异步获取数据时使用；

通常是在使用 `@RequestMapping` 后，返回值通常解析为跳转路径，加上 `@Responsebody` 后返回的字符串不会被解析为跳转路径，返回的字符串被解析为json格式的数据，而是直接写入HTTP 响应正文中。

该注解用于将Controller的方法返回的对象，通过适当的HttpMessageConverter转换为指定格式后，写入到Response对象的body数据区。 

`注：@ResponseBody要写在方法名上`

##### `@RequestBody`

作用是将前端传来的json格式的数据转为自己定义好的javabean对象

该注解用于读取Request请求的body部分数据，使用系统默认配置的HttpMessageConverter进行解析，然后把相应的数据绑定到要返回的对象上； 
再把HttpMessageConverter返回的对象数据绑定到 controller中方法的参数上。

`注：@RequestBody要写在方法的参数前，不能写在方法名上方`

##### `@PathVariable`

用来获取请求路径（`url`）中的动态参数。前台向后台传递的值，比如删除数据的ID号



[@ResponseBody和@RequestBody的区别](https://blog.csdn.net/shuxingcq/article/details/79616456)

### JSON乱码

在springmvc-servlet.xml添加，加了这个配置以后，响应头变成text/html;charset=UTF-8

```xml
<!--JSON乱码问题配置-->
<mvc:annotation-driven>
    <mvc:message-converters register-defaults="true">
        <bean class="org.springframework.http.conyerter.StringHttpMessageConverter">
        	<constructor-arg value= "UTF-8"/>
        </bean>
        <bean class=" org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
            <property name="objectMapper">
                <bean class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
                <property name="failOnEmptyBeans" value="false"/>
                </bean>
        	</property>
         </bean>
    </mvc:message-converters>
< /mvc:annotation-driven>
```



### 时间工具类

##### 代码复用原则

JsonUtils.java

```java
public class JsonUtils {
    public static String getJson(Object object){
    	return getJson(object, "yyyy-MM-dd HH:mm:ss");
    }
    
    public static String getJson(Object object, String dateFormat){
        ObjectMapper mapper = new ObjectMapper();
        //不使用时间戳的方式
        mapper.configure(SerializationFeature.WRITE_ DATES_ _AS_ TIMESTAMPS, false);
        //自定义日期的格式
        SimpleDateFormat sdf = new SimpleDateFormat(dateFormat);
        mapper.setDateFormat(sdf);
        
        try {
            return mapper.writeValueAsString(object);
        } catch (JsonProcessingException e) {
  	      	e.printStackTrace();
        return null;
}
```

```java
@RequestMapping("/date" )
public String json3() throws JsonProcessingException {
    Date date = new Date();
    return JsonUtils.getJson(date);
    //  return JsonUtils.getJson("yyyy-MM-dd HH:mm:ss");
}
```

### Fastjson

导入Maven包

```java
@RestController
public class UserController {
    @RequestMapping("/json1")
    public String json1(){
        // 创建一个对象
        User user = new User( name:"xx", age:3, sex:"男");
        // JSON
        String str = JSON.toJSONString(user);     
        return str; // 返回一个json字符串
    }
}
```



## [SSM整合 *](https://mp.weixin.qq.com/s/SDxqGu_il3MUCTcN1EYrng)

需求分析  设计数据库 做业务  前端界面

dao：

1. 编写mybatis配置文件将：

   1. 写入要扫描的实体类包
   2. 写入mapper
   3. 写好dao层的接口和xml

2. 编写database文件：driver，url，user，password

3. 编写连接mybatis的spring-dao.xml配置文件：

   1. 关联数据库配置文件
   2. 写入数据库连接池：c3p0,jdbc
   3. 运用SqlSessionFactoryBean加入连接池和mybatis配置文件创建SqlSessionFactory
   4. 运用MapperScannerConfigurer 加入SqlSessionFactory，和dao层的接口配置dao扫描包

   service：

   1. 编写spring-service.xml配置文件：

      1. 创建service层的包和类 
      2. 引入要扫描service层的包
      3. 注入service的实现类
      4. 写入业务管理器

   2. controller：

      1. 对项目添加web模块并写入jsp目录，对web.xml配置DispatcherServlet和静态资源过滤器

      2. 编写spring-MVC.xml配置文件：

         （非注解）

         1. 引入要扫描的controller包
         2.   配置适配器和映射器
         3. 配置视图解析器
         4. 配置静态资源过滤器

         （注解）

         1. 引入要扫描的controller包
         2. 配置注解驱动
         3. 配置静态资源过滤器
         4. 配置视图解析器

最后：将三层spring配置import进applicationContext.xml文件，添加lib

### Mybatis层

创建数据库表

```sql
CREATE DATABASE `ssmbuild`;

USE `ssmbuild`;

CREATE TABLE `books`(
`bookID` INT(10) NOT NULL AUTO_INCREMENT COMMENT '书id',
`bookName` VARCHAR(100) NOT NULL COMMENT '书名',
`bookCounts` INT(11) NOT NULL COMMENT '数量',
`detail` VARCHAR(200) NOT NULL COMMENT '描述',
KEY `bookID`(`bookID`)
)ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO `books`(`bookID`,`bookName`,`bookCounts`,`detail`) VALUES
(1,'Java',1,'从入门到放弃'),
(2,'MySQL',10,'从删库到跑路'),
(3,'Linux',5,'从进门到进牢');
```

创建Maven项目，导入依赖

```xml
<!--依赖:
junit,数据库驱动,连接池,servlet,jsp,mybatis, mybatis-spring,spring-->

<!--静态资源导出问题-->
<build>
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </ resource>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</ include>
            </includes>
            <filtering>false</filtering>
        </resource>
    </resources>
</build>
```

创建controller，dao，pojo，service包， 以及resources包

在resource下创建applicationContext.xml，mybatis-config.xml，database.properties

在pojo创建Book实体类，与数据库进行对应

在dao创建BookMapper接口，进行设置数据库CRUD的操作

在dao创建BookMapper.xml，绑定接口，实现数据库CRUD操作。

> xml不出现提示Sql数据库的信息的方法：在settings-->Languages &Frameworks-->SQL Dialects中配置数据库类型mysql，将项目添加进来，配置完成后Apply-OK
>
> 不会提示sql的，设置->Languages & Frameworks->sQL Dialects->有两个选项，选择自己对应的版本，比如我的是Mysql就两个都选MySQL

[可以在resources目录下建相同目录的包，再把mapper文件放里面]

在service创建BookService接口，BookServiceImpl实现类，service调用dao层，组合Dao层。

### Spring层

在Maven中配置mybatis-spring，c3p0依赖

applicationContext.xml

```xml
<!--1.关联spring包-->
<import resource="classpath:spring-dao.xml"/ >
<import resource="classpath:spring-service.xml"/ >  
<import resource="classpath:spring-mvc.xml"/ >  
```



在resource下创建spring-dao.xml

```xml
<!--1.关联数据库配置文件,database.properties-->
<!--2.连按池
dbcp,c3p0,druid,hikaricp
dbcp:半自动化操作，不能自动连接
c3pθ:自动化操作(自动化的加载配置文件，并且可以自动设置到对象中!)
druid: hikard-->
<!--3.sqlSessionFactory,注入数据源，绑定Mybatis的配置文件-->

<!--4.配置dao按口扫描包,动态的实现了Dao接口可以注入到Spring容器中!，因此不需要写Dao实现类，Spring通过反射,替代了实现类去注入一个sqlSession-->

<!--原来使用接口实现类，是因为接口无法注入到Spring，而这个配置文件就是注入接口！-->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
<!--MapperScannerConfigurer会扫描这个包中的所有接口，把每个接口都执行一次getMapper()方法，得到每个接口的dao对象。
service也可以整合在spring配置文件里
刚配置的扫描dao会自己创建好dao对象，所以直接在下面声明service就行。-->
	<!--注入sqlSessionFactory -->
    <property name="sqlSessionFactoryBeanName" value=" sqlSessionFactory"/>
    <!--要扫描的dao包-->
    <property name="basePackage" value="com.xxx.dao"/>
</bean>
```

### spring注入接口

spring为何要注入接口，而注入实现类就会报错？

如果只是单纯注入是可以用实现类接收注入对象的，但是往往开发中会对实现类做增强，如事务，日志等，实现增强的AOP技术是通过动态代理实现的，而spring默认是JDK动态代理，jdk动态代理只能对实现了接口的类生成代理，而不能针对类，而且还可以对实现类对象做增强得到增强类（增强类与实现类是兄弟关系，增强类不能用实现类接收增强类对象，只能用接口接收）。

[举个例子](https://blog.csdn.net/QinYHJ/article/details/108376283)：例如类A中需要调用类B中的方法add()，为了方便扩展，使用一个接口Q声明方法add()，B是Q的实现类，此时可以在**类A中注入接口或者方法B**，来实现add()方法，但是当我们去对B进行扩展时，定义一个类C，C继承了类B，并且重写了add()方法，此时如果A中注入的是类B，则C中的新的add()无法实现。

对实现类B做增强得到的增强类C与实现类B是兄弟关系，所以不能用实现类B接收增强类C对象，只能用接口接收。



在resource下创建spring-service.xml

```xml
<!--1.扫描service下的包-->
<!--2.将我们的所有业务类，注入到Spring，可以通过配置，或者注解实现,自动装配@Autowired-->
<!--3.声明式事务配置,注入数据源-->
<!--4.aop事务支持!-->
```

注意：spring4.0之后不建议在字段使用autowired，可以在构造方法或setter上写。甚至有构造方法的时候可以省略@Autowired也可以自动实现注入。



### SpringMVC层

首先更改项目为web项目

在web/WEB-INF/web.xml进行配置

```xml
<!--配置DispatchServlet，注意，classpath配置为applicationContext.xml-->
<!--2.配置SpringMVC的乱码过滤-->
<!--Session-->
```

在resource下创建spring-mvc.xml

```xml
<!--1.注解驱动-->
<!--2.静态资源过滤-->
<!--3.扫描包: controller-->
<!--4.视图解析器-->
```

 在web/WEB-INF下创建jsp包



## AJAX

AJAX = Asynchronous JavaScript and XML (异步的JavaScript和XML)。

AJAX是一种在无需重新加载整个网页的情况下，能够更新部分网页的技术。

Ajax不是一种新的编程语言，而是一种用于创建更好更快以及交互性更强的Web应用程序的技术。

### jQuery AJAX

Ajax的核心是XMLHttpRequest对象(XHR)。XHR为向服务器发送请求和解析服务器响应提供了接口。能够以异步方式从服务器获取新数据。

jQuery提供多个与AJAX有关的方法。

通过jQuery AJAX方法，您能够使用HTTP Get和HTTP Post从远程服务器上请求文本、HTML、XML或JSON -- 同时您能够把这些外部数据直接载入网页的被选元素中。

### Ajax失去焦点应用

添加jquery包到web/statics/js。或者在线链接的jquery。

编写web.xml

springmvc-servlet.xml

```xml
<!-- 自动扫描指定的包，下面所有注解类交给IOC容器管理-->
<context:comiponent-scan base-package= "com.xxx.controller"/>
<!-- 让Spring MVC不处理静态资源  .css .js .html-->
<!--加了过滤还是不行的，clean一下项目就可以了-->
<mvc:default-servlet-handler />

<mvc:annotation-driven />
<!--视图解析器-->
```

编写index.jsp

![image-20210527170133279](E:\Markdown\JAVA\SpringMVC.assets\image-20210527170133279.png)

```jsp
<head>
<!--加载jquery包-->
<script src="${pageContext.request.contextPath}/statics/js/jquery-3.4.1.js"></script>
<script>
    // 失去焦点的请求函数
    function a(){
        // ajax操作步骤
        // $.post (url,param[可以省略],success),也可以
        $.post({
            // 请求地址
            // 寻找项目路径,pageContext.request.Contextpath当前项目
            url:"${pageContext.request.contextPath}/a1",
            // 向后台传入的数据
            // controller层的名字为name,#id
            data:{"name":$("#username").val()},
            // 返回请求成功结果
            success:function (data) {
            	alert(data);
            },
            // 返回请求失败结果
            error:function () {
            }
        })
    }
</script>

</head>
<body>
    <%-- 失去焦点的时候，发起一个请求(携带信息)到后台 --%>
    用户名: <input type="text" id="username" onblur="a()">
</body>
< /html>

```

controller/Ajaxcontroller.java

```java
@RestController
public class AjaxController {
    @RequestMapping("/a1" )
    public void a1(String name, HttpServletResponse response) throws IOException {
        System.out.println("a1 :param=>" + name) ;
        if ("xxx".equals(name)){
        response.getWriter().print("true");
        }else {
        response.getWriter().print("false");
        }
    }
}
```

 

### 异步请求

动态响应，局部刷新

```jsp
<head>
<script>
    // 保证让dom结构加载完后再执行代码
	$(function)() {
        // 绑定事件，ajax请求
        $("#btn").click(function(){
            // $.post (url,param[可以省略],success)
            $.post(
                // 配置请求
"${pageContext.request.contextPath}/a2",function (data) {
                    // 拼接,返回的数据
                    var html="";
                    for (let i =0; i < data.length; i++) {
                        html += "<tr>" + 
                            "<td>" + data[i].name + "</td>" + 
                            "<td>" + data[i].age + "</td>" + 
                            "<td>" + data[i].sex + "</td>" + 
                    }
                    // 返回结果,写在tbody里面
                    $("#content").html(html);
                })；
        })
    }
</script>
</head>
<body>
    < input type="button" value="加载数据" id="btn">
    <table>
        <tr>
            <td>姓名</td>
            <td>年龄</td>
            <td>性别</td>
        </tr>
        <tbody id="content">
            <%-- 数据:后台--%>
        </tbody>
    </table>
</body>
```

Controller层，加载Jquery依赖

```java
@RestController
public class AjaxController {
	@RequestMapping("/a2")
    public List<User> a2(){
        List<User> userList = new ArrayList<User>();
        //添加数据
        userList.add(new User( name:"Java", age:1, sex:"男"));
        userList.add(new User( name:"前端", age:1, sex:"女"));
        userList.add(new User( name:"运维", age:1, sex:"男"));
        
        return userList;
    }
}
```



## 拦截器interceptor

词根：inter中间+cept拿，握，引申在中间拦截

[过滤器与拦截器的区别](https://blog.csdn.net/testcs_dn/article/details/80279578)：拦截器是AOP思想的具体应用。

> 不同拦截器
>
> 2、数据独立性：SpringMVC方法间独立，独享request和response
>
> 而struts2虽然方法也是独立，但是所有的action变量是共享的
>
> 4、拦截机制：SpringMVC 用的是独立的aop方式，struts2 有自己的interceptor机制
>
> 所以struts2 的配置文件量要大于SpringMVC

拦截器只会拦截访问的控制器(controller)方法。

配置applicationContext.xml

```xml
<mvc:interceptor>
	<mvc:interceptor>
        <!--包括这个请求下面的所有的请求/-->
        <!--/admin/* 拦截的是/admin/add等等这种 , /admin/add/user不会被拦截-->
       <!--/admin/** 拦截的是/admin/下的所有-->
        <mvc:mapping path="/**"/>
        <bean class=" com.xxx.config.MyInterceptor"/>
    </mvc:interceptor>
</mvc:interceptor>
```

切面：config.MyInterceptor.java

```java
public class MyInterceptor implements HandlerInterceptor {

   //在请求处理的方法之前执行
   //判断什么情况下没有登录
   //如果返回true执行下一个拦截器
   //如果返回false就不执行下一个拦截器
   public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
       System.out.println("------------处理前------------");
       return true;
  }

   // 拦截日志
   //在请求处理方法执行之后执行
   public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
       System.out.println("------------处理后------------");
  }

   //在dispatcherServlet处理后执行,做清理工作.
   public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
       System.out.println("------------清理------------");
  }
}
```



## 总结























