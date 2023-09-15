
# 具体过滤逻辑

```java
/* ------------------------------------------ CsrfFilter ------------------------------------------ */
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {  
	// 
    DeferredCsrfToken deferredCsrfToken = this.tokenRepository.loadDeferredToken(request, response);  

	
    request.setAttribute(DeferredCsrfToken.class.getName(), deferredCsrfToken);  

	// CsrfFilter.
    this.requestHandler.handle(request, response, deferredCsrfToken::get);  
    if (!this.requireCsrfProtectionMatcher.matches(request)) {  
        ... // logger.log()
        filterChain.doFilter(request, response);  
        return;  
	}  
    CsrfToken csrfToken = deferredCsrfToken.get();  
    String actualToken = this.requestHandler.resolveCsrfTokenValue(request, csrfToken);  
    if (!equalsConstantTime(csrfToken.getToken(), actualToken)) {
	    //   
        boolean missingToken = deferredCsrfToken.isGenerated();  
        ... // logger.log() 
        this.accessDeniedHandler.handle(request, response, exception);  
        return; 
	}  
    filterChain.doFilter(request, response);  
}
```

## 