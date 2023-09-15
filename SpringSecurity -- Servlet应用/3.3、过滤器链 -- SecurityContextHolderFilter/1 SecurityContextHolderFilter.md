
```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)  
      throws IOException, ServletException {  
    doFilter((HttpServletRequest) request, (HttpServletResponse) response, chain);  
}
private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)      throws ServletException, IOException {  

	// FILTER_APPLIED = SecurityContextHolderFilter.class.getName() + ".APPLIED"
    if (request.getAttribute(FILTER_APPLIED) != null) {  
        chain.doFilter(request, response);  
        return; 
	}  
    request.setAttribute(FILTER_APPLIED, Boolean.TRUE);  

    Supplier<SecurityContext> deferredContext = this.securityContextRepository.loadDeferredContext(request); 
    try {  
        this.securityContextHolderStrategy.setDeferredContext(deferredContext);  
        chain.doFilter(request, response);  
    }  
    finally {  
        this.securityContextHolderStrategy.clearContext();  
        request.removeAttribute(FILTER_APPLIED);  
    }  
}
```