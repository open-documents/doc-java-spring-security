
```java
http.exceptionHandling()

```


# ExceptionHandlingConfigurer

```java
/* -------------------------------- ExceptionHandlingConfigurer -------------------------------- */
public void configure(H http) {  
    AuthenticationEntryPoint entryPoint = getAuthenticationEntryPoint(http);  
    ExceptionTranslationFilter exceptionTranslationFilter = new ExceptionTranslationFilter(entryPoint,    getRequestCache(http));  
    AccessDeniedHandler deniedHandler = getAccessDeniedHandler(http);  
    exceptionTranslationFilter.setAccessDeniedHandler(deniedHandler);  
    exceptionTranslationFilter.setSecurityContextHolderStrategy(getSecurityContextHolderStrategy());  
    exceptionTranslationFilter = postProcess(exceptionTranslationFilter);  
    http.addFilter(exceptionTranslationFilter);  
}
```
## 获取所需的组件 AuthenticationEntryPoint
```java
AuthenticationEntryPoint getAuthenticationEntryPoint(H http) {  
    AuthenticationEntryPoint entryPoint = this.authenticationEntryPoint;  
    if (entryPoint == null) {  
        entryPoint = createDefaultEntryPoint(http);  
    }  
    return entryPoint;  
}
```
## 获取所需的组件 RequestCache
```java
private RequestCache getRequestCache(H http) {  
    RequestCache result = http.getSharedObject(RequestCache.class);  
    if (result != null) {  
        return result;  
    }  
    return new HttpSessionRequestCache();  
}
```
## 获取所需的组件 AccessDeniedHandler
```java
AccessDeniedHandler getAccessDeniedHandler(H http) {  
    AccessDeniedHandler deniedHandler = this.accessDeniedHandler;  
    if (deniedHandler == null) {  
        deniedHandler = createDefaultDeniedHandler(http);  
    }  
    return deniedHandler;  
}
```

# 自定义 AuthenticationEntryPoint

