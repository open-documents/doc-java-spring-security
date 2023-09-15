
UsernamePasswordAuthenticationFilter是由FormLoginConfigurer创建。


HttpSecurity


# 向HttpSecurity中添加Filter

```java
/* ------------------------- AbstractAuthenticationFilterConfigurer -------------------------
 * 
 */
public void configure(B http) throws Exception {  

    PortMapper portMapper = http.getSharedObject(PortMapper.class);  
    if (portMapper != null) {  
        this.authenticationEntryPoint.setPortMapper(portMapper);  
    }

    RequestCache requestCache = http.getSharedObject(RequestCache.class);  
    if (requestCache != null) {  
        this.defaultSuccessHandler.setRequestCache(requestCache);  
    }

	// 
	// AbstractAuthenticationFilterConfigurer.this.authFilter: 
	// 类型 F, F extends AbstractAuthenticationProcessingFilter
    this.authFilter.setAuthenticationManager(http.getSharedObject(AuthenticationManager.class)); 

	// 
    this.authFilter.setAuthenticationSuccessHandler(this.successHandler);

	// 
    this.authFilter.setAuthenticationFailureHandler(this.failureHandler);  
    if (this.authenticationDetailsSource != null) {
        this.authFilter.setAuthenticationDetailsSource(this.authenticationDetailsSource);  
    }  

    SessionAuthenticationStrategy sessionAuthenticationStrategy = http  
          .getSharedObject(SessionAuthenticationStrategy.class);  
    if (sessionAuthenticationStrategy != null) {  
        this.authFilter.setSessionAuthenticationStrategy(sessionAuthenticationStrategy);  
    }  

    RememberMeServices rememberMeServices = http.getSharedObject(RememberMeServices.class);  
    if (rememberMeServices != null) {  
        this.authFilter.setRememberMeServices(rememberMeServices);  
    }  

    SecurityContextConfigurer securityContextConfigurer = http.getConfigurer(SecurityContextConfigurer.class);  
    if (securityContextConfigurer != null && securityContextConfigurer.isRequireExplicitSave()) {  
        SecurityContextRepository securityContextRepository = securityContextConfigurer  
             .getSecurityContextRepository();  
        this.authFilter.setSecurityContextRepository(securityContextRepository);  
    }  
    this.authFilter.setSecurityContextHolderStrategy(getSecurityContextHolderStrategy());

    F filter = postProcess(this.authFilter);  


    http.addFilter(filter);  
}
```


通过configure()过程知道，向HttpSecurity中


