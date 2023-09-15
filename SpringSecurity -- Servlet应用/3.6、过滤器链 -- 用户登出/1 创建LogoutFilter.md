
```java
public void configure(H http) throws Exception {  
    LogoutFilter logoutFilter = createLogoutFilter(http);  
    http.addFilter(logoutFilter);  
}

private LogoutFilter createLogoutFilter(H http) {  

    this.contextLogoutHandler.setSecurityContextHolderStrategy(getSecurityContextHolderStrategy());  
    this.contextLogoutHandler.setSecurityContextRepository(getSecurityContextRepository(http));  


    this.logoutHandlers.add(this.contextLogoutHandler);  
    this.logoutHandlers.add(postProcess(new LogoutSuccessEventPublishingLogoutHandler()));  
    LogoutHandler[] handlers = this.logoutHandlers.toArray(new LogoutHandler[0]);  
    LogoutFilter result = new LogoutFilter(getLogoutSuccessHandler(), handlers);  
    result.setSecurityContextHolderStrategy(getSecurityContextHolderStrategy());  
    result.setLogoutRequestMatcher(getLogoutRequestMatcher(http));  
    result.setSecurityContextHolderStrategy(getSecurityContextHolderStrategy());  
    result = postProcess(result);  
    return result;  
}
```