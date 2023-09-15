
```java
public final void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain)  
      throws ServletException, IOException {  
  
    if (!( 
	    (request instanceof HttpServletRequest httpRequest) &&
	    (response instanceof HttpServletResponse httpResponse))) {  
        throw new ServletException("OncePerRequestFilter only supports HTTP requests");  
    }  

	// 拼接 filterName 和 ".FILTERED"
	// 1. filterName的来源, 从上到下:
	// - 过滤器名称: 从GenericFilterBean.this.filterConfig(FilterConfig).getFilterName()获取
	// - bean名称: 从GenericFilterBean.this.beanName获取
	// - 类名称: 从getClass().getName()获取
    String alreadyFilteredAttributeName = getAlreadyFilteredAttributeName();  
    boolean hasAlreadyFilteredAttribute = request.getAttribute(alreadyFilteredAttributeName) != null;  
  
    if (
	    skipDispatch(httpRequest) || 
	    // protected boolean方法, 默认一直返回 false
	    shouldNotFilter(httpRequest)) {  
        // 该过滤器不进行过滤, 直接往后进行
        filterChain.doFilter(request, response);  
    }  
    // 已经过滤过了
    else if (hasAlreadyFilteredAttribute) {  
        if (DispatcherType.ERROR.equals(request.getDispatcherType())) {  
	        // protected void方法, 默认直接 filterChain.doFilter(request, response)
            doFilterNestedErrorDispatch(httpRequest, httpResponse, filterChain);  
            return;  
		}  
  
        // Proceed without invoking this filter...  
        filterChain.doFilter(request, response);  
    }  

    else {  
        // Do invoke this filter...  
        request.setAttribute(alreadyFilteredAttributeName, Boolean.TRUE);  
        try {  
	        // protected abstract void 方法
            doFilterInternal(httpRequest, httpResponse, filterChain);  
        }  
        finally {  
            // Remove the "already filtered" request attribute for this request.  
            request.removeAttribute(alreadyFilteredAttributeName);  
        }  
    }  
}
```


# 跳过过滤器的逻辑 1

```java
/* ------------------------ OncePerRequestFilter ------------------------ */
private boolean skipDispatch(HttpServletRequest request) {  

    if ( 
	    // DispatcherType.ASYNC.equals(request.getDispatcherType())
	    isAsyncDispatch(request) &&
	    // protected boolean方法, 默认直接返回true
	    shouldNotFilterAsyncDispatch()) {  
        return true;  
    }  
    if (
	    // ERROR_REQUEST_URI_ATTRIBUTE = "jakarta.servlet.error.request_uri"
		request.getAttribute(WebUtils.ERROR_REQUEST_URI_ATTRIBUTE) != null &&
		// protected boolean方法, 默认直接返回true
        shouldNotFilterErrorDispatch()) {  
        return true;  
    }  
    return false;  
}
```
