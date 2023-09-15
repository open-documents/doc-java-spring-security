
先来看该过滤器的定义。
```java
public abstract class OncePerRequestFilter extends GenericFilterBean {}
```

# 该Filter的具体逻辑

```java
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {  
    filterChain.doFilter(request, new DisableEncodeUrlResponseWrapper(response));  
}
```

