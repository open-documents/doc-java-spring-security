
```java
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {  
    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request); 

    SecurityContextCallableProcessingInterceptor securityProcessingInterceptor = (SecurityContextCallableProcessingInterceptor) asyncManager  
	    .getCallableInterceptor(CALLABLE_INTERCEPTOR_KEY);  

    if (securityProcessingInterceptor == null) {  
      SecurityContextCallableProcessingInterceptor interceptor = new SecurityContextCallableProcessingInterceptor();  
      interceptor.setSecurityContextHolderStrategy(this.securityContextHolderStrategy);  
      asyncManager.registerCallableInterceptor(CALLABLE_INTERCEPTOR_KEY, interceptor);  
    }  

    filterChain.doFilter(request, response);  
}
```