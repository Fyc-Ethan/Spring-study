# Spring Security

## Day1

Spring Security为Spring框架提供安全支持

虽然Spring Security重量级并且配置烦琐，但在和Spring家族的产品一起使用时， 可以更加方便的使用Spring Security。因此Spring Security具有很大的优势。

Spring Security最核心的功能：

### 认证

Spring Security支持多种不同的认证方式，包括：表单认证。 OAuth2.0认证等等。

也可以进行自定义认证，通过引入第三方依赖来支持更多的认证方式。

- Spring Security中的认证工作主要是由AuthenticationManager接口来负责，在该接口中通过 authenticate方法来做认证。
- AuthenticationManager最主要的实现类是ProviderManager，ProviderManager管理了众多的 AuthenticationProvider实例。

### 授权

无论采用了哪种认证方式，都不影响在Spring Security中使用授权功能。

在Spring Security的授权体系中，有两个关键接口：
AccessDecisionManager 

AccessDecisionVoter是一个投票器，投票器会检查用户是否具备应有的角色，进而投出赞成、反对或者弃 权票；

AccessDecisionVoter

AccessDecisionManager则是一个决策器，来决定此次访问是否被允许。

### 其他

同时，Spring Security还提供了很多安全管理的“周边功能”，这也是一个非常重要的特色，例如：密码加密 ，RememberMe等等。

在Spring Security的架构设计中，认证（Authentication）和授权（Authorization）是分开的，无论使 用什么样的认证方式，都不会影响授权，这是两个独立的存在的。

### Web 安全

在Spring Security中，认证、授权等功能都是基于过滤器来完成的。

自定义过滤器：通过@Order注解去调整自定义过滤器在过滤器链中的位置。 

注意：Spring Security中的过滤器链通过FilterChainProxy嵌入到Web项目的原生过滤器链中。当存在多个过滤器链时，多个过 滤器链之间要指定优先级。

FilterChainProxy作为一个顶层管理者，将统一管理Security Filter。

并且FilterChainProxy本身将通过Spring框架提供的DelegatingFilterProxy整合到原生过滤器链中。 



## Day2

### Spring Security 基本认证

在Spring boot项目中使用Spring Security，需要引入web和Spring Security 依赖即可。项目里面可以直接自动的保护起来

### 组件

#### AuthenticationManager

AuthenticationManager是一个认证管理器，在认证成功，会返回一个Authentication对象。

AuthenticationManager也是一个接口，开发者也可以自定义AuthenticationManager的实现类。一般的实现类是ProviderManager。

#### AuthenticationProvider

AuthenticationProvider针对不同的身份类型执行具体的认证。

#### ProviderManager

可能会支持多种认证，可以有多种AuthenticationProvider，但只需要一个ProviderManager去代理

![image-20210402202458835](C:\Users\FYC\AppData\Roaming\Typora\typora-user-images\image-20210402202458835.png)

在ProviderManager中存在一个AuthenticationProvider列表， 在ProviderManager中遍历列表中的每一个AuthenticationProvider去执行身份认证，最终得到认证结果。

### AbstractAuthenticationProcessingFilter

过滤器，通过过滤器把校验流程嵌入到过滤器*里面

在spring security中,具体处理表单登录验证的是UsernamePasswordAuthenticationFilter,另外一个过滤器DefaultLoginPageGeneratingFilter用于在没有指定登录页时动态生成一个默认登录页




## Day3

### 密码加密

明文密码泄漏就会造成很大的安全隐患

#### 加密方式

• Hash算法： 直接密码加密存储

单向加密Hash算法：密文推不出原文

彩虹表(rainbow table)破解：破解Hash密文

• 密码加盐 ：添加随机数(盐)，即密码+随机数再加密，但是随机数也要存储。但是计算机算例增强也不适合加密

• 自适应单向函数：(Adaptable One-way function)

有意增加cpu和内存资源使用，降低性能，增加破解的难度，spring security中有很多这种函数。

#### 常见实现类

聚类PasswordEncoder的实现类

- BCryptPasswordEncoder(常用)：BCrypt算法，自己加盐，可以设置加密强度
- Argon2PasswordEncoder：Argon2算法，慢哈希函数，定制硬件上密码被破解问题
- Pbkdf2PasswordEncoder：
- SCryptPasswordEncoder

```java
BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
encoder.encode("123");//加密
```

#### DelegatingPasswordEncoder

代理类，spring security中默认使用

特点：

- 兼容性：旧密码加密方式易迁移
- 便捷性：方便修改加密方案
- 稳定性：加密方案方便自动升级

如果开发者使用DelegatingPasswordEncoder，则会进行检测，进行自动更新加密方式为BCryptPasswordEncoder。



## Day4

### 会话(Session)

当浏览器调用登录接口登录成功后，服务端会和浏览器之间建立一个会话。服务端可以通过会话确认用户身份。

在spring security中，与会话相关的功能由SessionManagementFilter和SessionAuthenticationStrategy接口来处理。

SessionManagementFilter过滤器委托给SessionAuthenticationStrategy接口进行处理，用法：防止会话攻击，配置会话并发数等等。

Session ID：确认用户身份，合法用户及权限

#### 会话并发管理

会话并发管理就是指在当前系统中，同一个用户可以同时创建多少个会话。，如果一台设备对应一个会话，那 么也可以简单理解为同一个用户可以同时在多少台设备上进行登录。

例如：电脑端QQ，使用spring security中的会话并发管理，可以确保只有一台设备进行登录。

##### 挤下线

当会话并发数达到限制时，新会话替代旧会话 ，旧会话失效。 

.maximumSessions(1)：只允许有个session会话存在。电脑端QQ

##### 限制登录

当会话并发数达到限制时，不能创建新会话，除非旧会话消失。 

.maxSessionsPreventsLogin(true)：当会话并发数达到限制时，禁止登录。

**实现原理**：服务端维护Session列表

#### 会话固定攻击

会话固定攻击(Session fixation attacks)是一种潜在的风险，恶意攻击者有可能通过访问当前应用程序来创建会话，然后诱导用户以相同的会话ID登录(点击链接，获取身份)。

##### 步骤

1. 攻击者自己可以正常访问网站，在访问的过程中，网站给攻击者分配了一个SessionID。
2. 攻击者利用自己拿到的SessionID构造一网站的链接，并把该链接发送给受害者。
3. 受害者使用该链接登录网站（该链接中含有SessionID，登录时网站检测到有SessionID，不重新分配SessionID，用现在的SessionID，因此网站认为当前SessionID是合法的SessionID。但是这个SessionID攻击者也有。
4. 攻击者冒充受害者

#### 会话固定攻击防御策略

1. spring security中默认自带Http防火墙，默认SessionID不放在地址栏。
2. 在Http响应的Set-Cookie字段中有httpOnly属性，默认开启httpOnly，意思是无法用js操作/读取Cookie。
3. 在用户登录成功后，改变SessionID，spring security中默认实现了该种方案

#### 修改SessionID的四种策略

- changSessionId()：默认方法
- none()：HttpSession不做变化
- migrateSession()：创建新的HttpSession对象，拷贝旧的HttpSession对象到新对象中。
- newSession()：创建新的HttpSession对象，但是migrateSessionAttributes设置为false。

## Day5

### Http防火墙

HttpFirewall，提前拦截非法请求，拒绝潜在危险请求，包装请求控制其行为。

在Servlet容器中，为HttpServletRequest接口定义了一些属性，如contextPath，servletPath，pathInfo，queryString等等，这些属性可以通过get方法获取。但是没有定义属性包含的值，各种容器处理方案不同，会出现混乱，造成隐患。

因此spring security中通过HttpFirewall进行加内存请求路径和参数是否合法。

#### 实现类

DefaultHttpFirewall：检测不是很严格的防火墙

StrictHttpFirewall：严格的HttpFirewall，默认的的Http防火墙

##### StrictHttpFirewall

可修改配置：

1. rejectForbiddenHttpMethod：校验请求方法是否合法。

   ```java
   void rejectForbiddenHttpMethod( HttpServletRequest request )
   ```

2. rejectedBlacklistedUrls：校验请求中的非法字符。

   > 即请求的Url是否规范：
   >
   > - Url编码之前或者之后包含分号;，%，/，\等，这个请求会被拒绝。
   > - Url编码之后英文句号**.**，这个请求会被拒绝。

3. rejectedUntrustedHosts：检验主机信息。

不可修改：(反射等修改，比较麻烦)

1. isNormalized：判断参数格式是否合法。不包含./，/../，/.三个字符就是合法。
2. containsOnlyPrintableAsciiCharacters：判断请求字符是否合法。即不可打印的ASCII码字符。

##### DefaultHttpFirewall

1. 将请求地址(path)中//换为/。

2. 将;分号提取处理，保存路径。

配置：

```java
@Bean
HttFirewall httpFirewall(){
    return new DefaultHttpFirewall();
}
```















