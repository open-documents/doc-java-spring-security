



```java
/* ------------------------------ AuthorizationFilter ------------------------------ */
public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain chain)  
      throws ServletException, IOException {  
  
    HttpServletRequest request = (HttpServletRequest) servletRequest;  
    HttpServletResponse response = (HttpServletResponse) servletResponse;  

	// this.observeOncePerRequest: 默认初值 false, 支持set注入
    if (this.observeOncePerRequest && isApplied(request)) {  
        chain.doFilter(request, response);  
        return; 
	}  
  
    if (skipDispatch(request)) {  
        chain.doFilter(request, response);  
        return; 
	}  
  
    String alreadyFilteredAttributeName = getAlreadyFilteredAttributeName();  
    request.setAttribute(alreadyFilteredAttributeName, Boolean.TRUE);  
    try {
	    // AuthorizationFilter.this.authorizationManager:
	    // - 默认注入的是 RequestMatcherDelegatingAuthorizationManager
	    // - 仅支持构造函数注入
        AuthorizationDecision decision = this.authorizationManager.check(this::getAuthentication, request);  
        this.eventPublisher.publishAuthorizationEvent(this::getAuthentication, request, decision);  
        if (decision != null && !decision.isGranted()) {
            throw new AccessDeniedException("Access Denied");  
        }  
        chain.doFilter(request, response);  
    }  
    finally {  
        request.removeAttribute(alreadyFilteredAttributeName);  
    }  
}
```

# 传递Authentication

```java
/* ------------------------------ AuthorizationFilter ------------------------------ */
private Authentication getAuthentication() {  

	// AuthorizationFilter.this.securityContextHolderStrategy:
	// - 类型     SecurityContextHolderStrategy
	// - 默认初值 SecurityContextHolder.getContextHolderStrategy()
    Authentication authentication = this.securityContextHolderStrategy.getContext().getAuthentication();  
    if (authentication == null) {  
        throw new AuthenticationCredentialsNotFoundException(...);  
    }  
    return authentication;  
}
```
