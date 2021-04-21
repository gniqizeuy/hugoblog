+++
title = "Spring Security"
description = "认识 Spring Security"
tags = [
    "security",
    "spring"

]
date = "2021-04-02"
categories = [
    "Spring",
    "java",
]
menu = "main"

+++

dfadsfa

<!--more-->

任何 Spring Web 应用本质上只是一个 servlet

Security Filter 在 HTTP 请求到达 controller 之前过滤每一个传入的 HTTP 请求

过滤链

### 认识 Spring Security

Spring Security 提供了声明式的安全访问控制解决方案（仅支持 Spring 的应用程序），对访问权限进行认证和授权，它基于 Spring Aop 和 Servlet 过滤器，提供了安全性方面的全面解决方案。

除常规的认证和授权外，它还提供了 ACls、LDAP、JAAS、CAS 等高级特性以满足复杂环境下的安全需求。

#### 1. 核心概念

- Principle：代表用户的对象 Principle（User），不仅指用户，还包括一切可以用于验证的设备。
- Authority：代表用户的角色 Authority（Role），每个用户都应该有一种角色，如管理员或是会员。
- Permission：代表授权，复杂的应用环境需要对角色的权限进行表述。

在 Spring Security 中，Authority 和 Permission 是两个完全独立的概念，两者并没有必然的联系，它们之间需要通过配置进行关联，可以是自己定义的各种关系。

#### 2. 认证和授权

安全主要分为验证（authentication）和授权（authorization）两个部分

##### 1）验证（authentication）

验证指的是，建立系统使用者信息（principal）的过程。使用者可以是一个用户、设备、和可以在应该程序种执行某些操作的其他系统。

用过户认证一般要求用户提供用户名和密码，系统通过校验用户名和密码的正确性来完成认证的通过或拒绝过程。

Spring Security 支持主流的认证方式，包括 HTTP 基本认证、HTTP 表单验证、HTTP 摘要验证、OPenID 和 LDAP 等。

Spring Security 进行验证的步骤如下：

1. 用户使用用户名和密码登录
2. 过滤器（UsernamePasswordAuthenticationFilter）获取到用户名、密码、然后封装成 Authentication。
3. AuthenticationManager 认证 token（Authentication的实现类传递）。
4. AuthenticationManager 认证成功，返回一个封装了用户权限信息的 Authentication 对象，用户的上下文信息（角色列表等）。
5. Authentication 对象赋值给当前的 SecurityContext，建立这个用户的安全上下文（通过调用 SecurityContextHolder.getContext().setAuthentication()）。
6. 用户进行一些收到访问控制机制保护的操作，访问控制机制会依据当前安全上下文信息检查这个操作所需的权限。

除利用提供的认证外，还可以编写自己的 Filter ，提供于那些不是基于 Spring Security 的验证系统的操作。

##### 2）授权（authorization）

在一个系统中，不同用户具有的权限是不同的。一般来说，系统会为不同的用户分配不同的角色，而每个角色则对应一系列的权限。

它判断某个 Principal 在应用程序中是否允许执行某个操作。在进行授权判断之前，要求其所要使用到的规则必须在验证过程中已经建立好了。

对 Web 资源的保护，最好的办法是使用过滤器。对方法调用的保护，最好的办法是使用 AOP。

Spring Security 在进行用户认证及授权时，也是通过各种拦截器和 AOP 来控制权限访问的，从而实现安全。

#### 3. 模块

- 核心模块 — spring-security-core.jar：包含核心验证和访问控制类和接口，以及支持远程配置的基本 API
- 远程调用 — spring-security-remote.jar：提供与 Spring Remote 集成
- 网页 — spring-security-web.jar：包括网站安全的模块，提供网站认证服务和基于 URL 访问控制
- 配置 — spring-security-config.jar：包含安全命令空间解析代码
- LDAP — spring-security-ldap.jar：LDAP 验证和配置
- ACL — spring-security-acl.jar：对 ACL 访问控制表的实现
- CAS — spring-security-cas.jar：对 CAS 客户端的安全实现
- OpenID — spring-security-openid.jar：对 OpenID 网页验证的支持
- Test — spring-security-test.jar：对 spring security 的测试的支持

### 核心类

#### 1. SecurityContext

SecurityContext 中包含当前正在访问系统的用户的详细信息，它只有以下两者方法：

- `getAuthentication()`：获取当前经过身份验证的主体或身份验证的请求令牌
- `setAuthentication()`：更改或删除当前已验证的主体身份验证信息

#### 2. SecurityContextHolder

SecurityContextHolder 用来保存 SecurityContext。最常用的是 `getContext()` 方法，用来获得当前 SecurityContext

SecurityContextHolder 中定义了一系列的静态方法，而这些静态方法的内部逻辑是通过 SecurityContextHolder 持有的 SecurityContextHolderStrategy 来实现的，如 `clearContext()`、`getContext()`、`setContext()`、`createEmptyContext()`。SecurityContextHolderStrategy 接口的关键代码如下：

~~~java
public interface SecurityContextHolderStrategy {
    void clearContext();
    SecurityContext getContext();
    void setContext(SecurityContext context);
    SecurityContext createEmptyContext();
}
~~~

##### 1)  strategy 实现

默认使用的 strategy 就是基于 ThreadLocal 的 ThreadLocalSecurityContextHolderStrategy 来实现的

除了上述提到的，Spring Security 还提供了 3 种类型的 strategy 来实现

- `ClobalSecurityContextHolderStrategy`：表示全局使用一个 SecurityContext，如 C/S 结构的客户端
- `InheritablThreadLocalSecurityContextHolderStrategy `：使用 InheritablThreadLocal 来存放 SecurityContext，即子线程可以使用父线程中存放的变量
- `ThreadLocalSecurityContextHolderStrategy`：使用 ThreadLocal 来存放 SecurityContext

一般情况下，使用默认的 Strategy 即可。但是，如果改变默认的 strategy，Spring Security 提供了两者方法来改变 "StrategyName"

SecurityContextHolder 类中有 3 种不同类型的 strategy，分别为 MODE_THREADLOCAL、MODE_INHERITABLETHREADLOCAL 和 MODE_GLOBAL，关键代码如下：

~~~java
public static final String MODE_THREADLOCAL="MODE_THREADLOCAL";
public static final String MODE_INHERITABLETHREADLOCAL = "MODE_INHERITABLETHREADLOCAL";
public static final String MODE_GLOBAL = "MODE_GLOBAL";
public static final String SYSTEM_PROPERTY = "spring.security.strategy";
private static String strategyName = System.getProperty(SYSTEM_PROPERTY);
private static SecurtyContextStrategy strategy;
~~~

MODE_THREADLOCAL 是默认的方法

如果改变 strategy ，则有以下两种方法：

- 通过 SecurityContextHolder 的静态方法 `setStrategyName(java.lanng.String strategyName)` 来改变需要使用的 strategy
- 通过系统属性（SYSTEM_PROPERTY）进行指定，其中属性名默认为 ”spring.security.strategy“，属性值为对应 strategy 的名称

##### 2）获取当前用户的 SecurityContext

Spring Security 使用一个 Authentication 对象来描述当前用户的相关信息。SecurityContextHolder 中持有的是当前用户的 SecurityContext ，而 SecurityContext 持有的是代表当前用户相关信息的 Authentication 的引用。

这个 Authentication 对象不需要自己创建，Spring Security 会自动创建相应的 Authentication 对象，然后赋值给当前的 SecurityContext 。但是，往往需要在程序中获取用户的相关信息，比如，最常见的是获取当前登录用户的用户名。在程序的任何地方，可以通过如下方式获取到当前用户的用户名。

~~~java
public String getCurrentUsername () {
    Object principal = SecurityContextHolder.getContext().getAuthentication().getPrincipal();
    
    if (principal instanceof UserDetails)
        return ((UserDetails) principal).getUserName();
    
    if (principal instanceof Principal)
        return ((Principal) principal).getName();
    
    return String.valueOf(principal);
}
~~~

`getAuthentication()` 会返回认证信息

`getPrincipal()` 方法返回身份信息，它是 UserDetails 对身份信息的封装

获取当前用户的用户名，最简单的方式如下：

~~~java
public String getCurrentUsername () {
    return SecurityContextHolder.getContext().getAuthentication().getName();
}
~~~

在调用 SecurityContextHolder.getContext() 获取 SecurityContext 时，如果对应的 SecurityContext 不存在，则会返回空的 SecurityContext

##### 3） ProviderManager

ProviderManager 会维护一个认证的列表，以便处理不同认证方式的认证，因此系统可能会存在多种认证方式，比如手机号、用户名密码、邮箱方式

在认证时，如果 ProviderManager 的认证结果不是 null ，则说明认成功，不再进行其他方式的认证，并且作为认证的结果保存在 SecurityContext 中，如果不存在，则输出错误信息 <i>ProviderNotFoundException</i>

##### 4） DaoAuthenticationProvider

它是 AuthenticationProvider 最常见的实现，用来获取用户提交的用户名和密码，并进行正确性比对。如果正确，则返回一个数据库中的用户信息

在用户在前台提交了用户名和密码后，就会封装成 UsernamePasswordAuthenticationToken。然后，DaoAuthenticationProvider 根据 retireveUser 方法，交给 additionalAuthenticationChecks 方法完成 UsernamePasswordAuthenticationToken 和 UserDetails 密码的比对。如果这个方法没有抛出异常，则认为对比成功。

比对密码需要用到 PasswordEncoder 和 SaltSource

##### 5） UserDetails

UserDetails 是 Spring Security 的用户实体类，包含用户名、密码、权限等信息。Spring Security 默认实现了内置的 User 类，供 Spring Security 安全认证使用。当然，也可以自己实现。

UserDetails 和 Authentication 接口很类似，都拥有 username 和 authorities 。一定要区分清楚 Authentication 的 getCredentials() 与 UserDetails 中的 getPassword() 。前者是用户提交的密码凭证，不一定是正确的，或数据库不一定存在；后者是用户正确的密码，认证器要进行比对的就是两者是否相同。

Authentication 的 getAuthorities() 方法是由 UserDetails 的 getAuthorities() 传递而形成的。UserDetails 的用户信息是经过 AuthenticationProvider 认证之后被填充的。

UserDetails 中提供了以下几种方法：

- `Stirng getPassword()`：返回验证用户密码，无法返回则显示为 null。
- `Stirng getUsername()`：返回验证用户名，无法返回则显示为 null。
- `boolean isAccountNonExpired()`：账户是否过期，过期无法验证。
- `boolean isAccountNonLocked()`：指定用户是否被锁定或解锁，锁定的用户无法进行身份验证。
- `boolean isCredentialsNonExpired()`：指定是否已过期的用户的凭证（密码），过期的凭证无法认证。
- `boolean isEabled()`：是否被禁用，禁用的用户不能进行身份验证。

##### 6） UserDetailsService

用户信息是通过 UserDetailsService 接口来加载的。该接口的唯一方法是 `loadUserByUsername(String username)` ，用来根据用户名加载相关信息，这个方法的返回值是 UserDetails 接口，其中包含了用户的信息，包括用户名、密码、权限、是否启用、是否被锁定、是否过期等。

##### 7） GrantedAuthority

GrantedAuthority 只定义了一个 `getAuthority()` 方法。该方法返回一个字符串，表示对应权限的字符串。如果对应权限不能用字符串表示，则返回 null。

GrantedAuthority 接口通过 UserDetailsService 进行加载，然后赋予 UserDetails

Authentication 的 getAuthorities() 方法可以返回当前 Authentication 对象拥有的权限，其返回值是一个 GrantedAuthority 类型的数组。每一个 GrantedAuthority 对象代表赋予当前用户的一种权限。

##### 8） Filter

**1 .  SecurityContextPersistenceFilter**

它从 SecurityContextResponsity 中取出用户认证信息。为了提高效率，避免每次请求都要查询认证信息，他会从 Session 中取出已认证的用户信息，然后将其放入 SecurityContextHolder 中，以便其他 Filter 使用

**2.  WebAsyncManagerIntergrationFilter**

集成了 SecurityContext 和 WebAsyncManager ，把 SecurityContext 设置到异步编程，使其也能获取到用户的上下文认证信息

**3. HanderWriterFilter**

它对请求的 Header 添加相应的信息

**4. CsrfFilter**

跨域请求伪造过滤器。通过客户端传过来的 token 与服务器端存储的 token 进行比较，来判断请求的合理性

**5. LogoutFilter**

匹配登出 URL 。匹配成功后，退出用户，并清除认证信息

**6. UsernamePasswordAuthenticationFilter**

登录认证过滤器，默认是对 "/login" 的 POST 请求进行认证。该方法会调用 attemptAuthentication ，尝试获取一个 Authentication 认证对象，以保存认证信息，然后转下下一个 Filter ，最后调用 successfulAuthentication 执行认证后的事件。

**7.  AnonymousAuthenticationFilter**

如果 SecurityContextHolder 中的认证信息为空，则会创建一个匿名用户到 SecurityContextHolder 中。

**8. SessionManagementFilter**

持久化登录用户的用户信息。用户信息会被保存到 Session、Cookie、或 Redis 中

### 配置 Spring Security

#### 1. 继承 WebSecurityConfigurerAdapter

通过重写抽象接口 WebSecurityConfigurerAdapter ，再加上注解 `@EnableWebSecurity` ，可以实现 Web 的安全配置

WebSecurityConfigurerAdapter Config 模块一共有 3 个 builder（构造程序）

- `AuthenticationManagerBuilder`：认证相关 builder ，用来配置全局的认证相关的信息。它包含 AuthenticationProvider 和 UserDetailsService ，前者是认证服务器提供者，后者是用户详情查询服务
- `HttpSecurity`：进行权限控制规则相关配置
- `WebSecurity`：进行全局请求忽略规则配置、HTTPFirewall 配置、debug 配置、全局 SecurityFilterChain 配置

配置安全，通常要重写以下方法：

~~~java
// 通过 auth 对象的方法添加身份验证
protected void configure(AuthenticationManagerBuilder auth) throws Exception {}

// 通常用于设置忽略权限的静态资源
public void configure(WebSecurity web) throws Exception {}

// 通过 HTTP 对象的 authorizeRequests() 方法定义 URL 访问权限。默认为 formLogin() 提供一个简单的登录验证页面
protected void configure(HttpSecurity httpSecurity) throws Exception {}
~~~

#### 2. 配置自定义策略

配置安全需要继承 WebSecurityConfigurerAdapter ，然后重写其方法

~~~java
//指定为配置类
@Configuration
//指定为Spring Security配置类，如果是 WebFlux，则需要启用 @EnableWebFluxSecurity
@EnableWebSecurity
// 如果要启用方法安全设置，则开启此项
@EnableGloabelMethodSecurity(prePostEnabled=true)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    public void configure(WebSecurity web) throws Exception {
		// 不拦截静态资源
        web.ignoring().antMatchers("/static/**");
    }
    
    @Bean
    public PasswordEncoder PasswordEncoder() {
		// 使用 BCrypt 加密
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()
            .usernameParameter("uname")
            .passwordParameter("pwd")
            .loginPage("/admin/login")
            .peimitAll()
            .and()
            .authorizeRequests()
            .antMatchers("/admin/**").hasRole("ADMIN")
            // 除上面外的所有请求全部需要鉴权认证
            .anyRequest().authemticated();
        
        http.logout().permitAll();
        http.rememberMe().rememberMeParameter("rememberme");
        // 处理异常，拒绝访问就重定向 403 页面
        http.exceptionHandling().accessDeniedPage("/403");
        http.logout().logoutSuccessUrl("/");
        http.csrf().ignoringAntMatchers("/admin/upload");
}

~~~

代码解释如下：

- `authorizeRequest()`：定义哪些 URL 需要被保护，哪些不需要被保护
- `antMatchers("/admin/**").hasRole("ADMIN")`：定义 /admin/ 下的所有 URL 。只有拥有 admin 角色的用户才有访问权限
- `formLogin()`：自定义用户登录验证的页面
- `http.csrf()`：配置是否开启 CSRF 保护，还可以再开启之后指定忽略的接口

如果开启了 CSRF ，则一定在验证页面加入下面代码以传递 token 值。

~~~html
<head>
    <meta name="_csrf" th:content="${_csrf.token}"/>
    <!-- default header name is X-CSRF-TOKEN -->
    <mate name="_csrf_header" th:content="${_csrf.headerName}"/>
</head>
~~~

如果要提交表单，则需要在表单中添加以下代码以提交 token 值

~~~html
<input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}">
~~~

http.reemberMe()：记住我功能，可以指定参数。

使用时，添加如下代码

~~~html
<input class="i-checks" type="checkbox" name="rememberme"> 记住我
~~~

#### 配置加密方式

默认的加密方式是 BCrypt 。只要在安全配置类配置即可使用

~~~java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();	// 使用 BCrypt 加密
}
~~~

在业务代码中，可以用以下方式对密码进行加密：

~~~java
BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
String encodePassword = encoder.encode(password);
~~~

#### 自定义加密规则

除默认的加密规则，也可以自定义加密规则

~~~java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.userDetailsService(UserService()).passwordEncoder(new PasswordEncoder() {
        @Override
        public String encode(CharSequence charSequence) {
            return MD5Util.encode((String)charSequence);
        }
        
        @Override
        public boolean matches(CharSequence charSequence, String s) {
            return s.equals(MD5Util.encode((String)charSequence));
        }
    });
}
~~~

#### 配置多用户系统

一个完整的系统一般包含多种用户系统，比如后台管理系统+前端用户系统。Spring Security 默认只提供一个用户系统，所以，需要通过配置实现多用户系统

比如，要创建一个前台会员系统，则可以通过以下步骤实现

##### 构建 UserDetailsService 用户信息服务接口

构建前端用户 UserDetailsService 类，并继承 UserDetailsService 。

~~~java
public class UserSecurityService implements UserDetailsService {
    @Autowired
    private UserRepository userRepository;
    
    @Override
    public UserDetails loadUserByUsername(String name) throws UsernameNotFoundException {
        User user = userRepository.findByName(name);
        
        if (user == null) {
            User mobileUser = userRepository.findByMobile(name);
            
            if (mobileUser == null) {
            	User emailUser = userRepository.findByEmail(name);
                
                if (emailUser == null) {
            		throw new UsernameNotFoundException("用户名，邮箱，或者手机号不存在")
                } else {
                    user = userRepository.findByEmail(name);
                }
            } else {
                user = userRepository.findByMobile(name);
            }
        } else if ("locked".equals(user.getStatus())) {
            // 被锁定，无法登录
            throw newLockedException("用户被锁定");
        }
        
        return user;
    }
}
~~~

 ##### 进行安全配置

在继承 WebSecurityConfigurerAdapter 的 Spring Security 的配置类中，配置 UserSecurtyService 类。

~~~java
@Bean
UserDetailsService UserService() {
    return new UserSecurityService();
}
~~~

如果加入后台管理系统，只需要重复上面步骤即可

#### 获取当前登录用户信息的几种方式

##### 在 Thymeleaf 视图中获取

要在 Thymeleaf 视图中获取用户信息，可以使用 Spring Securty 的标签特性。

在 Thymeleaf 页面中引入 Thymeleaf 的 Spring Securty 依赖

~~~html
<!DOCTYPE html>
<html lang="zh" xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity5">
    <!- 省略 ->
    
    <body>
    	<!- 匿名 ->
        <div sec:authorize="isAnonymous()">
    		未登录，单击<a th:href="@{/home/login}">登录</a>
        </div>
        <!- 已登录 ->
        <div sec:authorize="isAuthenticated()">
            <p>已登录</p>
            <p>登录名：<span sec:authentication="name"></span></p>
            <p>角色：<span sec:authentication="principal.authorities"></span></p>
            <p>id：<span sec:authentication="principal.id"></span></p>
            <p>Username：<span sec:authentication="principal.username"></span></p>
        </div>
    </body>
</html>
~~~

这里要注意版本的对应。如果引入 thymeleaf-extras-springsecurity 依赖依然获取不到信息，那么可能是 Thymeleaf 版本和 thymeleaf-extras-springsecurity 的版本不对，请检查在 pom.xml 文件的两个依赖

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity5</artifactId>
</dependency>
~~~

##### 在 Controller 中获取

在控制器中获取用户信息有 3 种方式，见代码注释。

~~~java
@GetMapping("userinfo")
public String getProduct(Principal principal, Authentication authentication, HttpServletRequest httpServletRequest) {
    /**
    *@Decription:1，通过 Principal 参数获取
    */
    String username = principal.getName();
    
    /**
    *@Decription:2，通过 Authentication 参数获取
    */
    String username2 = authentication.getName();
    
    /**
    *@Decription:3，通过 HttpServletRequest 参数获取
    */
    Principal httpServletRequestUserPrincipal = httpServletRequest.getUserPrincipal();
    String username3 = httpServletRequestUserPrincipal.getName();
    
    return username;
}
~~~

##### 在 Bean 种获取

~~~java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
if(!authentication instanceof AnonymousAuthenticationToken) {
    String username = authentication.getName();
    return username;
}
~~~

在其他 Authentication 类也可以这样获取，比如在 UsernamePasswordAuthenticationToken 类中。

使用以上代码获取不到，可能是以下问题：

- 要使上面的获取生效，必须在继承 WebSecurityConfigurerAdapter 的类中的 `http.antMacther("/*")` 的鉴权 URL 范围内。
- 没有添加 Thymeleaf 的 thymeleaf-extras-springsecurity 依赖
- 添加了 Spring Security 的依赖，但是版本不对，比如 Spring Security 和 Thymeleaf 的版本不对

#### 实例，用 Spring Security 实现后台登录及权限认证功能

##### 引入依赖

~~~xml
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
		</dependency>
		<dependency>
			<groupId>org.thymeleaf.extras</groupId>
			<artifactId>thymeleaf-extras-springsecurity5</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-test</artifactId>
			<scope>test</scope>
		</dependency>
~~~

##### 创建权限开放的页面

这个页面不需要鉴权即可访问，以区别演示需要鉴权的页面

~~~html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity5">
    <head>
        <title>Spring Security 案例</title>
    </head>
    <body>
        <h1>Welcome!</h1>
        <p><a th:href="@{/home}">会员中心</a></p>
    </body>
</html>
~~~

##### 创建需要权限验证的页面

其实可以和不需要鉴权的页面一样，鉴权可以不在 HTML 页面中进行

~~~html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity5">
	<head><title>home</title></head>
    <body>
        <p>会员中心</p>
        <p th:inline="text">Hello <span sec:authentication="name"></span></p>
        <form th:action="@{/logout}" method="post">
            <input type="submit" value="登出"/>
        </form>
	</body>
</html>
~~~

使用 Spring Securty5 后，可以在模板中用 `<span sec:authentication="name"></span>` 或 `[[${#httpServletRequest.remoteUser}]]` 来获取用户名，登出请求将被发送到 "/logout" 。成功注销后，会将用户重定向到 "/login?logout" 。

##### 配置 Spring Security

1）配置 Spring MVC

可以继承 WebMvcConfigurer

~~~java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        //设置登录处理操作
        registry.addViewController("/home").setViewName("springsecurity/home");
        registry.addViewController("/").setViewName("springsecurity/welcome");
        registry.addViewController("/login").setViewName("springsecurity/login");
    }
}

~~~

2）配置 Spring Security

Spring Security 的安全配置需要继承 WebSecurityConfigurerAdapter，然后重写其方法

~~~java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

@Configuration
@EnableWebSecurity//指定为Spring Security配置类
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/", "/welcome", "/login").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin().loginPage("/login").defaultSuccessUrl("/home")
                .and()
                .logout().permitAll();
    }
    
    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder()) // 指定编码方式
            .withUser("admin").password("e10adc3949ba59abbe56e057f20f883e")
            .roles("USER");
    }
}
~~~

代码解释如下

- `@EnableWebSecurity`：集成了 Spring Security 的 Web 安全支持
- `WebSecurityConfig`：在配置类的同时集成了 WebSecurityConfigurerAdapter ，重写了其中特定方法，用于自定义 Spring Security 配置，Spring Security 的工作量都集中在该配置中
- `configure(HttpSecurity)`：定义了哪些 URL 路径应该被拦截
- `configureClobal(AuthenticationManagerBuilder)`：在内存中配置一个用户，admin/123456，这个用户拥有 User 角色

##### 创建登录页面

登录页面要特别注意是否开启了 CSRF 功能。如果开启了，则需要提供 token 信息。

~~~html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity5">
	<head><title>Spring Security Example </title></head>
    <body>
        <div th:if="${param.error}">
            无效的用户名或者密码
        </div>
        <div th:if="${param.logout}">
            你已经登出
        </div>
        <form th:action="@{/login}" method="post">
            <div><label> 用户名 : <input type="text" name="username"/> </label></div>
            <div><label> 密码: <input type="password" name="password"/> </label></div>
            <div><input type="submit" value="登录"/></div>
        </form>
    </body>
</html>
~~~

#### 权限控制方式

##### Spring EL 表达式

Spring Security 支持在定义 URL 访问或方法访问权限时使用 Spring EL 表达式。根据表达式返回的值（true 或 false）来授权或拒绝对应的权限。Spring Security 可用表达式对象的基类是 SecurityExpressionRoot，它提供了通用的内置表达式，见下表

| 表达式                          | 描述                                                         |
| :------------------------------ | :----------------------------------------------------------- |
| hasRole([role])                 | 当前用户是否拥有指定角色                                     |
| hasAnyRole([role1, role2])      | 多个角色以逗号进行分隔的字符串，如果当前用户拥有指定角色中的任意一个，则返回 true |
| hasAuthority([auth])            | 等同于 hasRole                                               |
| hasAnyAuthority([auth1, auth2]) | 等同于 hasAnyRole                                            |
| Principle                       | 代表当前用户的 principle 对象                                |
| authentication                  | 直接从 SecurityContext 获取当前 Authentication 对象          |
| permitAll                       | 总是返回 true，表示允许所有的                                |
| denyAll                         | 总是返回 false，表示拒绝所有的                               |
| isAnonymous()                   | 当前用户是否是一个匿名用户                                   |
| isRememberMe()                  | 表示当前用户是否是通过 Remember-Me 自动登录的                |
| isAuthenticated()               | 表示当前用户是否已经登录认证成功了                           |
| isFullAuthenticated()           | 如果当前用户既不是匿名用户，又不是通过 Remember-Me 自动登录的，则返回 true |

在视图模板文件中，可以通过表达式控制显示权限

~~~html
<p sec:authorize="hasRole('ROLE_ADMIN')">管理员</p>
<p sec:authorize="hasRole('ROLE_USER')">普通用户</p>
~~~

在 WebSecurityConfig 中添加两个内存用户用于测试，角色分别是 ADMIN、USER

~~~java
	.withUser("admin").password("123456").roles("ADMIN")
    .and().withUser("user").password("123456").roles("USER");
~~~

用户 admin 登录，则显示 管理员

用户 user 登录，则显示 普通用户

然后，在 WebSecurityConfig 中加入如下的 URL 权限配置

~~~java
	.andMatchers("/home").hasRole("ADMIN")
~~~

这时，当用户访问 "home" 页面时能正常访问，而用 "user" 用户访问时则会提示 "403 禁止访问" 。因为，这段代码配置使这个页面访问必须具备ADMIN（管理员）角色，这是通过 URL 控制权限的方法

##### 通过表达式控制 URL 权限

如果要限定某类用户访问某个 URL ，则可以通过 Spring Security 提供的基于 URL 的权限控制来实现

Spring Security 提供的保护 URL 的方法是重写 `configure(HttpSecurity http)` 的方法，HttpSecurity 提供的方法见下表

| 方法名                                  | 用途                                            |
| --------------------------------------- | ----------------------------------------------- |
| access(String attribute)                | SpringEL 表达式结果为 true 时可访问             |
| anonymous()                             | 匿名可访问                                      |
| denyAll()                               | 用户不可访问                                    |
| fullAuthenticated()                     | 用户完全认证通过（非 remember me 下的自动登录） |
| hasAnyAuthority(String ...)             | 参数中任意权限可访问                            |
| hasAnyRole(String ...)                  | 参数中任意角色可访问                            |
| hasAuthority(String)                    | 某一权限的用户可访问                            |
| hasRole(String role)                    | 某一角色的用户可访问                            |
| permitAll()                             | 所有用户可访问                                  |
| rememberMe()                            | 允许通过 remember me 登录的用户访问             |
| authenticated()                         | 用户登录后可访问                                |
| hasIpAddress(String ipaddressException) | 用户来自参数中的 IP 可访问                      |

还需要额外注意几下几点

- `authenticated()`：保护 URL ，需要用户登录。如：`anyRequest().authenticated()` 代表其他未配置的页面都已经授权
- `permitAll()`：指定某些 URL 不进行保护。一般针对静态资源文件和注册等未授权情况下需要访问的页面
- `hasRole(String role)`：限制单个角色访问。在 Spring Security 中，角色是被默认增加 "ROLE_" 前缀的，所以角色 ADMIN 代表 ROLE_ADMIN
- `hasAnyRole(String... roles)`：允许多个角色访问。这和 Spring Boot1.x 版本有所不同
- `access(String attribute)`：该方法可以创建复杂的限制，比如可以增加 RBAC 的权限表达式
- `hasIpAddress(String ipaddressException)`：用于限制 IP 地址或子网

具体用法见下代码

~~~java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        .antMatchers("/static", "/register").permitAll()
        .antMatchers("/user/**").hasAnyRole("USER", "ADMIN")
        // 代表 "/admin" 下的所有 URL 只允许 IP 为 "100.100.100.100" 且用户角色是 "ADMIN" 的用户访问
        .antMatchers("/admin/**").access("hasRole('ADMIN') and hasIpAddress('100.100.100.100')")
        // 其他未配置的页面都已经授权
        .anyRequest().authenticated()
}
~~~

##### 通过表达式控制方法权限

要想在方法上使用权限控制，则需要使用启用方法安全设置的注解 `@EnableGlobalMethodSecurity()` 。它是默认是禁用的，需要在继承 `WebSecurityConfigureAdapter` 的类上加注解来启用，还需要配置启用的类型，它支持开启如下三种类型

-  `@EnableGlobalMethodSecurity(jsr250Enabled = true)`：开启 JSR-250
-  `@EnableGlobalMethodSecurity(prePostEnabled = true)`：开启 prePostEnabled
-  `@EnableGlobalMethodSecurity(securedEnabled = true)`：开启 secured

##### JSR-250

JSR 是 Java Specification Requests 的缩写，是 Java 规提案，任何人都可以提交 JSR ，以向 Java 平台增添新的 API 和服务，JSR 是 Java 的一个重要标准

Java 提供了很多 JSR ，比如 JSR-250、JSR-303、JSR-308。只需要记住不同的 JSR 功能定义不一样。JSR-303 主要是为数据的验证提供了一些规范的 API。这里 JSR-250 是用于提供方法安全设置的，它注意提供了注解 `@RolesAllowed` 。

它提供的方法主要有如下几种

- `@DenyAll`：拒绝所有访问
- `@RolesAllowed({"USER", "ADMIN"})`：该方法只要有 USER、ADMIN 任一权限即可访问
- `@PermitAll`：允许所有访问

##### prePostEnabled

`@preAuthorize`

它在方法执行之前执行，使用方法如下

1）限制 userId 的值是否等于 principal 中保存的当前用户的 userId ，或当前用户是否具有 ROLE_ADMIN 的权限

~~~java
@preAuthorize("#userId == authentication.principal.userId or hasAuthority("ADMIN")")
~~~

2）限制用于 ADMIN 角色才能执行

~~~java
@preAuthorize("hasRole('ROLE_ADMIN')")
~~~

3）限制用于 ADMIN 角色或 USER 角色才能执行

~~~java
@preAuthorize("hasRole('ROLE_USER')" or "hasRole('ROLE_ADMIN')")
~~~

4）限制只能查询 id 小于 3 的用户才能执行

~~~java
@preAuthorize("#id<3")
~~~

5）限制只能查询自己的信息，这里一定要在当前页面经过权限验证，否则会报错

~~~java
@preAuthorize("principal.username.equals(#username)")
~~~

6) 限制用户名只能为 long 的用户

~~~java
@preAuthorize("#user.name.equals('long')")
~~~

对于低版本的 Spring Security ，添加注解后还需要将 AuthenticationMananger 定义为 Bean

~~~java
@Bean
@Override
public AuthenticationMananger authenticationManangerBean() throws Exception {
    return super.authenticationManangerBean();
}

@Autowired
AuthenticationMananger authenticationMananger;
~~~

`@PostAuthorize`

表示在方法执行之后执行，有时需要在方法调用完后才进行权限检查。可以通过注解 @PostAuthorize 达到这一效果

注解 @PostAuthorize 是方法调用完成后进行权限检查的，它不能控制方法是否被调用，只能在方法调用完后检查权限，来决定是否抛出 <i>AccessDeniedException</i>

这里也可以调用方法的返回值。如果 EL 为 false ，那么该方法已经执行完了，可能会回滚。EL 变量 returnObject 表示返回的对象，如：

~~~java
@PostAuthorize("returnObject.userId == authentication.principal.userId or hasPermission(returnObject, 'ADMIN')");
~~~

`@PreFilter`

表示在方法执行之前执行。它可以调用方法的参数，然后对参数值进行过滤、处理或修改。EL 变量 filterObject 表示参数。如有多个参数，则使用 filterTarget 注释参数。方法参数必须是集合或数组

`@postFilter`

表示方法执行之后执行。而且可以调用方法的返回值，然后对返回值进行过滤、处理、或修改，并返回。EL 变量 returnObject 表示返回的对象，方法需要返回集合或数组

如使用 @PreFilter 和 @PostFilter 时，Spring Security 将移除使对应表达式的结果为 false 的元素。

当 Filter 标注的方法拥有多个几个类型的参数时，需要通过 filterTarget 属性指定当前是针对哪个参数进行过滤的

##### securedEnabled

开启 securedEnabled 支持后，可以使用注解 `@Secured` 来认证用户是否有权限访问

~~~java
@Secured("IS_AUTHENTICATED_ANONYMOUSLY")
public Account readAccount(Long id);
@Secured(ROLE_TELLER)
~~~

##### 使用 JSR-280注解

1）开启支持

在安全配置类中，启用注解 `EnableGlobalMethodSecurity(jsr250Enabled=true)`

2）创建 user 服务接口 UserService

~~~java
public interface UserService {
    public String addUser();
    public String updateUser() ;
    public String deleteUser() ;
}
~~~

3）实现 user 服务接口

~~~java
@Service
public class UserServiceImpl implements UserService {
    @Override
    public String addUser() {
        System.out.println("addUser");
        return null;
    }

    @Override
    @RolesAllowed({"ROLE_USER","ROLE_ADMIN"})
    public String updateUser() {
        System.out.println("updateUser");
        return null;
    }

    @Override
    @RolesAllowed("ROLE_ADMIN")
    public String deleteUser() {
        System.out.println("delete");
        return null;
    }
}
~~~

4）编写控制器

~~~java
@RestController
@RequestMapping("user")
public class UserController {
    @Autowired
    private UserService userService;

    @GetMapping("/addUser")
    public void addUser() {
        userService.addUser();
    }

    @GetMapping("/updateUser")
    public void updateUser() {
        userService.updateUser();
    }

    @GetMapping("/delete")
    public void delete() {
        userService.deleteUser();
    }
}
~~~

5）测试

启动项目，访问 "/user/addUser"，则控制台输出

~~~text
addUser
~~~

访问 "/user/delete" 和 "/user/updateUser"，则会提示没有权限：

~~~text
There was an unexpectd error(type=Forbidden, status=403)
Access Denied
~~~

#### 实现 RBAC 权限模型

RBAC 模型简化了用户和权限的关系。通过角色对用户进行分组，分组后可以很方便地进行权限分配和管理。RBAC 模型易扩展和维护，下面是具体步骤

1）创建 RBAC 验证服务接口（用于权限检查）

~~~java
public interface RbacService {
    boolean hasPermission(HttpServletRequest request, Authentication authentication);
}
~~~

2）编写 RBAC 服务实现，判断 URL 是否在权限表中

实现 RBAC 服务，步骤如下：

1 通过诸如用户和该用户权限（权限在登录成功时已经缓存起来，当需要访问权限时，直接从缓存取出）验证该请求是否有权限，有就返回 true ，没有就返回 false，不允许访问该 URL

2 传入 request ，可以使用 request 获取该次请求的类型

3 根据 Restful 风格使用它来控制的权限。如请求是 POST ，则证明该请求时向服务器发送一个新建资源请求，可以使用 `request.getMethod()` 来获取该请求的方式

4 配合角色所允许的权限路径进行判断和授权操作

5 如果获取到的 Principal 对象不为空，则代表授权已经通过

本实例不针对 HTTP 请求进行判断，只根据 URL 进行鉴权

~~~java
@Component("rbacService")
public class RbacServiceImpl implements RbacService {
    private org.springframework.util.AntPathMatcher AntPathMatcher = new AntPathMatcher();
    @Autowired
    private SysPermissionRepository permissionRepository;
    @Autowired
    private SysUserRepository sysUserRepository;
     @Override
    public boolean hasPermission(HttpServletRequest request, Authentication authentication) {
        Object principal = authentication.getPrincipal();
        boolean hasPermission = false;

        /**
         *这里应该注入用户和该用户所拥有的权限（权限在登录成功的时候已经缓存起来，当需要访问该用户的权限是，直接从缓存取出！）
         *然后验证该请求是否有权限，有就返回true，否则则返回false不允许访问该Url。
         *还传入了request,可以使用request获取该次请求的类型。
         *根据restful风格使用它来控制我们的权限
         *例如当这个请求是post请求，证明该请求是向服务器发送一个新建资源请求，
         *我们可以使用request.getMethod()来获取该请求的方式，然后在配合角色所允许的权限路径进行判断和授权操作！
         *如果能获取到Principal对象不为空证明，授权已经通过*/
        if (principal != null && principal instanceof UserDetails) {
            // 登录的用户名
            String userName = ((UserDetails) principal).getUsername();
            //获取请求登录的url
            Set<String> urls = new HashSet<>();//用户具备的系统资源集合，从数据库读取
            Set<String> curds = new HashSet<>();//用户具备的系统资源集合，从数据库读取
            SysUser sysUser = sysUserRepository.findByName(userName);
            try {
                for (SysRole role : sysUser.getRoles()) {
                   for (SysPermission permission : role.getPermissions()) {
                       urls.add(permission.getUrl());
                       //curds.add(permission.getPermission());
                       curds.add(permission.getPermission()+permission.getUrl());
                   }

               }
            } catch (Exception e) {
                e.printStackTrace();
            }

            //urls.add("/sys/user/add");

            for (String url : urls) {
                if (AntPathMatcher.match(url, request.getRequestURI())) {

                    hasPermission = true;
                    break;
                }
            }
        }

        return hasPermission;
    }
}
~~~

3）配置 HttpSecurity

在继承 WebSecurityConfigurerAdapter 的类中重写 `void configure(HttpSecurity http)` 方法，添加如下代码

~~~java
.antMatchers("/admin/**").access("@rbacService.check(request.authentication)")
~~~

这里注意，@rbacService 接口的名字是服务实现上定义的名字，即注解 @Component("rbacService") 定义的参数。具体代码如下：

~~~java
@EnableGlobalMethodSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    http.formLogin()
        .authorizeRequests()
        .antMatchers("/admin").permitAll()
        // 自定义授权策略
        .antMatchers("/admin/rbac").access("@rbacService.hasPermission(request,authentication)")
~~~

4）创建实体，添加测试数据

3 个实体，分别是用户、权限和角色实体，创建完成后需要添加数据，点击左下角 code 查看代码

5）测试

访问 "/admin/rbac" ，会提示无权访问，跳转到登录页面

在登录页面输入用户名，密码登录，会提示登录成功

访问 "/admin/rbac" ，提示访问成功