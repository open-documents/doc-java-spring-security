

```java
/* -------------------------------- HttpSecurityConfiguration --------------------------------
 * withDefaults(): 来自 import static org.springframework.security.config.Customizer.withDefaults;
 */
http.csrf(withDefaults())

public HttpSecurity csrf(Customizer<CsrfConfigurer<HttpSecurity>> csrfCustomizer) throws Exception {  
    ApplicationContext context = getContext();  
    // csrfCustomizer.customize: 自定义 CsrfConfigurer<HttpSecurity>
	// 因为withDefaults()的原因, 此情况下不对CsrfConfigurer<HttpSecurity>做任何处理
    csrfCustomizer.customize(getOrApply(new CsrfConfigurer<>(context)));  
    return HttpSecurity.this;  
}
```



```java
/* --------------------------------- CsrfConfigurer --------------------------------- 
 * 
 */
public void configure(H http) {  

	// this.csrfTokenRepository: 
	// - 类型 CsrfTokenRepository
	// - 默认初值 new LazyCsrfTokenRepository(new HttpSessionCsrfTokenRepository())
    CsrfFilter filter = new CsrfFilter(this.csrfTokenRepository);  

    RequestMatcher requireCsrfProtectionMatcher = getRequireCsrfProtectionMatcher();  
    if (requireCsrfProtectionMatcher != null) {  
       filter.setRequireCsrfProtectionMatcher(requireCsrfProtectionMatcher);  
    }

    AccessDeniedHandler accessDeniedHandler = createAccessDeniedHandler(http);  
    ObservationRegistry registry = getObservationRegistry();  
    if (!registry.isNoop()) {  
        ObservationMarkingAccessDeniedHandler observable = new ObservationMarkingAccessDeniedHandler(registry);  
        accessDeniedHandler = new CompositeAccessDeniedHandler(observable, accessDeniedHandler);  
    }  
    if (accessDeniedHandler != null) {  
	    filter.setAccessDeniedHandler(accessDeniedHandler);  
    }  

    LogoutConfigurer<H> logoutConfigurer = http.getConfigurer(LogoutConfigurer.class);  
    if (logoutConfigurer != null) {  
	    logoutConfigurer.addLogoutHandler(new CsrfLogoutHandler(this.csrfTokenRepository));  
    }  
    SessionManagementConfigurer<H> sessionConfigurer = http.getConfigurer(SessionManagementConfigurer.class);  
    if (sessionConfigurer != null) {  
        sessionConfigurer.addSessionAuthenticationStrategy(getSessionAuthenticationStrategy());  
    }  

    if (this.requestHandler != null) {  
        filter.setRequestHandler(this.requestHandler);  
    }  
    filter = postProcess(filter);  
    http.addFilter(filter);  
}
```

```java
/* --------------------------------- SecurityConfigurerAdapter --------------------------------- 
 * 
 */
protected <T> T postProcess(T object) {  
    return (T) this.objectPostProcessor.postProcess(object);  
}
```