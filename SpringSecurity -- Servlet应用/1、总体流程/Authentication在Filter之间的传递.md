
因为Authentication在随后的权限检查中也会使用到，因此 Authentication 需要在多个类之间传递。关系总结如下：

1）Authentication 保存在 SecurityContext 中。



2）SecurityContextHolder 保存在 SecurityContextHolderStrategy 中。

因为SecurityContextHolderStrategy 实际存储的是SecurityContext，只是以Supplier的方式保存，所以叫做SecurityContextHolder而已，所以在需要 SecurityContext 对象的地方是直接通过 SecurityContextHolderStrategy的 getContext() 和 getDeferredContext() 获得。

以 SecurityContextHolderStrategy 的一个实现类为例。
```java
final class ThreadLocalSecurityContextHolderStrategy implements SecurityContextHolderStrategy {
	private static final ThreadLocal<Supplier<SecurityContext>> contextHolder = new ThreadLocal<>();
	public SecurityContext getContext() {  
	    return getDeferredContext().get();  
	}
	public Supplier<SecurityContext> getDeferredContext() {  
	    Supplier<SecurityContext> result = contextHolder.get();  
	    if (result == null) {  
	        SecurityContext context = createEmptyContext();  
	        result = () -> context;  
	        contextHolder.set(result);  
	    }  
	    return result;  
	}
}
```

3）SecurityContextHolderStrategy 以静态属性的方式保存在 SecurityContextHolder 中，从而保证了全局唯一，在需要SecurityContextHolderStrategy 的地方用 SecurityContextHolder#getContextHolderStrategy()方法获得。


## 从SecurityContextHolder获取SecurityContextHolderStrategy

```java
/* ------------------------------ SecurityContextHolder ------------------------------ */
public static SecurityContextHolderStrategy getContextHolderStrategy() {  
    return strategy;  
}
```
接下来看SecurityContextHolder.this.strategy属性是如何初始化的。可以从下面看到：SecurityContextHolderStrategy是在SecurityContextHolder静态代码块中完成初始化的。
```java
/* ------------------------------ SecurityContextHolder ------------------------------ */
static {  
    initialize();  
}
private static void initialize() {  
    initializeStrategy();  
    // initializeCount: 类型 static int, 初值 0
    initializeCount++;  
}
private static void initializeStrategy() {  
	// strategyName: 
	// - 类型     String
	// - 默认初值 System.getProperty(SYSTEM_PROPERTY), SYSTEM_PROPERTY = "spring.security.strategy"
	// - 仅支持set注入

	// MODE_PRE_INITIALIZED = "MODE_PRE_INITIALIZED"
    if (MODE_PRE_INITIALIZED.equals(strategyName)) {  
        Assert.state(strategy != null, ...);  
        return; 
	}  

	// strategyName长度为0时, 设置默认值
    if (!StringUtils.hasText(strategyName)) {  
        // MODE_THREADLOCAL = "MODE_THREADLOCAL"
        strategyName = MODE_THREADLOCAL;  
    }  

	// ------------- 下面是根据strategyName来选择对应的SecurityContextHolderStrategy实现类 -------------

    if (strategyName.equals(MODE_THREADLOCAL)) {  
        strategy = new ThreadLocalSecurityContextHolderStrategy();  
        return; 
	}  
    if (strategyName.equals(MODE_INHERITABLETHREADLOCAL)) {  
        strategy = new InheritableThreadLocalSecurityContextHolderStrategy();  
        return; 
	}  
    if (strategyName.equals(MODE_GLOBAL)) {  
        strategy = new GlobalSecurityContextHolderStrategy();  
        return; 
	}  
    // Try to load a custom strategy  
    try {  
        Class<?> clazz = Class.forName(strategyName);  
        Constructor<?> customStrategy = clazz.getConstructor();  
        strategy = (SecurityContextHolderStrategy) customStrategy.newInstance();  
    }  
    catch (Exception ex) {  
        ReflectionUtils.handleReflectionException(ex);  
    }  
}
```

## 从SecurityContextHolderStrategy获取SecurityContext

可以从上面看到默认的 SecurityContextHolderStrategy 实现类是 ThreadLocalSecurityContextHolderStrategy 。下面来看 ThreadLocalSecurityContextHolderStrategy 对象如何获取 SecurityContext ，进而获取Authentication。
```java
/* -------------------------- ThreadLocalSecurityContextHolderStrategy -------------------------- */
public SecurityContext getContext() {  
    return getDeferredContext().get();  
}
public Supplier<SecurityContext> getDeferredContext() {  
	// contextHolder: 
	// - 类型ThreadLocal<Supplier<SecurityContext>>, 默认初值 new ThreadLocal<>()
	// - 仅支持set注入
    Supplier<SecurityContext> result = contextHolder.get();  
    if (result == null) {  
	    // createEmptyContext(): return new SecurityContextImpl()
        SecurityContext context = createEmptyContext();  
        result = () -> context;  
        contextHolder.set(result);  
    }  
    return result;  
}
```
## 获取Authentication

从上面可以看出，默认的SecurityContext实现类是 SecurityContextImpl ，接下来看 SecurityContextImpl 是如何获取Authentication的。
```java

```
