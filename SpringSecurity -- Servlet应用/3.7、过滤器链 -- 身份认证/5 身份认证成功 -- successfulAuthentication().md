


```java
/* --------------------------- AbstractAuthenticationProcessingFilter --------------------------- */
protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authResult) throws IOException, ServletException {  
	//   AbstractAuthenticationProcessingFilter.this.securityContextHolderStrategy:
	// - 静态(static)类型 SecurityContextHolderStrategy, 用于创建 SecurityContext对象
	// - 默认初值 SecurityContextHolder.getContextHolderStrategy()
	// - 仅支持set注入
    SecurityContext context = this.securityContextHolderStrategy.createEmptyContext();  
    context.setAuthentication(authResult);  
    this.securityContextHolderStrategy.setContext(context);  
    //   AbstractAuthenticationProcessingFilter.this.securityContextRepository:
    // - 静态(static)类型 SecurityContextRepository 
    // - 默认初值 new RequestAttributeSecurityContextRepository()
	// - 仅支持set注入
    this.securityContextRepository.saveContext(context, request, response);  

	... // logger.debug()

    this.rememberMeServices.loginSuccess(request, response, authResult);  

    if (this.eventPublisher != null) {  
        this.eventPublisher.publishEvent(
	        new InteractiveAuthenticationSuccessEvent(authResult, this.getClass())
	    );  
    }  

    this.successHandler.onAuthenticationSuccess(request, response, authResult);  
}
```