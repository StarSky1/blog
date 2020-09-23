[pixiv: 032]: # 'https://cdn.jsdelivr.net/gh/starsky1/poi/2020/32.png'

最近的工作是在使用 springsecuriy 进行认证和授权的项目中集成 cas 客户端，由于对 springsecurity 的不了解，导致工作进展缓慢。于是，我开始恶补 springsecurity 的原理和使用方法。下面，是我这几天的学习成果，有不足和错误的地方，欢迎大家来指正。

## springsecurity 的过滤器链

springsecurity 通过一系列过滤器，即使不做配置，它也默认实现了许多功能，比如：

- 任何与应用的互动都需要一个已认证的用户
- 为你创建一个默认的登录表单
- 让通过用户名和密码登录应用的用户，使用基于表单的认证
- 使用 BCrypt 保护你的密码存储
- 让用户可以登出
- CSRF（跨站请求攻击）阻止
- 会话固定保护
- 安全请求头集成（为机密的请求使用http强制安全传输技术，[X-Content-Type-Options](https://msdn.microsoft.com/en-us/library/ie/gg622941(v=vs.85).aspx) 集成，静态资源缓存控制，[X-XSS-Protection](https://msdn.microsoft.com/en-us/library/dd565647(v=vs.85).aspx)（跨站脚本攻击） 集成，X-Frame-Options 集成以防范 [Clickjacking](https://en.wikipedia.org/wiki/Clickjacking) 攻击）

springsecurity 对 servlet 容器的支持是基于servlet的过滤器，所以通常首先了解一下过滤器扮演的角色是有帮助的。下面的图片展示了单个 http 请求处理处理程序的典型分层。

![filterchain](https://cdn.jsdelivr.net/gh/starsky1/poi/2020/filterchain.png)

### 过滤器链

客户端发送一个请求到应用，servlet容器会创建一个包含所有处理这个 http 请求 uri 的过滤器和 servlet 的过滤器链。在 springmvc 的应用中，servlet 指的是 [`DispatcherServlet`](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-servlet) 的一个实例。最多一个 servlet 能处理一个 HttpServletRequest 和 HttpServletResponse。然而，可以使用多个过滤器去做些事情，比如：

- 阻止下游的过滤器或 servlet 被调用。在这种情况下，过滤器通常会编写 HttpServletResponse,进行请求响应。

- 修改将被下游过滤器和 servlet 使用的 HttpServletRequest 或 HttpServletResponse。

  过滤器链使用案例

  ```java
  public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
      // do something before the rest of the application
      chain.doFilter(request, response); // invoke the rest of the application
      // do something after the rest of the application
  }
  ```

  因为一个过滤器只能影响下游的过滤器和 servlet，所以每个过滤器被调用的次序是非常重要的。

### DelegatingFilterProxy

spring 提供了一个叫做 DelegatingFilterProxy 的过滤器实现，它能够桥接 servlet容器和 spring 的上下文。servlet 容器允许使用它自己的标准来注册过滤器，但是它访问不到 spring 定义的 bean。DelegatingFilterProxy 能使用标准的 servlet 容器机制来注册，但是它将委托所有的工作给实现了过滤器接口的 spring bean。

这里的图片展示了 DelegatingFilterProxy 如何适应 过滤器和过滤器链。

![delegatingfilterproxy](https://cdn.jsdelivr.net/gh/starsky1/poi/2020/delegatingfilterproxy.png)

DelegatingFilterProxy 从 spring 上下文环境寻找 Bean Filter0，然后调用 Bean Filter0。DelegatingFilterProxy 的伪代码看起来像下面这样。

```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
    // Lazily get Filter that was registered as a Spring Bean
    // For the example in DelegatingFilterProxy delegate is an instance of Bean Filter0
    Filter delegate = getFilterBean(someBeanName);
    // delegate work to the Spring Bean
    delegate.doFilter(request, response);
}
```

DelegatingFilterProxy 的另一个好处是，它允许延迟寻找 过滤器 Bean 实例。这是重要的，因为 servlet 容器需要注册过滤器实例在容器能启动之前。然而，spring 通常使用一个 ContextLoaderListener 去加载 spring beans，它不会立即加载 bean，直到这个过滤器实例需要被注册时。

###  FilterChainProxy

spring security  的 servlet 支持被包含在  FilterChainProxy 中。 FilterChainProxy 是一个特殊的过滤器，它能够通过 [`SecurityFilterChain`](https://docs.spring.io/spring-security/site/docs/5.4.0/reference/html5/#servlet-securityfilterchain) 委托许多过滤器实例。由于 FilterChainProxy 是一个 Bean，它通常被包裹在 [DelegatingFilterProxy](https://docs.spring.io/spring-security/site/docs/5.4.0/reference/html5/#servlet-delegatingfilterproxy) 中。

![filterchainproxy](https://cdn.jsdelivr.net/gh/starsky1/poi/2020/filterchainproxy.png)

### SecurityFilterChain

[`SecurityFilterChain`](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/SecurityFilterChain.html) 被 [FilterChainProxy](https://docs.spring.io/spring-security/site/docs/5.4.0/reference/html5/#servlet-filterchainproxy) 使用，去决定哪一个 spring security 的过滤器应该被调用去处理这个请求。

![securityfilterchain](https://cdn.jsdelivr.net/gh/starsky1/poi/2020/securityfilterchain.png)

这些 security 过滤器是典型的 spring beans，但是他们通过 FilterChainProxy 来注册而不是 [DelegatingFilterProxy](https://docs.spring.io/spring-security/site/docs/5.4.0/reference/html5/#servlet-delegatingfilterproxy)。FilterChainProxy 提供了许多优点相对于直接在 servlet 容器注册或 [DelegatingFilterProxy](https://docs.spring.io/spring-security/site/docs/5.4.0/reference/html5/#servlet-delegatingfilterproxy) 。首先，它提供了一个起点，为所有的 spring security 的 servlet 支持。因此，如果你正在尝试去解决 spring security 的 servlet 支持问题，添加一个 Debug 点在 FilterChainProxy 将是一个很好的起始打桩点。

第二，因为 FilterChainProxy 是 spring security 使用的中心，它能执行那些不被视为可选的任务。举例来说，它能清理掉 SecurityContext 去避免内存泄露。它也会应用 spring security 的  [`HttpFirewall`](https://docs.spring.io/spring-security/site/docs/5.4.0/reference/html5/#servlet-httpfirewall) 去保护应用抵抗某些类型的攻击。

另外，它提供了更多的灵活来决定何时一个 SecurityFilterChain 应该被调用。在一个 servlet 容器中，过滤器被调用是基于单独的URL。然而，FilterChainProxy 能决定调用基于 HttpServletRequest 中的任何事情，通过RequestMatcher 接口的匹配帮助。

实际上，FilterChainProxy 能被使用去决定哪一个 SecurityFilterChain 应该被使用。这将允许提供完全独立的配置，为你的应用的不同片段。

![multi securityfilterchain](https://cdn.jsdelivr.net/gh/starsky1/poi/2020/multi-securityfilterchain.png)

在这个图中，FilterChainProxy 决定哪一个 SecurityFilterChain 应该被使用。只有首个匹配的 SecurityFilterChain 将被调用。如果一个 URL `/api/messages/` 被请求，它将首先匹配 SecurityFilterChain0 的模式 `/api/**`，所以只有 SecurityFilterChain0 会被调用，即使这个请求也匹配 SecurityFilterChainn。如果一个URL `/messages/` 被请求，它将不匹配 SecurityFilterChain0 的模式 `/api/**`，所以 FilterChainProxy 将继续尝试每个 SecurityFilterChain。假设没有其他的匹配，SecurityFilterChainn 将被调用。

注意，SecurityFilterChain0 只有三个 security 过滤器实例被配置。然而，SecurityFilterChainn 有四个 security 过滤器被配置。这是重要的去注意每个 SecurityFilterChain 是独立的和配置隔离的。实际上，一个 SecurityFilterChain 可能有零个 security 过滤器，如果应用想让 spring security 忽略某些请求。

### Security Filters

security 过滤器被插入  [FilterChainProxy](https://docs.spring.io/spring-security/site/docs/5.4.0/reference/html5/#servlet-filterchainproxy) 通过  [SecurityFilterChain](https://docs.spring.io/spring-security/site/docs/5.4.0/reference/html5/#servlet-securityfilterchain) 的 API。过滤器的顺序是重要的。通常没有必要知道 Spring security 过滤器的顺序。然而，有时候知道顺序是有帮助的。

下面是 spring security 过滤器排序的综合列表（只列了常用的过滤器，完整的链条请查看 [Security Filters](https://docs.spring.io/spring-security/site/docs/5.4.0/reference/html5/#servlet-security-filters)）：

- <span style="color: blue;">WebAsyncManagerIntegrationFilter</span>

  （异步方式）提供了对securityContext和WebAsyncManager的集成。方式是通过SecurityContextCallableProcessingInterceptor的beforeConcurrentHandling(NativeWebRequest, Callable)方法来将SecurityContext设置到Callable上。其实就是把SecurityContext设置到异步线程中，使其也能获取到用户上下文认证信息。

- <span style="color: blue;background-color: yellow;">SecurityContextPersistenceFilter </span>

  （同步方式）在请求之前从SecurityContextRepository（默认实现是HttpSessionSecurityContextRepository）获取信息并填充SecurityContextHolder（如果没有，则创建一个新的ThreadLocal的SecurityContext），并在请求完成并清空SecurityContextHolder并更新SecurityContextRepository。

  在Spring Security中，虽然安全上下文信息被存储于Session中，但实际的Filter中不应直接操作Session（过滤器一般负责核心的处理流程，而具体的业务实现，通常交给其中聚合的其他实体类），而是用如HttpSessionSecurityContextRepository中loadContext()，saveContext()来存取session。

- <span style="color: blue;">HeaderWriterFilter </span>

  用来给http响应添加一些Header，比如X-Frame-Options，X-XSS-Protection*，X-Content-Type-Options。

- <span style="color: blue;">CsrfFilter</span>

   默认开启，用于防止csrf攻击的过滤器

- <span style="color: blue;">LogoutFilter</span>

  处理注销的过滤器

- CasAuthenticationFilter 

  用于对使用cas统一认证平台进行认证的用户进行票据校验，默认处理校验地址为"/login/cas"

- <span style="color: blue;background-color: yellow;">[`UsernamePasswordAuthenticationFilter`](https://docs.spring.io/spring-security/site/docs/5.4.0/reference/html5/#servlet-authentication-usernamepasswordauthenticationfilter)</span>

  表单提交了username和password，被封装成UsernamePasswordAuthenticationToken对象进行一系列的认证，便是主要通过这个过滤器完成的，即调用AuthenticationManager.authenticate()。在表单认证的方法中，这是最最关键的过滤器。具体过程是：

  （1）调用AbstractAuthenticationProcessingFilter.doFilter()方法执行过滤器

  （2）调用UsernamePasswordAuthenticationFilter.attemptAuthentication()方法

  （3）调用AuthenticationManager.authenticate()方法（实际上委托给AuthenticationProvider的实现类来处    理）

- <span style="color: blue;">DefaultLoginPageGeneratingFilter & DefaultLogoutPageGeneratingFilter</span>

  如果没有配置/login及login page, 系统则会自动配置这两个Filter。

- <span style="color: blue;">RequestCacheAwareFilter</span>

  内部维护了一个RequestCache，用于缓存request请求

- <span style="color: blue;">SecurityContextHolderAwareRequestFilter</span>

  此过滤器对ServletRequest进行了一次包装，使得request具有更加丰富的API（populates the ServletRequest with a request wrapper which implements servlet API security methods）

- <span style="color: blue;">RememberMeAuthenticationFilter</span>

  提供 rememberme 服务

- <span style="color: blue;">AnonymousAuthenticationFilter</span>

   匿名身份过滤器，spring security为了兼容未登录的访问，也走了一套认证流程，只不过是一个匿名的身份。它位于身份认证过滤器（e.g. UsernamePasswordAuthenticationFilter）之后，意味着只有在上述身份过滤器执行完毕后，SecurityContext依旧没有用户信息，AnonymousAuthenticationFilter该过滤器才会有意义。

- <span style="color: blue;background-color: yellow;">[`ExceptionTranslationFilter`](https://docs.spring.io/spring-security/site/docs/5.4.0/reference/html5/#servlet-exceptiontranslationfilter) </span>

  异常转换过滤器，这个过滤器本身不处理异常，而是将认证过程中出现的异常（AccessDeniedException and AuthenticationException）交给内部维护的一些类去处理。它
  位于整个springSecurityFilterChain的后方，用来转换整个链路中出现的异常，将其转化，顾名思义，转化以意味本身并不处理。一般其只处理两大类异常：AccessDeniedException访问异常和AuthenticationException认证异常。

  它将Java中的异常和HTTP的响应连接在了一起，这样在处理异常时，我们不用考虑密码错误该跳到什么页面，账号锁定该如何，只需要关注自己的业务逻辑，抛出相应的异常便可。如果该过滤器检测到AuthenticationException，则将会交给内部的AuthenticationEntryPoint去处理，如果检测到AccessDeniedException，需要先判断当前用户是不是匿名用户，如果是匿名访问，则和前面一样运行AuthenticationEntryPoint，否则会委托给AccessDeniedHandler去处理，而AccessDeniedHandler的默认实现，是AccessDeniedHandlerImpl。

- <span style="color: blue;background-color: yellow;">[`FilterSecurityInterceptor`](https://docs.spring.io/spring-security/site/docs/5.4.0/reference/html5/#servlet-authorization-filtersecurityinterceptor)</span>

  用于保护Http 资源的，它需要一个AccessDecisionManager和一个AuthenticationManager 的

  引用。它会从 SecurityContextHolder 获取 Authentication，然后通过 SecurityMetadataSource 可以得知当前请求是否在请

  求受保护的资源。对于请求那些受保护的资源，如果Authentication.isAuthenticated()返回false或者FilterSecurityInterceptor

  的alwaysReauthenticate 属性为 true，那么将会使用其引用的 AuthenticationManager 再认证一次，认证之后再使用认证后

  的 Authentication 替换 SecurityContextHolder 中拥有的那个。然后就是利用 AccessDecisionManager 进行权限的检查；

## springsecurity 集成 cas 客户端

### 依赖

创建 maven 项目，在 pom.xml 文件中加入 springboot-starter-web、starter-springsecurity 这两个依赖。推荐使用 [阿里云maven中央仓库](https://maven.aliyun.com/mvn/guide) ,下载速度很快。

```xml
<parent>
	<groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.4.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- security starter Poms -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
    <!-- security 对CAS支持 -->
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-cas</artifactId>
        </dependency>
    <!-- security taglibs -->  
        <dependency>  
            <groupId>org.springframework.security</groupId>  
            <artifactId>spring-security-taglibs</artifactId>  
        </dependency>  
        <!-- 热加载 -->  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-devtools</artifactId>  
            <optional>true</optional>  
        </dependency>  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-configuration-processor</artifactId>  
            <optional>true</optional>  
        </dependency>  
    	<dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.0</version>
        </dependency>
</dependencies>
```

### 配置你的安全设置

在 src/main/java 文件夹下，创建 package com.zs.security，在包下创建 ZsMain.java 文件，写入

```java
@SpringBootApplication
public class ZsMain {
  public static void main(String[] args){SpringApplication.run(ZsMain.class,args);}
}
```

在 security 的下一级，创建 package config，创建 SecurityConfig.java，继承 WebSecurityConfigurerAdapter抽象类。

```java
@Configuration
@EnableWebSecurity //启用web权限
@EnableGlobalMethodSecurity(prePostEnabled = true) //启用方法验证
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    private CasProperties casProperties;

    /**定义认证用户信息获取来源，密码校验规则等*/
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        super.configure(auth);
        auth.authenticationProvider(casAuthenticationProvider());
        //inMemoryAuthentication 从内存中获取
        //auth.inMemoryAuthentication().withUser("lidd").password("123456").roles("USER")
        //.and().withUser("admin").password("123456").roles("ADMIN");

        //jdbcAuthentication从数据库中获取，但是默认是以security提供的表结构
        //usersByUsernameQuery 指定查询用户SQL
        //authoritiesByUsernameQuery 指定查询权限SQL
        //auth.jdbcAuthentication().dataSource(dataSource).usersByUsernameQuery(query).authoritiesByUsernameQuery(query);

        //注入userDetailsService，需要实现userDetailsService接口
        //auth.userDetailsService(userDetailsService);
    }

    /**定义安全策略*/
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()//配置安全策略
                .antMatchers("/","/hello").permitAll()//定义/请求不需要验证
                .anyRequest().authenticated()//其余的所有请求都需要验证
                .and()
                .logout().logoutUrl(casProperties.getClient().getLogoutUrl())
                .permitAll()//定义logout不需要验证
                .and()
                .formLogin();//使用form表单登录

        http.exceptionHandling().authenticationEntryPoint(casAuthenticationEntryPoint())
                .and()
                .addFilter(casAuthenticationFilter())
                .addFilterBefore(casLogoutFilter(), LogoutFilter.class)
                .addFilterBefore(singleSignOutFilter(), CasAuthenticationFilter.class);

        //http.csrf().disable(); //禁用CSRF
    }

    /**认证的入口*/
    @Bean
    public CasAuthenticationEntryPoint casAuthenticationEntryPoint() {
        CasAuthenticationEntryPoint casAuthenticationEntryPoint = new CasAuthenticationEntryPoint();
        casAuthenticationEntryPoint.setLoginUrl(casProperties.getServer().getLoginUrl());
        casAuthenticationEntryPoint.setServiceProperties(serviceProperties());
        return casAuthenticationEntryPoint;
    }

    /**指定service相关信息*/
    @Bean
    public ServiceProperties serviceProperties() {
        ServiceProperties serviceProperties = new ServiceProperties();
        serviceProperties.setService(casProperties.getClient().getHost() + casProperties.getClient().getLoginProcessesUrl());
        serviceProperties.setAuthenticateAllArtifacts(true);
        return serviceProperties;
    }

    /**CAS认证过滤器*/
    @Bean
    public CasAuthenticationFilter casAuthenticationFilter() throws Exception {
        CasAuthenticationFilter casAuthenticationFilter = new CasAuthenticationFilter();
        casAuthenticationFilter.setAuthenticationManager(authenticationManager());
        casAuthenticationFilter.setFilterProcessesUrl(casProperties.getClient().getLoginProcessesUrl());
        return casAuthenticationFilter;
    }

    /**cas 认证 Provider*/
    @Bean
    public CasAuthenticationProvider casAuthenticationProvider() {
        CasAuthenticationProvider casAuthenticationProvider = new CasAuthenticationProvider();
        casAuthenticationProvider.setAuthenticationUserDetailsService(customUserDetailsService());
        //casAuthenticationProvider.setUserDetailsService(customUserDetailsService()); //这里只是接口类型，实现的接口不一样，都可以的。
        casAuthenticationProvider.setServiceProperties(serviceProperties());
        casAuthenticationProvider.setTicketValidator(cas20ServiceTicketValidator());
        casAuthenticationProvider.setKey("casAuthenticationProviderKey");
        return casAuthenticationProvider;
    }

    /*@Bean
    public UserDetailsService customUserDetailsService(){
        return new CustomUserDetailsService();
    }*/

    /**用户自定义的AuthenticationUserDetailsService*/
    @Bean
    public AuthenticationUserDetailsService<CasAssertionAuthenticationToken> customUserDetailsService(){
        return new CustomUserDetailsService();
    }

    @Bean
    public Cas20ServiceTicketValidator cas20ServiceTicketValidator() {
        return new Cas20ServiceTicketValidator(casProperties.getServer().getApiHost());
    }

    /**单点登出过滤器*/
    @Bean
    public SingleSignOutFilter singleSignOutFilter() {
        SingleSignOutFilter singleSignOutFilter = new SingleSignOutFilter();
        singleSignOutFilter.setCasServerUrlPrefix(casProperties.getServer().getApiHost());
        singleSignOutFilter.setIgnoreInitConfiguration(true);
        return singleSignOutFilter;
    }

    /**请求单点退出过滤器*/
    @Bean
    public LogoutFilter casLogoutFilter() {
        LogoutFilter logoutFilter = new LogoutFilter(casProperties.getServer().getLogoutUrl(), new SecurityContextLogoutHandler());
        logoutFilter.setFilterProcessesUrl(casProperties.getClient().getLogoutUrl());
        return logoutFilter;
    }
    
    // 忽略ssl证书
    @Bean
    public FilterRegistrationBean<IgnoreSSLValidateFilter> ignoreSSLValidateFilter() {
        FilterRegistrationBean<IgnoreSSLValidateFilter> registrationBean = new FilterRegistrationBean<>();
        registrationBean.setFilter(new IgnoreSSLValidateFilter());
        registrationBean.setName("ignoreSSLValidateFilter");
        registrationBean.setOrder(0);
        registrationBean.setUrlPatterns(Arrays.asList("/*"));
        return registrationBean;
    }

}
```

### 创建访问入口 IndexController

```java
@RestController
public class IndexController {


    @RequestMapping("/")
    public String index() {
        return "访问了首页哦";
    }

    @RequestMapping("/hello")
    public String hello() {
        return "不验证哦";
    }

    @PreAuthorize("hasAuthority('TEST')")//有TEST权限的才能访问
    @RequestMapping("/security")
    public String security() {
        return "hello world security";
    }

    @PreAuthorize("hasAuthority('ADMIN')")//必须要有ADMIN权限的才能访问
    @RequestMapping("/authorize")
    public String authorize() {
        return "有权限访问";
    }

    /**这里注意的是，TEST与ADMIN只是权限编码，可以自己定义一套规则，根据实际情况即可*/
```

### 创建 application.yml 配置文件

```yaml
#CAS服务
cas:
  server:
    host: https://yj.com:8443/cas
    login-url: ${cas.server.host}/login
    logout-url: ${cas.server.host}/logout?service=${cas.client.host}
    api-host: https://yj.com:8443/cas
  # cas客户端，即应用服务地址
  client:
    host: http://localhost:8080/boot-cas
    login-processes-url: /login/cas
    system-param: ?system=test11
    login-url: /login
    logout-url: /logout

server:
  port: 8080
  servlet:
    context-path: /boot-cas

spring:
  application:
    name: boot-cas
logging:
  level:
    org.springframework: DEBUG

```

### 创建 CasProperties 类，加载 cas 配置信息

```java
@Getter
@Setter
@Component // 这里保证 CasProperties 可以作为Spring的bean注册到容器中
@ConditionalOnProperty(prefix = "cas.server", name = "host")
@ConfigurationProperties(value = "cas")
public class CasProperties {

    /**
     * CAS客户端配置
     */
    private Client client = new Client();

    /**
     * CAS服务端配置
     */
    private Server server = new Server();

    @Getter
    @Setter
    public static class Client {

        /**
         * CAS登录处理URL
         */
        private String loginProcessesUrl;

        private String host;

        private String systemParam;

        private String logoutUrl;
    }

    @Getter
    @Setter
    public static class Server {

        private String host;

        private String loginUrl;

        private String logoutUrl;

        private String apiHost;
    }

}
```

### 定义 CustomUserDetailsService 类

```java
public class CustomUserDetailsService implements AuthenticationUserDetailsService<CasAssertionAuthenticationToken> {

    @Override
    public UserDetails loadUserDetails(CasAssertionAuthenticationToken token) throws UsernameNotFoundException {
        System.out.println("当前的用户名是："+token.getName());
        /*这里我为了方便，就直接返回一个用户信息，实际当中这里修改为查询数据库或者调用服务什么的来获取用户信息*/
        UserInfo userInfo = new UserInfo();
        userInfo.setUsername("admin");
        userInfo.setName("123456");
        Set<AuthorityInfo> authorities = new HashSet<AuthorityInfo>();
        AuthorityInfo authorityInfo = new AuthorityInfo("TEST");
        authorities.add(authorityInfo);
        userInfo.setAuthorities(authorities);
        return userInfo;
    }
}
```

### 定义 AuthorityInfo 类，用于加载当前登录用户的权限信息，实现 GrantedAuthority 接口

```java
public class AuthorityInfo implements GrantedAuthority {

    private static final long serialVersionUID = -175781100474818800L;

    /**
     * 权限CODE
     */
    private String authority;

    public AuthorityInfo(String authority) {
        this.authority = authority;
    }

    @Override
    public String getAuthority() {
        return authority;
    }

    public void setAuthority(String authority) {
        this.authority = authority;
    }
```

### 定义 UserInfo 类，用于加载当前用户信息，实现 UserDetails 接口

```java
@Getter
@Setter
public class UserInfo implements UserDetails {
    private static final long serialVersionUID = -1041327031937199938L;

    /**
     * 用户ID
     */
    private Long id;

    /**
     * 用户名称
     */
    private String name;

    /**
     * 登录名称
     */
    private String username;

    /**
     * 登录密码
     */
    private String password;

    private boolean isAccountNonExpired = true;

    private boolean isAccountNonLocked = true;

    private boolean isCredentialsNonExpired = true;

    private boolean isEnabled = true;

    private Set<AuthorityInfo> authorities = new HashSet<AuthorityInfo>();
}
```

### 定义 IgnoreSSLValidateFilter ，忽略 SSL 认证

```java
public class IgnoreSSLValidateFilter implements Filter {
    static {
        //执行设置，禁用ssl认证
        try {
            TrustManager trustAllCerts = new X509TrustManager() {
                public X509Certificate[] getAcceptedIssuers() {
                    return null;
                }

                public void checkClientTrusted(X509Certificate[] arg0, String arg1)
                        throws CertificateException {
                }

                public void checkServerTrusted(X509Certificate[] arg0, String arg1)
                        throws CertificateException {
                }
            };
            SSLContext sc = SSLContext.getInstance("SSL");
            sc.init(null, new TrustManager[]{trustAllCerts}, new SecureRandom());
            HttpsURLConnection.setDefaultSSLSocketFactory(sc.getSocketFactory());
            HttpsURLConnection.setDefaultHostnameVerifier((hostname, session) -> true);
        } catch (Exception e) {
        }
    }


    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {

    }
```

### 本地启动一个 cas 服务端

具体流程，请查看 [SpringBoot整合CAS单点登录](https://blog.csdn.net/qq_35618489/article/details/88186959)



## 引用文章

- [spring-security5.4.0版本参考文档](https://docs.spring.io/spring-security/site/docs/5.4.0/reference/html5/#servlet-security-filters)
- [Spring Security(2)：过滤器链（filter chain）的介绍](https://www.cnblogs.com/storml/p/10937486.html)
- [spring security过滤器链及认证流程](https://blog.csdn.net/zhong_csdn/article/details/79447185)
- [springboot+cas单点登录](https://www.jianshu.com/p/67db23e9d005)
-  [SpringBoot整合CAS单点登录](https://blog.csdn.net/qq_35618489/article/details/88186959)

