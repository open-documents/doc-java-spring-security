
```java
/* ---------------------------------- LogoutFilter ---------------------------------- */
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)  
      throws IOException, ServletException {  
    doFilter((HttpServletRequest) request, (HttpServletResponse) response, chain);  
}
private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)     throws IOException, ServletException {  
    if (requiresLogout(request, response)) {  
    
        Authentication auth = this.securityContextHolderStrategy.getContext().getAuthentication();  

		... // logger.debug()  

		// 做登出处理
		// LogoutFilter.this.handler:
		// 类型 LogoutHandler
		// 默认初值 null, 但一定会在构造函数中赋值, 并且类型为 CompositeLogoutHandler
        this.handler.logout(request, response, auth);

		// 登出成功, 做进一步处理
		// LogoutFilter.this.logoutSuccessHandler:
		// - 类型 LogoutSuccessHandler
		// - 默认初值 null, 但在一定会在构造函数中赋值, 当提供String类型URL而非直接的LogoutSuccessHandler实例时, 类型是SimpleUrlLogoutSuccessHandler
		// - 仅支持构造函数注入
        this.logoutSuccessHandler.onLogoutSuccess(request, response, auth);

        return; 
	}  

    chain.doFilter(request, response);  
}
```




# 是否需要登出

```java
/* ---------------------------------- LogoutFilter ---------------------------------- */
protected boolean requiresLogout(HttpServletRequest request, HttpServletResponse response) {  
	//   LogoutFilter.this.logoutRequestMatcher:
	// - 类型 RequestMatcher
	// - 默认初值 null, 但是在构造函数中会默认赋值 new AntPathRequestMatcher("/logout")
	// - 仅支持set注入
    if (this.logoutRequestMatcher.matches(request)) {  
        return true;  
    }  

	... // logger.trace()  

    return false;  
}
```


# 组件

## RequestMatcher logoutRequestMatcher

功能：用于判断当前请求是否是登出请求。
```java
private RequestMatcher logoutRequestMatcher;
// 仅支持set注入 
public void setLogoutRequestMatcher(RequestMatcher logoutRequestMatcher) {  
    ... // Assert.notNull(logoutRequestMatcher, ...);  
    this.logoutRequestMatcher = logoutRequestMatcher;  
}
```
初值：一定会在构造函数中赋初值。
```java
public LogoutFilter(LogoutSuccessHandler logoutSuccessHandler, LogoutHandler... handlers) {  
    ... 
    setFilterProcessesUrl("/logout");  
}
public LogoutFilter(String logoutSuccessUrl, LogoutHandler... handlers) {  
    ...
    setFilterProcessesUrl("/logout");  
}
```

## LogoutHandler

功能：在登出前做处理，处理完成即为登出成功。
```java
private final LogoutHandler handler;
// 仅支持构造函数注入, 并且类型一定是 CompositeLogoutHandler
public LogoutFilter(LogoutSuccessHandler logoutSuccessHandler, LogoutHandler... handlers) {  
	this.handler = new CompositeLogoutHandler(handlers);
    ... 
}
public LogoutFilter(String logoutSuccessUrl, LogoutHandler... handlers) {  
	this.handler = new CompositeLogoutHandler(handlers);
    ...
}
```

## LogoutSuccessHandler

功能：登出成功后做处理。
```java
private final LogoutSuccessHandler logoutSuccessHandler;
// 仅支持构造函数注入
public LogoutFilter(LogoutSuccessHandler logoutSuccessHandler, LogoutHandler... handlers) {  
    ...
    ... // Assert
    this.logoutSuccessHandler = logoutSuccessHandler;  
    ... 
}
public LogoutFilter(String logoutSuccessUrl, LogoutHandler... handlers) {  
	...
    ... // Assert
    SimpleUrlLogoutSuccessHandler urlLogoutSuccessHandler = new SimpleUrlLogoutSuccessHandler();  
    if (StringUtils.hasText(logoutSuccessUrl)) {  
        urlLogoutSuccessHandler.setDefaultTargetUrl(logoutSuccessUrl);  
    }  
    this.logoutSuccessHandler = urlLogoutSuccessHandler;  
    ... 
}
```