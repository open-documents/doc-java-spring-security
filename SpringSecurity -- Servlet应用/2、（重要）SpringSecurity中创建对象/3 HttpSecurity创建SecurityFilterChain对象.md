
# HttpSecurity

下面是HttpSecurity中的方法创建的SecurityConfigurer以及每个SecurityConfigurer创建的Filter。

| HttpSecurity方法         | SecurityConfigurer              | Filter                                                                       |
| ------------------------ | ------------------------------- | ---------------------------------------------------------------------------- |
| .csrf()                  | CsrfConfigurer                  | CsrfFilter                                                                   |
| .exceptionHandling()     | ExceptionHandlingConfigurer     | ExceptionTranslationFilter                                                   |
| .headers()               | HeadersConfigurer               | HeaderWriterFilter                                                           |
| .sessionManagement()     | SessionManagementConfigurer     | DisableEncoderUrlFilter                                                      |
| .securityContext()       | SecurityContextConfigurer       | SecurityContextHolderFilter                                                  |
| .requestCache()          | RequestCacheConfigurer          | RequestCacheAwareFilter                                                      |
| .anonymous()             | AnonymousConfigurer             | AnonymousAuthenticationFilter                                                |
| .servletApi()            | ServletApiConfigurer            | SecurityContextHolderAwareRequestFilter                                      |
|                          | DefaultLoginPageConfigurer      | - DefaultLoginPageGeneratingFilter</br>- DefaultLoginoutPageGeneratingFilter |
| .logout()                | LogoutConfigurer                | LogoutFilter                                                                 |
| .authorizeHttpRequests() | AuthorizeHttpRequestsConfigurer | AuthorizationFilter                                                          |
| .formLogin()             | formLoginConfigurer             | UsernamePasswordAuthenticationFilter                                         |
| .httpBasic()             | HttpBasicConfigurer             | BasicAuthenticationFilter                                                    |


```java
httpSecurity.authorizeHttpRequests((authorize) -> authorize.anyRequest().authenticated()); 

/* ------------------------------------ HttpSecurity ------------------------------------ */
public HttpSecurity authorizeHttpRequests(  
      Customizer<AuthorizeHttpRequestsConfigurer<HttpSecurity>.AuthorizationManagerRequestMatcherRegistry> authorizeHttpRequestsCustomizer) throws Exception {  
    ApplicationContext context = getContext();  
    authorizeHttpRequestsCustomizer  
	    .customize(getOrApply(new AuthorizeHttpRequestsConfigurer<>(context)).getRegistry());  
    return HttpSecurity.this;  
}
```



# 创建



```java
/* ---------------------------------- HttpSecurity ---------------------------------- */
 
protected DefaultSecurityFilterChain performBuild() {  
    ExpressionUrlAuthorizationConfigurer<?> expressionConfigurer = getConfigurer(  
         ExpressionUrlAuthorizationConfigurer.class);  
    AuthorizeHttpRequestsConfigurer<?> httpConfigurer = getConfigurer(AuthorizeHttpRequestsConfigurer.class);  
    
    boolean oneConfigurerPresent = expressionConfigurer == null ^ httpConfigurer == null;  
    Assert.state(
	    (expressionConfigurer == null && httpConfigurer == null) || oneConfigurerPresent,  
		"authorizeHttpRequests cannot be used in conjunction with authorizeRequests. Please select just one.");  

	// 
    this.filters.sort(OrderComparator.INSTANCE);  
    List<Filter> sortedFilters = new ArrayList<>(this.filters.size());  
    for (Filter filter : this.filters) {  
        sortedFilters.add(((OrderedFilter) filter).filter);  
    }  
    return new DefaultSecurityFilterChain(this.requestMatcher, sortedFilters);  
}
```

```java
public <C extends SecurityConfigurer<O, B>> C getConfigurer(Class<C> clazz) {  
   List<SecurityConfigurer<O, B>> configs = this.configurers.get(clazz);  
   if (configs == null) {  
      return null;  
   }  
   Assert.state(configs.size() == 1,  
         () -> "Only one configurer expected for type " + clazz + ", but got " + configs);  
   return (C) configs.get(0);  
}
```