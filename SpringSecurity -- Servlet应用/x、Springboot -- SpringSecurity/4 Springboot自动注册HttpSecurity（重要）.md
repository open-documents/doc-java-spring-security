
先说结论，默认情况下向Spring容器中注册的FilterChainProxy Bean中的SecurityFilterChain包含如下的Filter（按顺序）：

| Filter                                  | HttpSecurity方法 |
| --------------------------------------- | ---------------- |
| DisableEncodeUrlFilter                  |                  |
| WebAsyncManagerIntegrationFilter        |                  |
| SecurityContextHolderFilter             |                  |
| HeaderWriterFilter                      |                  |
| CsrfFilter                              |                  |
| LogoutFilter                            |                  |
| UsernamePasswordAuthenticationFilter    |                  |
| DefaultLoginPageGeneratingFilter        |                  |
| DefaultLogoutPageGeneratingFilter       |                  |
| BasicAuthenticationFilter               |                  |
| RequestCacheAwareFilter                 |                  |
| SecurityContextHodlerAwareRequestFilter |                  |
| AnonymousAuthenticationFilter           |                  |
| ExceptionTranslationFilter              |                  |
| AuthorizationFilter                     |                  |
|                                         |                  |

# 一、Springboot主动注册HttpSecurity

先看自动注册到Spring容器中的HttpSecurity Bean。
```java
@Configuration(proxyBeanMethods = false)  
class HttpSecurityConfiguration {
	// HTTPSECURITY_BEAN_NAME = org.springframework.security.config.annotation.web.configuration.HttpSecurityConfiguration.httpSecurity
	@Bean(HTTPSECURITY_BEAN_NAME)  
	@Scope("prototype")  
	HttpSecurity httpSecurity() throws Exception {  
	
	    LazyPasswordEncoder passwordEncoder = new LazyPasswordEncoder(this.context);  

		// this.objectPostProcessor: , 支持set注入
		// - 类型 ObjectPostProcessor<Object>
		// - 自动装配 @Autowired 注入属性
	    AuthenticationManagerBuilder authenticationBuilder = new DefaultPasswordEncoderAuthenticationManagerBuilder(  
	         this.objectPostProcessor, 
	         passwordEncoder
	    );  
	    authenticationBuilder.parentAuthenticationManager(authenticationManager());  
	    authenticationBuilder.authenticationEventPublisher(getAuthenticationEventPublisher());  

	    HttpSecurity http = new HttpSecurity(this.objectPostProcessor, authenticationBuilder, createSharedObjects());
	    WebAsyncManagerIntegrationFilter webAsyncManagerIntegrationFilter = new WebAsyncManagerIntegrationFilter();  
	    webAsyncManagerIntegrationFilter.setSecurityContextHolderStrategy(this.securityContextHolderStrategy);  
	    // import static org.springframework.security.config.Customizer.withDefaults;
	    http  
	      .csrf(withDefaults())                                 
	      .addFilter(webAsyncManagerIntegrationFilter)          
	      .exceptionHandling(withDefaults())                   
	      .headers(withDefaults())                            
	      .sessionManagement(withDefaults())                
	      .securityContext(withDefaults())                    
	      .requestCache(withDefaults())                       
	      .anonymous(withDefaults())                       
	      .servletApi(withDefaults())                      
	      .apply(new DefaultLoginPageConfigurer<>());  
	    http.logout(withDefaults());  

	    applyDefaultConfigurers(http);  
	    return http;  
	}
}
```

```java
private void applyDefaultConfigurers(HttpSecurity http) throws Exception {  
    ClassLoader classLoader = this.context.getClassLoader();  
    // springframe特性: 从META/INF/spring.factories中加载指定全限定名对应的类
    // 此处默认0个
    List<AbstractHttpConfigurer> defaultHttpConfigurers = 
	    SpringFactoriesLoader.loadFactories(AbstractHttpConfigurer.class, classLoader);  
    for (AbstractHttpConfigurer configurer : defaultHttpConfigurers) {  
        http.apply(configurer);
    }  
}
```


