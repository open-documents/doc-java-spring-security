
FilterChainProxy中包含了一组SecurityFilterChain，通过判断每个SecurityFilterChain针对URL的匹配情况，将真正的Filter工作进一步交给第一个匹配的SecurityFilterChain中的Filters。

```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {  
    boolean clearContext = request.getAttribute(FILTER_APPLIED) == null;  
    if (!clearContext) {  
        doFilterInternal(request, response, chain);  
        return; 
	}  
    try {  
        request.setAttribute(FILTER_APPLIED, Boolean.TRUE);  
        doFilterInternal(request, response, chain);  
    }  
    catch (Exception ex) {  
        Throwable[] causeChain = this.throwableAnalyzer.determineCauseChain(ex);  
        Throwable requestRejectedException = this.throwableAnalyzer  
            .getFirstThrowableOfType(RequestRejectedException.class, causeChain);  
        if (!(requestRejectedException instanceof RequestRejectedException)) {  
            throw ex;  
        }  
        this.requestRejectedHandler.handle((HttpServletRequest) request, (HttpServletResponse) response,  
            (RequestRejectedException) requestRejectedException);  
    }  
    finally {  
        this.securityContextHolderStrategy.clearContext();  
        request.removeAttribute(FILTER_APPLIED);  
    }  
}
```

# doFilterInternal()

```java
private void doFilterInternal(ServletRequest request, ServletResponse response, FilterChain chain)     throws IOException, ServletException {  

	// this.firewall: 类型HttpFirewall, 初始值 new StrictHttpFirewall()
    FirewalledRequest firewallRequest = this.firewall.getFirewalledRequest((HttpServletRequest) request);  
    HttpServletResponse firewallResponse = this.firewall.getFirewalledResponse((HttpServletResponse) response);  
    List<Filter> filters = getFilters(firewallRequest);  
    if (filters == null || filters.size() == 0) {  
        ... // logger.trace()
        firewallRequest.reset();  
        // this.filterChainDecorator
        // - 类型 FilterChainDecorator
        // - 初始值 new VirtualFilterChainDecorator()
        this.filterChainDecorator.decorate(chain).doFilter(firewallRequest, firewallResponse);  
        return; 
	}  
	... // logger.debug()
    FilterChain reset = (req, res) -> {  
		... // logger.debug()
        // Deactivate path stripping as we exit the security filter chain  
        firewallRequest.reset();  
        chain.doFilter(req, res);  
    };  
    
    this.filterChainDecorator.decorate(reset, filters).doFilter(firewallRequest, firewallResponse);  
}
```

```java
private VirtualFilterChain(FilterChain chain, List<Filter> additionalFilters) {  
   this.originalChain = chain;  
   this.additionalFilters = additionalFilters;  
   this.size = additionalFilters.size();  
}  
  
@Override  
public void doFilter(ServletRequest request, ServletResponse response) throws IOException, ServletException {  
	// this.currentPosition: 默认初值 0
    if (this.currentPosition == this.size) {  
        this.originalChain.doFilter(request, response);  
        return; 
	}  
    this.currentPosition++;  
    Filter nextFilter = this.additionalFilters.get(this.currentPosition - 1);  
	... // log
    nextFilter.doFilter(request, response, this);  
}
```


# getFilters()

```java
private List<Filter> getFilters(HttpServletRequest request) {  
    int count = 0;  
    for (SecurityFilterChain chain : this.filterChains) {  
		... // logger.trace()
		// 每个SecurityFilterChain将真正的匹配工作交给自身的RequestMatcher实例
        if (chain.matches(request)) {  
            return chain.getFilters();  
        }  
    }  
    return null;  
}
```