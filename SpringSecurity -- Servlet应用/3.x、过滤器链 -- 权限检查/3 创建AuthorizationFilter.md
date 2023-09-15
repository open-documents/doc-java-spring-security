
AuthorizationFilter由AuthorizeHttpRequestsConfigurer向HttpSecurity的Filter数组中添加。

```java
public HttpSecurity authorizeHttpRequests(     Customizer<AuthorizeHttpRequestsConfigurer<HttpSecurity>.AuthorizationManagerRequestMatcherRegistry> authorizeHttpRequestsCustomizer) throws Exception {  
    ApplicationContext context = getContext();  
    authorizeHttpRequestsCustomizer  
         .customize(getOrApply(new AuthorizeHttpRequestsConfigurer<>(context)).getRegistry());  
    return HttpSecurity.this;  
}
```

# 创建AuthorizationFilter -- AuthorizeHttpRequestsConfigurer

由前面的分析可知，SecurityChainFilter 中的各个Filter（包含此处的 AuthorizationFilter ）具体都是在 SecurityConfigurer 中的configure()创建并向 HttpSecurity 中添加的。

AuthorizationFilter 是由 AuthorizeHttpRequestsConfigurer 创建并向 HttpSecurtiy 中添加的。

```java
public void configure(H http) {  
	// this.registry: 
	// - 构造函数初值 new AuthorizationManagerRequestMatcherRegistry(ApplicationContext)
	// - 仅支持在构造函数赋值
    AuthorizationManager<HttpServletRequest> authorizationManager = this.registry.createAuthorizationManager();  
    AuthorizationFilter authorizationFilter = new AuthorizationFilter(authorizationManager);  
    
    authorizationFilter.setAuthorizationEventPublisher(this.publisher);  
	authorizationFilter.setShouldFilterAllDispatcherTypes(this.registry.shouldFilterAllDispatcherTypes);  
    authorizationFilter.setSecurityContextHolderStrategy(getSecurityContextHolderStrategy());  

    http.addFilter(postProcess(authorizationFilter));  
}
```
## 获取所需的组件 RequestMatcherDelegatingAuthorizationManager
```java
private AuthorizationManager<HttpServletRequest> createAuthorizationManager() {  
    ... // Assert.state(...);  
    ... // Assert.state(...);  

    ObservationRegistry registry = getObservationRegistry();  
    // AuthorizationManagerRequestMatcherRegistry#this.managerBuilder:
    // - 默认初值 RequestMatcherDelegatingAuthorizationManager.builder()
    RequestMatcherDelegatingAuthorizationManager manager = postProcess(this.managerBuilder.build());  
    if (registry.isNoop()) {  
        return manager;  
    }  
    return new ObservationAuthorizationManager<>(registry, manager);  
}
```
下面接着看RequestMatcherDelegatingAuthorizationManager具体如何创建的。
```java
/* ----------------- RequestMatcherDelegatingAuthorizationManager$Builder ----------------- */
public RequestMatcherDelegatingAuthorizationManager build() {  
    return new RequestMatcherDelegatingAuthorizationManager(this.mappings);  
}
```

# 自定义 RequestMatcherDelegatingAuthorizationManager

从前面可知 AuthorizationFilter 的创建需要一个最重要的组件 RequestMatcherDelegatingAuthorizationManager，它内部维护了一个 `RequestMatcherEntry<AuthorizationManager<RequestAuthorizationContext>>` 列表 mappings。
```java
private final List<RequestMatcherEntry<AuthorizationManager<RequestAuthorizationContext>>> mappings;
```
遍历mappings，通过每个RequestMatcherEntry对象中的RequestMatcher对请求路径进行匹配，从而知道对该请求进行权限检查的 AuthorizationManager 对象是每个RequestMatcherEntry对象中 AuthorizationManager对象。

由此可知，自定义 RequestMatcherDelegatingAuthorizationManager 的功能其实就是向该mappings列表中添加元素。而mappings列表中的每个RequestMatcherEntry对象都由两个组件构成：
- RequestMatcher
- AuthorizationManager


## 自定义 RequestMatcher

从前面可知，
```java
/* -------------------------- AbstractRequestMatcherRegistry --------------------------
 * Description: 该类是AuthorizationManagerRequestMatcherRegistry的父类
 */
public C requestMatchers(HttpMethod method) {  
    return requestMatchers(method, "/**");  
}
public C requestMatchers(String... patterns) {  
    return requestMatchers(null, patterns);  
}
public C requestMatchers(HttpMethod method, String... patterns) {  
	// mvcPresent: SpringMVC中的HandlerMappingIntrospector类是否存在
    if (!mvcPresent) {  // 不存在
        return requestMatchers(RequestMatchers.antMatchersAsArray(method, patterns));  
    }  

    if (!(this.context instanceof WebApplicationContext)) {  
        return requestMatchers(RequestMatchers.antMatchersAsArray(method, patterns));  
    }  

    WebApplicationContext context = (WebApplicationContext) this.context;  
    ServletContext servletContext = context.getServletContext();  
    if (servletContext == null) {  
        return requestMatchers(RequestMatchers.antMatchersAsArray(method, patterns));  
    }  

    Map<String, ? extends ServletRegistration> registrations = mappableServletRegistrations(servletContext);  
    if (registrations.isEmpty()) {  
        return requestMatchers(RequestMatchers.antMatchersAsArray(method, patterns));  
    }  
    if (!hasDispatcherServlet(registrations)) {  
        return requestMatchers(RequestMatchers.antMatchersAsArray(method, patterns));  
    }  
    if (registrations.size() > 1) {  
        String errorMessage = computeErrorMessage(registrations.values());  
        throw new IllegalArgumentException(errorMessage);  
    }  
    return requestMatchers(createMvcMatchers(method, patterns).toArray(new RequestMatcher[0]));  
}

public C requestMatchers(RequestMatcher... requestMatchers) {  
    Assert.state(!this.anyRequestConfigured, "Can't configure requestMatchers after anyRequest");  
    return chainRequestMatchers(Arrays.asList(requestMatchers));  
}

protected AuthorizedUrl chainRequestMatchers(List<RequestMatcher> requestMatchers) {  
    this.unmappedMatchers = requestMatchers;  
    return new AuthorizedUrl(requestMatchers);  
}
```



### RequestMatchers.antMatchersAsArray


```java
/* ----------------------------- RequestMatchers ----------------------------- */
static RequestMatcher[] antMatchersAsArray(HttpMethod httpMethod, String... antPatterns) {  
    String method = (httpMethod != null) ? httpMethod.toString() : null;  
    RequestMatcher[] matchers = new RequestMatcher[antPatterns.length];  
    for (int index = 0; index < antPatterns.length; index++) {  
        matchers[index] = new AntPathRequestMatcher(antPatterns[index], method);  
    }  
    return matchers;  
}
```
### createMvcMatchers()
```java
protected final List<MvcRequestMatcher> createMvcMatchers(HttpMethod method, String... mvcPatterns) {  
   Assert.state(!this.anyRequestConfigured, "Can't configure mvcMatchers after anyRequest");  
   ObjectPostProcessor<Object> opp = this.context.getBean(ObjectPostProcessor.class);  
   if (!this.context.containsBean(HANDLER_MAPPING_INTROSPECTOR_BEAN_NAME)) {  
      throw new NoSuchBeanDefinitionException("A Bean named " + HANDLER_MAPPING_INTROSPECTOR_BEAN_NAME  
            + " of type " + HandlerMappingIntrospector.class.getName()  
            + " is required to use MvcRequestMatcher. Please ensure Spring Security & Spring MVC are configured in a shared ApplicationContext.");  
   }  
   HandlerMappingIntrospector introspector = this.context.getBean(HANDLER_MAPPING_INTROSPECTOR_BEAN_NAME,  
         HandlerMappingIntrospector.class);  
   List<MvcRequestMatcher> matchers = new ArrayList<>(mvcPatterns.length);  
   for (String mvcPattern : mvcPatterns) {  
      MvcRequestMatcher matcher = new MvcRequestMatcher(introspector, mvcPattern);  
      opp.postProcess(matcher);  
      if (method != null) {  
         matcher.setMethod(method);  
      }  
      matchers.add(matcher);  
   }  
   return matchers;  
}
```

```java
public C anyRequest() {  
    Assert.state(!this.anyRequestConfigured, "Can't configure anyRequest after itself");  
    C configurer = requestMatchers(ANY_REQUEST);  
    this.anyRequestConfigured = true;  
    return configurer;  
}
```


## 自定义权限

AuthorizedUrl提供下面的方法来自定义AuthenticatedAuthorizationManager的身份验证策略AbstractAuthorizationStrategy。

1）
```java
/* --------------------------------- AuthorizedUrl --------------------------------- */
public AuthorizationManagerRequestMatcherRegistry anonymous() {  
    return access(AuthenticatedAuthorizationManager.anonymous());  
}
/* -------------------------- AuthenticatedAuthorizationManager -------------------------- */
public static <T> AuthenticatedAuthorizationManager<T> anonymous() {  
    return new AuthenticatedAuthorizationManager<>(new AnonymousAuthorizationStrategy());  
}
```
2）
```java
/* --------------------------------- AuthorizedUrl --------------------------------- */
public AuthorizationManagerRequestMatcherRegistry rememberMe() {  
    return access(AuthenticatedAuthorizationManager.rememberMe());  
}
/* -------------------------- AuthenticatedAuthorizationManager -------------------------- */
public static <T> AuthenticatedAuthorizationManager<T> rememberMe() {  
   return new AuthenticatedAuthorizationManager<>(new RememberMeAuthorizationStrategy());  
}
```
3）
```java
/* --------------------------------- AuthorizedUrl --------------------------------- */
public AuthorizationManagerRequestMatcherRegistry authenticated() {  
    return access(AuthenticatedAuthorizationManager.authenticated());  
}
/* -------------------------- AuthenticatedAuthorizationManager -------------------------- */
public static <T> AuthenticatedAuthorizationManager<T> authenticated() {  
    return new AuthenticatedAuthorizationManager<>();  
}
```
4）
```java
/* --------------------------------- AuthorizedUrl --------------------------------- */
public AuthorizationManagerRequestMatcherRegistry fullyAuthenticated() {  
    return access(AuthenticatedAuthorizationManager.fullyAuthenticated());  
}
/* -------------------------- AuthenticatedAuthorizationManager -------------------------- */
public static <T> AuthenticatedAuthorizationManager<T> fullyAuthenticated() {  
    return new AuthenticatedAuthorizationManager<>(new FullyAuthenticatedAuthorizationStrategy());  
}
```

5）
```java
/* --------------------------------- AuthorizedUrl --------------------------------- */
public AuthorizationManagerRequestMatcherRegistry permitAll() {  
	// permitAllAuthorizationManager:
	// (a, o) -> new AuthorizationDecision(true)
    return access(permitAllAuthorizationManager);  
}
```
6）
```java
/* --------------------------------- AuthorizedUrl --------------------------------- */
public AuthorizationManagerRequestMatcherRegistry denyAll() {  
    return access((a, o) -> new AuthorizationDecision(false));  
}
```

## 自定义角色权限

1）
```java
public AuthorizationManagerRequestMatcherRegistry hasRole(String role) {  
    return access(withRoleHierarchy(AuthorityAuthorizationManager.hasRole(role)));  
}
public static <T> AuthorityAuthorizationManager<T> hasRole(String role) {  
    ... // Assert
    return hasAuthority(ROLE_PREFIX + role);  
}
public static <T> AuthorityAuthorizationManager<T> hasAuthority(String authority) {  
    ... // Assert
    return new AuthorityAuthorizationManager<>(authority);  
}
```
2）
```java
public AuthorizationManagerRequestMatcherRegistry hasAnyRole(String... roles) {  
    return access(withRoleHierarchy(AuthorityAuthorizationManager.hasAnyRole(roles)));  
}
public static <T> AuthorityAuthorizationManager<T> hasAnyRole(String... roles) {  
    return hasAnyRole(ROLE_PREFIX, roles);  
}
public static <T> AuthorityAuthorizationManager<T> hasAnyRole(String rolePrefix, String[] roles) {  
    ... // Assert

    // toNamedRolesArray(): 简单地将roles中的元素逐一加上前缀rolePrefix
    return hasAnyAuthority(toNamedRolesArray(rolePrefix, roles));  
}
public static <T> AuthorityAuthorizationManager<T> hasAnyAuthority(String... authorities) {  
    ... // Assert
    return new AuthorityAuthorizationManager<>(authorities);  
}
```
3）
```java
public AuthorizationManagerRequestMatcherRegistry hasAuthority(String authority) {  
    return access(withRoleHierarchy(AuthorityAuthorizationManager.hasAuthority(authority)));  
}
public static <T> AuthorityAuthorizationManager<T> hasAuthority(String authority) {  
    ... // Assert  
    return new AuthorityAuthorizationManager<>(authority);  
}
```
4）
```java
public AuthorizationManagerRequestMatcherRegistry hasAnyAuthority(String... authorities) {  
    return access(withRoleHierarchy(AuthorityAuthorizationManager.hasAnyAuthority(authorities)));  
}
public static <T> AuthorityAuthorizationManager<T> hasAnyAuthority(String... authorities) {  
	... // Assert
    return new AuthorityAuthorizationManager<>(authorities);  
}
```



## access()

```java
/* ------------------------------ AuthorizedUrl ------------------------------ */
public AuthorizationManagerRequestMatcherRegistry access(AuthorizationManager<RequestAuthorizationContext> manager) {  
    ... // Assert.notNull(manager, ...);  

    return AuthorizeHttpRequestsConfigurer.this.addMapping(this.matchers, manager);  
}

private AuthorizationManagerRequestMatcherRegistry addMapping(List<? extends RequestMatcher> matchers,    AuthorizationManager<RequestAuthorizationContext> manager) {  
    for (RequestMatcher matcher : matchers) {  
        this.registry.addMapping(matcher, manager);  
    }  
    // 
    return this.registry;  
}
```
