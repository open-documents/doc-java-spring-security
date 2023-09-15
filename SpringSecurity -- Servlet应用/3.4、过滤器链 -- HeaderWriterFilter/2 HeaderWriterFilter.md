
先来看该过滤器的定义。
```java
public class HeaderWriterFilter extends OncePerRequestFilter{}
```

```java
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {  
	// HeaderWriterFilter.this.shouldWriteHeadersEagerly: 默认初值 false, 仅支持set注入
    if (this.shouldWriteHeadersEagerly) {  
        doHeadersBefore(request, response, filterChain);  
    }  
    else {  
        doHeadersAfter(request, response, filterChain);  
    }  
}
```
# 先写响应头，后执行过滤器链中的下一个过滤器
```java
/* ----------------------------------- HeaderWriterFilter ----------------------------------- */
private void doHeadersBefore(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws IOException, ServletException {  
   writeHeaders(request, response);  
   filterChain.doFilter(request, response);  
}
void writeHeaders(HttpServletRequest request, HttpServletResponse response) {  
	// 1. HeaderWriter: 根据源码定义, 向 HttpServletResponse 写响应头
	// 2. HeaderWriterFilter.this.headerWriters:
	//  - 类型 List<HeaderWriter>
	//  - 默认初值 null, 仅支持构造函数注入
    for (HeaderWriter writer : this.headerWriters) {  
        writer.writeHeaders(request, response);  
    }  
}
```
# 先执行过滤器链中的下一个过滤器，后写响应头
```java
private void doHeadersAfter(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws IOException, ServletException {  
    HeaderWriterResponse headerWriterResponse = new HeaderWriterResponse(request, response);  
    HeaderWriterRequest headerWriterRequest = new HeaderWriterRequest(request, headerWriterResponse);  
    try {  
        filterChain.doFilter(headerWriterRequest, headerWriterResponse);  
    }  
    finally {  
        headerWriterResponse.writeHeaders();  
    }  
}
```