
```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)  
      throws IOException, ServletException {  
    doFilter((HttpServletRequest) request, (HttpServletResponse) response, chain);  
}
private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)      throws IOException, ServletException {  
    if (this.securityContextHolderStrategy.getContext().getAuthentication() != null) {  
	    // logger.debug()
        chain.doFilter(request, response);  
        return; 
	}  

    Authentication rememberMeAuth = this.rememberMeServices.autoLogin(request, response);  

    if (rememberMeAuth != null) {  
        // Attempt authentication via AuthenticationManager  
        try {  
            rememberMeAuth = this.authenticationManager.authenticate(rememberMeAuth);  

            // Store to SecurityContextHolder  
            SecurityContext context = this.securityContextHolderStrategy.createEmptyContext();  
            context.setAuthentication(rememberMeAuth);  
            this.securityContextHolderStrategy.setContext(context);  

            onSuccessfulAuthentication(request, response, rememberMeAuth);  

            ... // logger.debug()  

            this.securityContextRepository.saveContext(context, request, response);  

            if (this.eventPublisher != null) {  
	            this.eventPublisher.publishEvent(
		            new InteractiveAuthenticationSuccessEvent(  
	                    this.securityContextHolderStrategy.getContext().getAuthentication(),
	                    this.getClass()
	                )
	            );  
            }  

            if (this.successHandler != null) {  
	            this.successHandler.onAuthenticationSuccess(request, response, rememberMeAuth);  
	            return; 
			}  
        }  
        catch (AuthenticationException ex) {  
            ... // logger.debug()  
            this.rememberMeServices.loginFail(request, response);  
            onUnsuccessfulAuthentication(request, response, ex);  
        }  
    }  
   chain.doFilter(request, response);  
}
```