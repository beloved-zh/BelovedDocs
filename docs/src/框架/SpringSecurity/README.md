# 

> 官网地址：https://spring.io/projects/spring-security

# 整体架构

在 `SpringSecurity` 的架构设计中，认证 `Authentication` 和授权 `Authorization` 是分开的，无论什么样的认证方式，都不影响授权，两个是独立存在的。

![image-20220313140953884](image/image-20220313140953884.png)

## 认证

### AuthenticationManager

在 SpringSecurity 中认证是由 `AuthenticationManager` 接口来负责的。

![image-20220313142216247](image/image-20220313142216247.png)

```java
public interface AuthenticationManager {
	Authentication authenticate(Authentication authentication) throws AuthenticationException;
}
```

- 返回 `Authentication` 表示认证成功
- 返回 `AuthenticationException` 异常，表示认证失败

AuthenticationManager 主要的实现类为 ProviderManager ，在 ProviderManager 中管理了众多 AuthenticationProvider 实例。在一次完整的认证流程中，SpringSecurity 允许存在多个 AuthenticationProvider，用来实现多种认证方式，这些 AuthenticationProvider 都是由 ProviderManager 进行统一管理。

![image-20220313142808166](image/image-20220313142808166.png)



### Authentication

认证及认证成功的信息主要由 `Authentication` 的实现类进行保存的

![image-20220313143055124](image/image-20220313143055124.png)

- getAuthorities：获取用户权限信息
- getCredentials：获取用户凭证信息，一般指密码
- getDetails：获取用户详细信息
- getPrincipal：获取用户身份信息，用户名，用户对象等
- isAuthenticated：用户是否认证成功

### SecurityContextHolder

SecurityContextHolder 用来获取登录之后用户信息。SpringSecurity 会将登录用户数据保存在 Session 中。当用户登录成后，SpringSecurity 会将登录成功的用户信息保存到 SecurityContextHolder 中。SecurityContextHolder 中的数据保存默认是通过 ThreadLocal 来实现的，使用 ThreadLocal 创建的变量只能被当前线程访问，不能被其他线程访问和修改，也就是用户数据和请求线程绑定在一起。当登录请求处理完毕后，SpringSecurity 会将 SecurityContextHolder 中的数据拿出来保存到 Session 中，同时将 SecurityContexHolder 中的数据清空。以后每当有请求到来时，SpringSecurity 就会先从 Session 中取出用户登录数据，保存到 SecurityContextHolder 中，方便在该请求的后续处理过程中使用，同时在请结束时将 SecurityContextHolder 中的数据拿出来保存到 Session 中，然后将SecurityContextHolder 中的数据清空。这一策略非常方便用户在 Controller、Service 层以及任何代码中获取当前登录用户数据。

## 授权

### AccessDecisionManager

> AccessDecisionManager（访问决策管理器），用来决定此次访问是否被允许

![image-20220313145049657](image/image-20220313145049657.png)

### AccessDecisionVoter

> AccessDecisionVoter（访问决定投票器），投票器会检查用户是否具备应有的角色，进而投出赞成、反对或弃权

![image-20220313145230556](image/image-20220313145230556.png)

AccessDecisionVoter 和 AccessDecisionManager 都有众多的实现类，在 AccessDecisionManager 中会挨个遍历 AccessDecisionVoter，进而决定是否被用户访问，因而 AccessDecisionVoter 和 AccessDecisionManager 两者的关系类似于 AuthenticationProvider 和 ProviderManager 的关系

### ConfigAttribute

> ConfigAttribute：用来保存授权时的角色信息

![image-20220313145703726](image/image-20220313145703726.png)

在 SpringSecurity 中，用户请求一个资源需要的角色会被封装成一个 ConfigAttribute 对象，在 ConfigAttribute 中只有一个 `getAttribute` 方法，该方法返回一个 String 字符串，就是角色的名称。投票器 AccessDecisionVoter 所做的事，就是比较用户所具备的角色和请求资源所需的 ConfigAttribute 的关系。

# 环境搭建

> - SpringBoot
>
> - SpringSecurity

## 创建应用

- 创建 SpringBoot 应用
- 编写测试 Controller
- 启动测试

![image-20220313161436489](image/image-20220313161436489.png)

## 整合 SpringSecurity

**添加依赖**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

再次启动访问 `/hello` 跳转到一个登录页。

![image-20220313162033330](image/image-20220313162033330.png)

观察控制台生成了一个默认密码。

![image-20220313162104509](image/image-20220313162104509.png)

登录系统：

- 默认用户名：user
- 默认密码：控制台生成

![image-20220313162206679](image/image-20220313162206679.png)

> - 只引入依赖没有任何配置所有请求需要认证？
> - 登录页面从何而来？
> - 默认用户的数据源在哪里？

## 实现原理

> 官网解释：https://docs.spring.io/spring-security/reference/servlet/architecture.html

在 SpringSecurity 中 `认证、授权` 等功能都是基于 `过滤器` 完成的。

![securityfilterchain](image/securityfilterchain.png)

默认的过滤器并不是直接放在 Web 项目的原生的过滤器链中，而是通过一个 [FilterChainProxy](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-filterchainproxy) 嵌入到 Web 项目的原生过滤器链中。 [FilterChainProxy](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-filterchainproxy) 作为一个顶层的管理者，将统一管理  [Security Filter](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-security-filters)。[FilterChainProxy](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-filterchainproxy) 本身是通过 Spring 框架提供的 [DelegatingFilterProxy](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-delegatingfilterproxy) 整合到原生的过滤器链中。

## SpringFilters

> https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-security-filters

SpringSecurity 提供了非常多的过滤器，其中有的是默认加载的。

| 过滤器                                                       | 作用                                                      | 是否默认 |
| ------------------------------------------------------------ | --------------------------------------------------------- | -------- |
| ChannelProcessingFilter                                      | 过滤请求协议HTTP、HTTPS                                   | NO       |
| `WebAsyncManagerIntegrationFilter`                           | 将 WebAsyncManager 与 SpringSecurity 上下文进行集成       | YES      |
| `SecurityContextPersistenceFilter`                           | 在处理请求之前，将安全信息加载到 SecurityContextHolder 中 | YES      |
| `HeaderWriterFilter`                                         | 处理头信息加入响应中                                      | YES      |
| CorsFilter                                                   | 处理跨域问题                                              | NO       |
| `CsrfFilter`                                                 | 处理 CSRF 攻击                                            | YES      |
| `LogoutFilter`                                               | 处理注销登录                                              | YES      |
| OAuth2AuthorizationRequestRedirectFilter                     | 处理 OAuth2 认证重定向                                    | NO       |
| Saml2WebSsoAuthenticationRequestFilter                       | 处理 SAML 认证                                            | NO       |
| X509AuthenticationFilter                                     | 处理 X509 认证                                            | NO       |
| AbstractPreAuthenticatedProcessingFilter                     | 处理预认证问题                                            | NO       |
| CasAuthenticationFilter                                      | 处理 CAS 单点登录                                         | NO       |
| OAuth2LoginAuthenticationFilter                              | 处理 OAuth2 认证                                          | NO       |
| Saml2WebSsoAuthenticationFilter                              | 处理 SAML 认证                                            | NO       |
| [`UsernamePasswordAuthenticationFilter`](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/form.html#servlet-authentication-usernamepasswordauthenticationfilter) | 处理表单登录                                              | YES      |
| OpenIDAuthenticationFilter                                   | 处理 OpenID 认证                                          | NO       |
| `DefaultLoginPageGeneratingFilter`                           | 处理默认登录页面                                          | YES      |
| `DefaultLogoutPageGeneratingFilter`                          | 处理默认注销页面                                          | YES      |
| ConcurrentSessionFilter                                      | 处理 Session 有效期                                       | NO       |
| [DigestAuthenticationFilter](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/digest.html#servlet-authentication-digest) | 处理 HTTP 摘要认证                                        | NO       |
| BearerTokenAuthenticationFilter                              | 处理 OAuth2 认证的 Access Token                           | NO       |
| [`BasicAuthenticationFilter`](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/basic.html#servlet-authentication-basic) | 处理 HTTPBasic 登录                                       | YES      |
| `RequestCacheAwareFilter`                                    | 处理请求缓存                                              | YES      |
| `SecurityContextHolderAwareRequestFilter`                    | 包装原始请求                                              | YES      |
| JaasApiIntegrationFilter                                     | 处理 JASS 认证                                            | NO       |
| RememberMeAuthenticationFilter                               | 处理 RememberMe 登录                                      | NO       |
| `AnonymousAuthenticationFilter`                              | 处理匿名认证                                              | YES      |
| OAuth2AuthorizationCodeGrantFilter                           | 处理 OAuth2 认证中的授权码                                | NO       |
| `SessionManagementFilter`                                    | 处理 Session 并发问题                                     | YES      |
| [`AccessDeniedExpansion`](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-exceptiontranslationfilter) | 处理认证/授权 中的异常                                    | YES      |
| [`FilterSecurityInterceptor`](https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-requests.html#servlet-authorization-filtersecurityinterceptor) | 处理授权相关                                              | YES      |
| SwitchUserFilter                                             | 处理账户切换                                              | NO       |

SpringSecurity 提供了 32 个过滤器。默认情况下 SpringBoot 在对 SpringSecurity 进入自动化配置是，会创建一个名为 `SpringSecurityFilerChain` 的过滤器，并注入到 Spring 容器，这个过滤器将负责所有的安全管理，包括用户认证、授权、重定向到登录页等。具体可以参考 `WebSecurityConfiguration` 的源码

![image-20220313170020049](image/image-20220313170020049.png)

![image-20220313170531727](image/image-20220313170531727.png)

## SpringBootWebSecurityConfiguration

SpringBoot 对 Security 的自动配置类。

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnDefaultWebSecurity
@ConditionalOnWebApplication(type = Type.SERVLET)
class SpringBootWebSecurityConfiguration {

   @Bean
   @Order(SecurityProperties.BASIC_AUTH_ORDER)
   SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
      http.authorizeRequests().anyRequest().authenticated().and().formLogin().and().httpBasic();
      return http.build();
   }

}
```

默认配置：开启所有请求都拦截 且通过 from 表单方式进行认证

这就是引入 SpringSecurity 没有任何配置 且会拦截的原因。

通过上面的注解可以看出默认生效的条件为：

```java
class DefaultWebSecurityCondition extends AllNestedConditions {

   DefaultWebSecurityCondition() {
      super(ConfigurationPhase.REGISTER_BEAN);
   }

   @ConditionalOnClass({ SecurityFilterChain.class, HttpSecurity.class })
   static class Classes {

   }

   @ConditionalOnMissingBean({ WebSecurityConfigurerAdapter.class, SecurityFilterChain.class })
   static class Beans {

   }

}
```

- 条件一：classpath 中存在 SecurityFilterChain.class, HttpSecurity.class
- 条件二：没有自定义 WebSecurityConfigurerAdapter.class, SecurityFilterChain.class

WebSecurityConfigurerAdapter 这个类极其重要，SpringSecurity 核心配置都在这个类中。

![image-20220313171717122](image/image-20220313171717122.png)

自定义这个类实例，通过覆盖类中方法可达到修改SpringSecurity默认配置，可进行自定义配置。

## 流程分析

![image-20220313171911336](image/image-20220313171911336.png)

1. 请求 /hello 接口，在引入 SpringSecurity 之后会先经过一系列过滤器
2. 在请求到达 FilterSecurityInterceptor 时，发现请求并未认证。请求拦截并抛出 AccessDeniedExpansion 异常。
3. 抛出 AccessDeniedExpansion 异常会被 AccessDeniedExpansion 捕获，这个 Filter 中会调用 LoginUrlAuthenticationEntryPoint 中 commence 方法给客户端返回 302，要求客户端进行重定向到 /login 页面
4. 客户端发送 /login 请求
5. /login 请求会被刺被拦截器中 DefaultLoginPageGeneratingFilter 拦截到，并在拦截其中返回生成的登录页面

就是通过这种方式 SpringSecurity 默认过滤器中生成登录页并返回

## 默认用户生成

> 查看 SpringBootWebSecurityConfiguration#defaultSecurityFilterChain 方法表单登录

![image-20220313174118747](image/image-20220313174118747.png)

> 处理登录为 FormLoginConfigurer 类中，调用 UsernamePasswordAuthenticationFilter 这个类实例

![image-20220313174232122](image/image-20220313174232122.png)

> 查看类中 UsernamePasswordAuthenticationFilter#attemptAuthentication 方法得知实际调用 AuthenticationManager#authenticate 方法

![image-20220313174357650](image/image-20220313174357650.png)

> 调用 ProviderManager#authenticate 方法

![image-20220313174811886](image/image-20220313174811886.png)

> 调用了 ProviderManager 实现类 AbstractUserDetailsAuthenticationProvider#authenticate方法

![image-20220313175001648](image/image-20220313175001648.png)

> 最终调用实现类 DaoAuthenticationProvider 类中方法比较

![image-20220313175321689](image/image-20220313175321689.png)

**==默认实现时基于InMemoryUserDetailsManager这个类，也就是内存的实现==**

## UserDetailsService

通过源码分析 `UserDetailsService` 是顶层父接口，接口中 `loadUserByUsername` 方法是用来认证时进行用户名认证方法，默认的是基于内存实现，如果想要修改成数据库只需要自定义 `UserDetailsService` 实现，最终返回 `UserDetails` 即可。

```java
public interface UserDetailsService {
   UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

![image-20220313180148608](image/image-20220313180148608.png)

## UserDetailsServiceAutoConfiguration

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(AuthenticationManager.class)
@ConditionalOnBean(ObjectPostProcessor.class)
@ConditionalOnMissingBean(
      value = { AuthenticationManager.class, AuthenticationProvider.class, UserDetailsService.class,
            AuthenticationManagerResolver.class },
      type = { "org.springframework.security.oauth2.jwt.JwtDecoder",
            "org.springframework.security.oauth2.server.resource.introspection.OpaqueTokenIntrospector",
            "org.springframework.security.oauth2.client.registration.ClientRegistrationRepository" })
public class UserDetailsServiceAutoConfiguration {

   // ...

   @Bean
   @Lazy
   public InMemoryUserDetailsManager inMemoryUserDetailsManager(SecurityProperties properties,
         ObjectProvider<PasswordEncoder> passwordEncoder) {
      SecurityProperties.User user = properties.getUser();
      List<String> roles = user.getRoles();
      return new InMemoryUserDetailsManager(
            User.withUsername(user.getName()).password(getOrDeducePassword(user, passwordEncoder.getIfAvailable()))
                  .roles(StringUtils.toStringArray(roles)).build());
   }

   // ...

}
```

- 条件一：classpath 下存在 AuthenticationManager 类
- 条件二：系统没有提供 AuthenticationManager.class, AuthenticationProvider.class, UserDetailsService.class,
              AuthenticationManagerResolver.class 实例

默认情况下都会满足，SpringSecurity 会提供一个 InMemoryUserDetailsManager 实例

![image-20220313180707804](image/image-20220313180707804.png)

## SecurityProperties

```java
@ConfigurationProperties(prefix = "spring.security")
public class SecurityProperties {

   // ...
   public static class User {
      private String name = "user";
      private String password = UUID.randomUUID().toString();
      private List<String> roles = new ArrayList<>();
      private boolean passwordGenerated = true;
   }
   // ...
}
```

这就是默认情况下 创建的 user 及 uuid 密码

可以通过配置文件修改内存中的用户信息。

```properties
spring.security.user.name=root
spring.security.user.password=123
spring.security.user.roles=admin,user
```

## 总结

>  AuthenticationManager、ProviderManger、AuthenticationProvider 关系

​	![image-20220313181509736](image/image-20220313181509736.png)

> WebSecurityConfigurerAdapter 扩展 SpringSecurity 所有默认配置

![image-20220313181718969](image/image-20220313181718969.png)

> UserDetailService 用来修改默认认证的数据源信息

![image-20220313181815816](image/image-20220313181815816.png)

# 自定义认证

## 自定义资源权限规则

- `/index` 公共资源
- `/hello` 需要认证的资源

自定义 `WebSecurityConfigurerAdapter#configure` 方法进行实现

```java
@Configuration
public class WebSecurityConfigurer extends WebSecurityConfigurerAdapter {
	@Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .mvcMatchers("/index").permitAll()  // 放行资源要写在认证之前
                .anyRequest().authenticated()
                .and()
                .formLogin();
    }
}
```

![image-20220314215907408](image/image-20220314215907408.png)

> mvcMatchers(...)：匹配资源
>
> permitAll()：放行资源，无需认证授权即可访问
>
> anyRequest()：代表所有请求
>
> authenticated()：匹配的资源都需要认证
>
> formLogin()：开启表单认证
>
> **注意：放行资源必须在认证之前！！！**

## 自定义登录页面

> 引入thymeleaf模板引擎依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

关闭 thymeleaf 缓存

```properties
spring.thymeleaf.cache=false
```

> 定义登录页Controller

```
@Controller
public class LoginController {
    @RequestMapping("/login.html")
    public String login() {
        return "login";
    }
}
```

> 编写自定义登录页

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>用户登录</title>
</head>
<body style="text-align: center">

<h1>用户登录</h1>
<form th:action="@{/doLogin}" method="post">
    用户名：<input type="text" name="uname"> <br>
    密码：<input type="text" name="pwd"> <br>
    <input type="submit" value="登录">
</form>

</body>
</html>
```

**注意：**

- 表单 method 必须为 `post`，请求 action 默认为 `/login` 自定义为 `doLogin`
- 用户名的 name 默认为 `username` 自定义为 `ename`
- 密码的 name 默认为 `password` 自定义为 `pwd`

> 配置 SpringSecurity 核心配置类

```java
@Configuration
public class WebSecurityConfigurer extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .mvcMatchers("/index","/login.html").permitAll()  // 放行资源要写在认证之前
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .loginPage("/login.html")   // 指定登录页面
                .loginProcessingUrl("/doLogin") // 指定了自定义登录页面必须指定登录请求地址
                .usernameParameter("uname").passwordParameter("pwd") // 指定表单字段名
                //.successForwardUrl("/index") // 认证成功跳转地址    forward
                //.defaultSuccessUrl("/index")   // 认证成功跳转地址    redirect 重定向  注意：请求之前有地址，会有优先跳转之前的地址
                .defaultSuccessUrl("/index", true)  // 无论请求之前是否有地址都进行重定向
                .and()
                .csrf().disable()   // 禁止 csrf 跨站请求攻击保护
        ;
    }
}
```

**注意：**

- 指定了自定义登录页面必须指定登录请求地址
- `successForwardUrl` 和 `defaultSuccessUrl` 同时只能有一个
- `successForwardUrl` 默认是 `forward` 跳转，浏览器地址栏不会变
- `defaultSuccessUrl` 默认是 `redirect` 跳转，浏览器地址栏会发生变化
- `defaultSuccessUrl` 请求之前有地址，会有优先跳转之前的地址
- `defaultSuccessUrl` 可配置第二个参数，无论请求之前是否有地址都进行重定向

## 自定义登录成功处理

前后端分离开发中登录成功后不需要返回跳转页面，一般是返回一段JSON数据通知前端。

可通过自定义 `AuthenticationSuccessHandler` 实现

```java
public interface AuthenticationSuccessHandler {
   /**
    * Called when a user has been successfully authenticated.
    * @param request the request which caused the successful authentication
    * @param response the response
    * @param authentication the <tt>Authentication</tt> object which was created during
    * the authentication process.
    */
   void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response,
         Authentication authentication) throws IOException, ServletException;

}
```

![image-20220317223525408](image/image-20220317223525408.png)

**successForwardUrl 和 defaultSuccessUrl 也是它的子类实现**

> 自定义 AuthenticationSuccessHandler 实现

```java
/**
 * 自定义认证成功处理Handler
 */
public class MyAuthenticationSuccessHandler implements AuthenticationSuccessHandler {

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {

        JSONObject result = new JSONObject();
        result.put("status", 200);
        result.put("msg", "登录成功");
        result.put("authentication", authentication);

        // 写出数据
        response.setContentType("application/json;charset=UTF-8");
        response.getWriter().println(result);
    }
}
```

> 配置 successHandler

```java
@Configuration
public class WebSecurityConfigurer extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                // ......
                .successHandler(new MyAuthenticationSuccessHandler()) // 登录成功处理器  前后端分离处理方案
                .and()
                .csrf().disable()   // 禁止 csrf 跨站请求攻击保护
        ;
    }
}
```

![image-20220317225637941](image/image-20220317225637941.png)

## 显示登陆失败信息

> Spring Security 在登录失败后会根据具体的配置将异常信息存储到 `request` 或者 `session` 作用域中，其中的 `key` 为 `SPRING_SECURITY_LAST_EXCEPTION` ，源码可参考 `SimpleUrlAuthenticationFailureHandler`

![image-20220320220337608](image/image-20220320220337608.png)

> 显示异常信息 `login.html`

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>用户登录</title>
</head>
<body style="text-align: center">
<h5 style="color: red" th:text="${'request：' + SPRING_SECURITY_LAST_EXCEPTION}"></h5>
<h5 style="color: red" th:text="${'session：' + session.SPRING_SECURITY_LAST_EXCEPTION}"></h5>
<h1>用户登录</h1>
<form th:action="@{/doLogin}" method="post">
    用户名：<input type="text" name="uname"> <br>
    密码：<input type="text" name="pwd"> <br>
    <input type="submit" value="登录">
</form>

</body>
</html>
```

> 核心配置

```java
@Configuration
public class WebSecurityConfigurer extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
				// ......
                .and()
                .formLogin()
                // ......
                //.failureForwardUrl("/login.html") // 认证失败后跳转 forward   异常信息作用域 request
                .failureUrl("/login.html") // 认证失败后跳转 redirect 异常信息作用域 session
                .and()
                .csrf().disable()   // 禁止 csrf 跨站请求攻击保护
        ;
    }
}
```

- failureUrl、failureForwardUrl 关系 类似于 successForwardUrl、defaultSuccessUrl
- failureUrl 认证失败后重定向，异常信息在 session中
- failureForwardUrl 认证失败后 forward 跳转，异常信息在 request 

![image-20220320221550100](image/image-20220320221550100.png)

## 自定义登录失败处理

和登录成功一样，前后端分离项目，一般 JSON 数据通知，不进行页面跳转

可通过自定义 `AuthenticationFailureHandler` 实现

```java
public interface AuthenticationFailureHandler {

   /**
    * Called when an authentication attempt fails.
    * @param request the request during which the authentication attempt occurred.
    * @param response the response.
    * @param exception the exception which was thrown to reject the authentication
    * request.
    */
   void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response,
         AuthenticationException exception) throws IOException, ServletException;

}
```

![image-20220320222014065](image/image-20220320222014065.png)

**failureUrl、failureForwardUrl  也是由它的子类实现。**

> 自定义 `AuthenticationFailureHandler` 实现

```java
/**
 * 自定义登录失败处理器
 */
public class MyAuthenticationFailureHandler implements AuthenticationFailureHandler {
    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
        JSONObject result = new JSONObject();
        result.put("status", 500);
        result.put("msg", "登录失败：" + exception.getMessage());

        response.setContentType("application/json;charset=UTF-8");
        response.getWriter().println(result);
    }
}
```

> 配置 `failureHandler`

```java
@Configuration
public class WebSecurityConfigurer extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                // ......
                .failureHandler(new MyAuthenticationFailureHandler())   // 自定义认证失败后处理
                .and()
                .csrf().disable()   // 禁止 csrf 跨站请求攻击保护
        ;
    }
}
```

![image-20220320222524823](image/image-20220320222524823.png)

