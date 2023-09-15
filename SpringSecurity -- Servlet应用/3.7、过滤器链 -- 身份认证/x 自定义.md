

```java
/* --------------------------- AuthenticationManagerBuilder --------------------------- */
public HttpSecurity userDetailsService(UserDetailsService userDetailsService) throws Exception {  
    getAuthenticationRegistry().userDetailsService(userDetailsService);  
    return this;
}
```
