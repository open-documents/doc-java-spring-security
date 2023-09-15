
默认情况下，权限认证是由RequestMatcherDelegatingAuthorizationManager完成。

在正式介绍权限认证过程之前，先介绍几个简单类。

1）RequestMatcherEntry：对RequestMatcher和Entry的简单封装。
```java
// 下面是全部定义
public class RequestMatcherEntry<T> {  
    private final RequestMatcher requestMatcher;   // 有get()方法
    private final T entry;                         // 有get()方法
    public RequestMatcherEntry(RequestMatcher requestMatcher, T entry) {  
        this.requestMatcher = requestMatcher;  
        this.entry = entry;  
    }  
}
```
2）RequestAuthorizationContext：授权过程中用到的组件的简单封装，是对HttpServletRequest和Map<String, String>的简单封装。
```java
// 下面是全部定义
public final class RequestAuthorizationContext {  
    private final HttpServletRequest request;        // 有get()方法
    private final Map<String, String> variables;     // 有get()方法
	public RequestAuthorizationContext(HttpServletRequest request) {  
        this(request, Collections.emptyMap());  
    }  
	public RequestAuthorizationContext(HttpServletRequest request, Map<String, String> variables) {  
        this.request = request;  
        this.variables = variables;  
    }
}
```

# 权限检查过程 -- check()

```java
/* ---------------------- RequestMatcherDelegatingAuthorizationManager ---------------------- */
public AuthorizationDecision check(Supplier<Authentication> authentication, HttpServletRequest request) {  
	... // logger.trace()

	// this.mappings: 
	// - 类型 List<RequestMatcherEntry<AuthorizationManager<RequestAuthorizationContext>>>
	// - 仅支持构造器注入
    for (RequestMatcherEntry<AuthorizationManager<RequestAuthorizationContext>> mapping : this.mappings) {  
        RequestMatcher matcher = mapping.getRequestMatcher();  
        MatchResult matchResult = matcher.matcher(request);  
        if (matchResult.isMatch()) {  
            AuthorizationManager<RequestAuthorizationContext> manager = mapping.getEntry();  
            ... // logger.trace()
            return manager.check(
	            authentication,  
                new RequestAuthorizationContext(request, matchResult.getVariables())
            );
        }  
    }  

	... // logger.trace()

    return DENY;  
}
```


# AuthorityAuthorizationManager


```java
/* --------------------- AuthorityAuthorizationManager --------------------- */
public AuthorizationDecision check(Supplier<Authentication> authentication, T object) {  
	// this.delegate
	// - 类型 AuthoritiesAuthorizationManager
	// - 默认初值 new AuthoritiesAuthorizationManager()
	// - 不支持注入, 但是提供各种方法改变该对象的属性
    return this.delegate.check(authentication, this.authorities);  
}
```


