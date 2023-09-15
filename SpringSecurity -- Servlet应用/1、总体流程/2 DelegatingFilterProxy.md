
Since：1.2

先看该类的定义。
```java
public class DelegatingFilterProxy extends GenericFilterBean
```

下面来看该类最重要的doFilter()方法：
- 获取WebApplicationContext实例。
- 从WebApplicationContext实例获取被代理的Filter Bean（因为使用的是BeanFactory#getBean()所以属于懒加载）。
- 可选地执行被代理的Filter#init()，完成被代理的Filter的初始化工作。
- 将真正的过滤操作交给被代理的Filter。

```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws ServletException, IOException {  
  
    // Lazily initialize the delegate if necessary.  
    Filter delegateToUse = this.delegate;  

	// ------------- 这部分代码和Filter#init()过程中的initFilterBean()逻辑一致 -------------
    if (delegateToUse == null) {  
        synchronized (this.delegateMonitor) {  
            delegateToUse = this.delegate;  
            if (delegateToUse == null) {  
	            WebApplicationContext wac = findWebApplicationContext();  
	            if (wac == null) {  
	                throw new IllegalStateException("No WebApplicationContext found: " +  
                     "no ContextLoaderListener or DispatcherServlet registered?");  
	            }  
	            delegateToUse = initDelegate(wac);  
            }  
            this.delegate = delegateToUse;  
        }  
    }  
    // -----------------------------------------------------------------------------------
  
    // 调用this.delete的doFilter()方法
    invokeDelegate(delegateToUse, request, response, filterChain);  
}
```

## findWebApplicationContext()

获取WebApplicationContext实例顺序：
- 

```java
protected WebApplicationContext findWebApplicationContext() {  
	// this.webApplicationContext: 默认初值 null, 仅支持构造器注入, 不支持set()
    if (this.webApplicationContext != null) {  
        // The user has injected a context at construction time -> use it...  
        if (this.webApplicationContext instanceof ConfigurableApplicationContext cac && !cac.isActive()) {  
            // The context has not yet been refreshed -> do so before returning it...  
            cac.refresh();  
        }  
        return this.webApplicationContext;  
    }  
    // 返回this.contextAttribute, 默认初值为null, 仅支持set注入, 不支持构造器注入
    String attrName = getContextAttribute();  
    if (attrName != null) {  
        return WebApplicationContextUtils.getWebApplicationContext(getServletContext(), attrName);  
    }  
    else {  
        return WebApplicationContextUtils.findWebApplicationContext(getServletContext());  
    }  
}
```
## initDelegate()
```java
protected Filter initDelegate(WebApplicationContext wac) throws ServletException {  
    String targetBeanName = getTargetBeanName();  
    Assert.state(targetBeanName != null, "No target bean name set");  
    Filter delegate = wac.getBean(targetBeanName, Filter.class);  
    // isTargetFilterLifecycle(): 返回this.targetFilterLifecycle
    // - 决定是否调用target filter bean的Filter#init()和 Filter#destroy()方法
    // - 默认为false
    if (isTargetFilterLifecycle()) {  
        delegate.init(getFilterConfig());  
    }  
    return delegate;  
}
```


# 