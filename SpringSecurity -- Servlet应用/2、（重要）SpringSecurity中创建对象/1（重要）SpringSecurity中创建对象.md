
# 一、建造者 

SpringSecurity中创建对象使用建造者模式，提供SecurityBuilder接口来创建SpringSecurity中需要使用对象。

## SecurityBuilder

下面是该接口的定义：
```java
/* Type parameters: <O> – The type of the Object being built
 *     
 */
public interface SecurityBuilder<O> {
	O build() throws Exception;
}
```
其中类型参数O表示需要创建的对象的类型。

-- --
## AbstractSecurityBuilder

AbstractSecurityBuilder 是SecurityBuilder的抽象类，下面是该类的定义。
```java
public abstract class AbstractSecurityBuilder<O> implements SecurityBuilder<O>{}
```

下面来看AbstractSecurityBuilder对于build()的实现。
```java
/* ---------------------------------- AbstractSecurityBuilder ---------------------------------- */
@Override  
public final O build() throws Exception {  
	// this.building 默认初值为 new AtomicBoolean()
    if (this.building.compareAndSet(false, true)) {  
	    // doBuild()是 protected abstract方法
        this.object = doBuild();  
        return this.object;  
    }  
    throw new AlreadyBuiltException("This object has already been built");  
}
```
# 二、建造者配置者 -- SecurityConfigurer




下面来看该类的定义。
```java
public abstract class SecurityConfigurerAdapter<O, B extends SecurityBuilder<O>> implements SecurityConfigurer<O, B>
```


# 三、AbstractConfiguredSecurityBuilder（重要）

下面是该类的定义。
```java
/* --------------------------- AbstractConfiguredSecurityBuilder --------------------------- 
 * Type parameters:
 *     <O> – The object that this builder returns
 *     <B> – The type of this builder (that is returned by the base class)
 */ 
public abstract class AbstractConfiguredSecurityBuilder<O, B extends SecurityBuilder<O>> extends AbstractSecurityBuilder<O>{}
```
类型参数：
- O -- 需要创建的对象的类型。
- B -- 该Builder自身的类型。

该类名字（已经被配置的建造者）说明该类有两个特性：
- 建造者 -- 能够建造对象。
- 能被配置 -- 能够在建造对象之前被配置，这些配置能在创建对象时产生影响，从而产生类别相同但属性不同的对象。

该类内部维护一组建造者配置者来对自身进行配置。
```java
private final LinkedHashMap<Class<? extends SecurityConfigurer<O, B>>, List<SecurityConfigurer<O, B>>> configurers = new LinkedHashMap<>();
```

下面看该类的建造方法对该类的特性做进一步说明。
```java
/* ----------------------------- AbstractConfiguredSecurityBuilder ----------------------------- */
protected final O doBuild() throws Exception {  
    synchronized (this.configurers) {  

	    // 初始化该建造者(调用 SecurityConfigurer 的初始化方法完成 SecurityConfigurer 的初始化)
        this.buildState = BuildState.INITIALIZING;  
        beforeInit();       // protected No Op方法
        init();

        // 配置该建造者(使用初始化完成的 SecurityConfigurer 来对该建造者进行配置)
        this.buildState = BuildState.CONFIGURING;  
        beforeConfigure();  // protected No Op方法
        configure();

        // 创建对象
        this.buildState = BuildState.BUILDING;  
        O result = performBuild();   // protected abstract方法

		// 创建完成
        this.buildState = BuildState.BUILT;  

        return result;  
    }  
}
```
## 1）初始化建造者
```java
private void init() throws Exception {  
    Collection<SecurityConfigurer<O, B>> configurers = getConfigurers();  
    for (SecurityConfigurer<O, B> configurer : configurers) {  
        configurer.init((B) this);  
    }  
    for (SecurityConfigurer<O, B> configurer : this.configurersAddedInInitializing) {  
        configurer.init((B) this);  
    }  
}
```
## 2）配置建造者
```java
private void configure() throws Exception {  
    Collection<SecurityConfigurer<O, B>> configurers = getConfigurers();  
    for (SecurityConfigurer<O, B> configurer : configurers) {  
        configurer.configure((B) this);  
    }  
}
```






