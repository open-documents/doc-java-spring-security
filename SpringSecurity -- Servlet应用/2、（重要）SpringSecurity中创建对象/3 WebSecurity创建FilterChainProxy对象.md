
看WebSecurity对于performBuild()的实现。
```java
/* ----------------------------------- WebSecurity ----------------------------------- */
protected Filter performBuild() throws Exception {  
	... // Assert

	// 1. this.ignoredRequests: 
	// - 类型 List<RequestMatcher>
	// - 默认初值 new ArrayList<RequestMatcher>()
	// 2. this.securityFilterChainBuilders: 
	// - 类型 List<SecurityBuilder<? extends SecurityFilterChain>>
	// - 默认初值 new ArrayList<SecurityBuilder<? extends SecurityFilterChain>>()
	// - 由前面可知, 向该列表中添加了一个HttpSecurity实例
    int chainSize = this.ignoredRequests.size() + this.securityFilterChainBuilders.size();  
    List<SecurityFilterChain> securityFilterChains = new ArrayList<>(chainSize);  
    List<RequestMatcherEntry<List<WebInvocationPrivilegeEvaluator>>> requestMatcherPrivilegeEvaluatorsEntries = new ArrayList<>();  

    for (RequestMatcher ignoredRequest : this.ignoredRequests) {  
		... // logger.warn()
        SecurityFilterChain securityFilterChain = new DefaultSecurityFilterChain(ignoredRequest);  
        securityFilterChains.add(securityFilterChain);  
        requestMatcherPrivilegeEvaluatorsEntries  
	        .add(getRequestMatcherPrivilegeEvaluatorsEntry(securityFilterChain));  
    }  

	// ----------------------------- 重要代码 -----------------------------
	// WebSecurity.this.securityFilterChainBuilders: 
	// 默认情况, 只有前面添加的1个HttpSecurity
    for (SecurityBuilder<? extends SecurityFilterChain> securityFilterChainBuilder : this.securityFilterChainBuilders) {  
        SecurityFilterChain securityFilterChain = securityFilterChainBuilder.build();  
        securityFilterChains.add(securityFilterChain);  
        requestMatcherPrivilegeEvaluatorsEntries  
            .add(getRequestMatcherPrivilegeEvaluatorsEntry(securityFilterChain));  
    }  

    if (this.privilegeEvaluator == null) {  
        this.privilegeEvaluator = new RequestMatcherDelegatingWebInvocationPrivilegeEvaluator(  
            requestMatcherPrivilegeEvaluatorsEntries);  
    }  

	// ----------------------------- 重要代码 -----------------------------
	// 
    FilterChainProxy filterChainProxy = new FilterChainProxy(securityFilterChains);  
    if (this.httpFirewall != null) {  
        filterChainProxy.setFirewall(this.httpFirewall);  
    }  
    if (this.requestRejectedHandler != null) {  
        filterChainProxy.setRequestRejectedHandler(this.requestRejectedHandler);  
    }  
    else if (!this.observationRegistry.isNoop()) {  
        CompositeRequestRejectedHandler requestRejectedHandler = new CompositeRequestRejectedHandler(  
            new ObservationMarkingRequestRejectedHandler(this.observationRegistry),  
            new HttpStatusRequestRejectedHandler());  
        filterChainProxy.setRequestRejectedHandler(requestRejectedHandler);  
    }  
    filterChainProxy.setFilterChainDecorator(getFilterChainDecorator());  
    filterChainProxy.afterPropertiesSet();  
  
    Filter result = filterChainProxy;  

	... // logger.debug()

	// this.postBuildAction: Runnalbe类型, 默认为No Op, 支持set注入
    this.postBuildAction.run();  
    return result;  
}
```

