
```java
protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response,  
      AuthenticationException failed) throws IOException, ServletException {  
    this.securityContextHolderStrategy.clearContext();  
    
    ... // logger.trace()  
    ... // logger.trace()  
    ... // logger.trace()  
     
    this.rememberMeServices.loginFail(request, response);  

    this.failureHandler.onAuthenticationFailure(request, response, failed);  
}
```