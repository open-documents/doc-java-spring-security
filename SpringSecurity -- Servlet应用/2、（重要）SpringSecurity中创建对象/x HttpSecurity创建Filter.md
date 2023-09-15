
首先必须理清思路：

1）SecurityFilterChain是由HttpSecurity创建的，所以SecurityFilterChain对象实例中包含的Filter数组中的元素（即Filter对象实例）由HttpSecurity提供。

2）而HttpSecurity本身是SecurityBuilder接口实现类，SpringSecurity中提供了自定义SecurityBuilder的接口SecurityConfigurer。



# 准备工作

getOrApply()
```java
/* ------------------------------------ HttpSecurity ------------------------------------ 
 * Return: 
 *     SecurityConfigurerAdapter<DefaultSecurityFilterChain, HttpSecurity>子类对象
 */
private <C extends SecurityConfigurerAdapter<DefaultSecurityFilterChain, HttpSecurity>> C getOrApply(C configurer) throws Exception {  
    C existingConfig = (C) getConfigurer(configurer.getClass());  
    if (existingConfig != null) {  
        return existingConfig;  
    }  
    return apply(configurer);  
}
```
## getConfigurer()
```java
/* ------------------------------ AbstractConfiguredSecurityBuilder ------------------------------ 
 * 
 */
public <C extends SecurityConfigurer<O, B>> C getConfigurer(Class<C> clazz) {  
	// this.configurers: 默认初值 new LinkedHashMap<Class<? extends SecurityConfigurer<O, B>>()
    List<SecurityConfigurer<O, B>> configs = this.configurers.get(clazz);  
    if (configs == null) {  
        return null;  
    }  

    Assert.state(configs.size() == 1, () -> "Only one configurer expected for type " + clazz + ", but got " + configs);  
    return (C) configs.get(0);  
}
```
## apply()
```java
/* ---------------------------- AbstractConfiguredSecurityBuilder ---------------------------- 
 * Type parameters:
 *     <O> – The object that this builder returns
 *     <B> – The type of this builder (that is returned by the base class), B extends SecurityBuilder<O>
 */
public <C extends SecurityConfigurerAdapter<O, B>> C apply(C configurer) throws Exception {  
    configurer.addObjectPostProcessor(this.objectPostProcessor);  
    configurer.setBuilder((B) this);  
    add(configurer);  
    return configurer;  
}

private <C extends SecurityConfigurer<O, B>> void add(C configurer) {  
    ... // Assert.notNull(...);

    Class<? extends SecurityConfigurer<O, B>> clazz = (Class<? extends SecurityConfigurer<O, B>>) configurer.getClass();  
    synchronized (this.configurers) {  
        if (this.buildState.isConfigured()) {  
            throw new IllegalStateException("Cannot apply " + configurer + " to already built object");  
        }  
        List<SecurityConfigurer<O, B>> configs = null;  
        if (this.allowConfigurersOfSameType) {  
            configs = this.configurers.get(clazz);  
        }  
        configs = (configs != null) ? configs : new ArrayList<>(1);  
        configs.add(configurer);  
        this.configurers.put(clazz, configs);  
        if (this.buildState.isInitializing()) {  
            this.configurersAddedInInitializing.add(configurer);  
        }  
    }  
}
```

# 创建各个Filter

需要注意的是，HttpSecurity创建SecurityFilterChain中的Filter是通过SecurityConfigurer完成的。把前面提到的configure()过程再拿出来。
```java
/* ----------------------------- AbstractConfiguredSecurityBuilder ----------------------------- */
private void configure() throws Exception {  
	// 默认情况下, Springboot中默认的HttpSecurity一共13个SecurityConfigurer
    Collection<SecurityConfigurer<O, B>> configurers = getConfigurers();  
    for (SecurityConfigurer<O, B> configurer : configurers) {  
        configurer.configure((B) this);  
    }  
}
```

