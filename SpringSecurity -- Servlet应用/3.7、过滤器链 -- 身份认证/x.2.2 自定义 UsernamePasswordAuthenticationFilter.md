
# 使用另外的 AuthenticationManager

由前面的分析可知
```java
public HttpSecurity authenticationManager(AuthenticationManager authenticationManager) {  
    ... // Assert.notNull(authenticationManager, ...);  
    this.authenticationManager = authenticationManager;  
    return HttpSecurity.this;  
}
```
那么这个修改是如何生效的呢？
```java
protected void beforeConfigure() throws Exception {  
    if (this.authenticationManager != null) {  
        setSharedObject(AuthenticationManager.class, this.authenticationManager);  
    }  
    else {  
        ObservationRegistry registry = getObservationRegistry();  
        AuthenticationManager manager = getAuthenticationRegistry().build();  
        if (!registry.isNoop() && manager != null) {  
            setSharedObject(AuthenticationManager.class, new ObservationAuthenticationManager(registry, manager));  
        }
        else {  
            setSharedObject(AuthenticationManager.class, manager);  
        }  
    }  
}
```

# 自定义 AuthenticationManager








# 自定义添加的Filter

常用的自定义方法如下。
```java
public SecurityFilterChain filterChain(HttpSecurity http) {
	http.formLogin(
		form ->                         
		form.loginPage("/login")     // 登录页面设置
			.loginProcessingUrl()    // 登录访问路径
			.defaultSuccessUrl()      
			.permitAll()
	);
}
`````
## 自定义 -- 尚未认证

如果请求没有经过认证，需要将请求重定向到指定页面。自定义该指定页面的方法如下。
```java
public T loginProcessingUrl(String loginProcessingUrl) {  
    this.loginProcessingUrl = loginProcessingUrl;  
	this.authFilter.setRequiresAuthenticationRequestMatcher(createLoginProcessingUrlMatcher(loginProcessingUrl)); 
    return getSelf();  
}
```


## 自定义 -- 认证成功

针对认证成功后的进一步处理有下面3个方法来改变设置。
```java
public final T defaultSuccessUrl(String defaultSuccessUrl) {  
    return defaultSuccessUrl(defaultSuccessUrl, false);  
}
public final T defaultSuccessUrl(String defaultSuccessUrl, boolean alwaysUse) {  
    SavedRequestAwareAuthenticationSuccessHandler handler = new SavedRequestAwareAuthenticationSuccessHandler();  
    handler.setDefaultTargetUrl(defaultSuccessUrl);  
    handler.setAlwaysUseDefaultTargetUrl(alwaysUse);  
    this.defaultSuccessHandler = handler;  
    return successHandler(handler);  
}
public final T successHandler(AuthenticationSuccessHandler successHandler) {  
    this.successHandler = successHandler;  
    return getSelf();  
}

public FormLoginConfigurer<H> successForwardUrl(String forwardUrl) {  
    successHandler(new ForwardAuthenticationSuccessHandler(forwardUrl));  
    return this;
}
```

## 自定义 -- 认证失败

针对认证失败后的进一步处理有下面2个方法来改变设置。
```java
public final T failureUrl(String authenticationFailureUrl) {  
    T result = failureHandler(new SimpleUrlAuthenticationFailureHandler(authenticationFailureUrl));  
    this.failureUrl = authenticationFailureUrl;  
    return result;  
}
public final T failureHandler(AuthenticationFailureHandler authenticationFailureHandler) {  
    this.failureUrl = null;  
    this.failureHandler = authenticationFailureHandler;  
    return getSelf();  
}

public FormLoginConfigurer<H> failureForwardUrl(String forwardUrl) {  
    failureHandler(new ForwardAuthenticationFailureHandler(forwardUrl));  
    return this;
}
```

## 自定义 -- Filter的SecurityContextRepository
```java
public T securityContextRepository(SecurityContextRepository securityContextRepository) {  
    this.authFilter.setSecurityContextRepository(securityContextRepository);  
    return getSelf();  
}
```


## 自定义 -- 表单参数被验证的名称

```java
public FormLoginConfigurer<H> usernameParameter(String usernameParameter) {  
    getAuthenticationFilter().setUsernameParameter(usernameParameter);  
    return this;
}
public FormLoginConfigurer<H> passwordParameter(String passwordParameter) {  
    getAuthenticationFilter().setPasswordParameter(passwordParameter);  
    return this;
}
```

## loginPage()

也就是在调用Configurer自身的方法改变自身的属性。
```java
/* ------------------------------- FormLoginConfigurer ------------------------------- */
public FormLoginConfigurer<H> loginPage(String loginPage) {  
    return super.loginPage(loginPage);  
}
/* -------------------------- AbstractAuthenticationFilterConfigurer -------------------------- */
protected T loginPage(String loginPage) {  
    setLoginPage(loginPage);  
    updateAuthenticationDefaults();  
    this.customLoginPage = true;  
    return getSelf();  
}
```
```java
private void setLoginPage(String loginPage) {  
    this.loginPage = loginPage;  
    this.authenticationEntryPoint = new LoginUrlAuthenticationEntryPoint(loginPage);  
}
```

```java
protected final void updateAuthenticationDefaults() {  
    if (this.loginProcessingUrl == null) {  
        loginProcessingUrl(this.loginPage);  
    }  
    if (this.failureHandler == null) {  
        failureUrl(this.loginPage + "?error");  
    }  
    LogoutConfigurer<B> logoutConfigurer = getBuilder().getConfigurer(LogoutConfigurer.class);  
    if (logoutConfigurer != null && !logoutConfigurer.isCustomLogoutSuccess()) {  
        logoutConfigurer.logoutSuccessUrl(this.loginPage + "?logout");  
    }
}

public T loginProcessingUrl(String loginProcessingUrl) {  
    this.loginProcessingUrl = loginProcessingUrl;  
	this.authFilter.setRequiresAuthenticationRequestMatcher(
		createLoginProcessingUrlMatcher(loginProcessingUrl));  
    return getSelf();  
}

```
```java
private T getSelf() {  
   return (T) this;  
}
```



