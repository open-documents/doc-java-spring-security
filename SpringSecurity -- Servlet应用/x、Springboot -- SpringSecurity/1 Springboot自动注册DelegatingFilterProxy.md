
Springboot为SpringSecurity自动注册了DelegatingFilterProxyRegistrationBean Bean。
```java
@AutoConfiguration(after = SecurityAutoConfiguration.class)  
@ConditionalOnWebApplication(type = Type.SERVLET)  
@EnableConfigurationProperties(SecurityProperties.class)  
@ConditionalOnClass({ AbstractSecurityWebApplicationInitializer.class, SessionCreationPolicy.class })  
public class SecurityFilterAutoConfiguration {

	// DEFAULT_FILTER_NAME = "springSecurityFilterChain"
	@Bean  
	@ConditionalOnBean(name = DEFAULT_FILTER_NAME)  
	public DelegatingFilterProxyRegistrationBean securityFilterChainRegistration(SecurityProperties securityProperties) {  

		// 向ServletContext中添加DelegatingFilterProxy过滤器
	    DelegatingFilterProxyRegistrationBean registration = 
		    new DelegatingFilterProxyRegistrationBean(  
		         DEFAULT_FILTER_NAME
		    );  
	    // 过滤器优先级: -100(OrderedFilter.REQUEST_WRAPPER_FILTER_MAX_ORDER - 100 = -100)
		// 因为是从securityProperties中取值, 因此支持自定义
	    registration.setOrder(securityProperties.getFilter().getOrder());  
	    // 过滤器所属的Servlet: DispatcherServlet
	    // 因为是从securityProperties中取值, 因此支持自定义
	    registration.setDispatcherTypes(getDispatcherTypes(securityProperties));  
	    return registration;  
	}
}
```

-- -- 
下面介绍向ServletContext中添加DelegatingFilterProxy过滤器的具体过程。


首先介绍DelegatingFilterProxyRegistrationBean。下面是该类的定义：
```java
public class DelegatingFilterProxyRegistrationBean extends AbstractFilterRegistrationBean<DelegatingFilterProxy>  
implements ApplicationContextAware {}
```
该类的父类RegistrationBean实现了ServletContextInitializer接口，该接口的功能是向ServletContext中注册Servlet、Filter、Listener、初始化过程中的attribute。下面来看RegistrationBean对ServletContextInitializer接口的实现。
```java
/* ---------------------------------- RegistrationBean ---------------------------------- */
@Override  
public final void onStartup(ServletContext servletContext) throws ServletException {  
    String description = getDescription();  
    // - isEnabled() 返回 this.enabled
    // - this.enabled 默认初值为 true, 支持set
    if (!isEnabled()) {  
        ... // logger.info()
        return;  
    }  
    // register()方法是 protected abstract方法
    register(description, servletContext);  
}
```
下面来看RegistrationBean子类、DelegatingFilterProxyRegistrationBean父父类DynamicRegistrationBean对于该register()的实现。
```java
/* -------------------------------- DynamicRegistrationBean -------------------------------- */
@Override  
protected final void register(String description, ServletContext servletContext) {  
	// addRegistration()方法是 protected abstract方法
    D registration = addRegistration(description, servletContext);  
    if (registration == null) {  
        if (this.ignoreRegistrationFailure) {  
            ... // logger.info()
            return;
		}  
        throw new IllegalStateException(...);  
    }  
    configure(registration);  
}
```
下面进一步看DynamicRegistrationBean子类、DelegatingFilterProxyRegistrationBean父类对于addRegistration()的实现。
```java
/* --------------------------- AbstractFilterRegistrationBean --------------------------- */
@Override  
protected Dynamic addRegistration(String description, ServletContext servletContext) {  
	// 该类是 public abstract方法
    Filter filter = getFilter();  
    return servletContext.addFilter(getOrDeduceName(filter), filter);  
}
```
最后再看DelegatingFilterProxyRegistrationBean自身对于getFilter()的实现。
```java
/* ---------------------- DelegatingFilterProxyRegistrationBean ---------------------- */
@Override  
public DelegatingFilterProxy getFilter() {  
	// 
	return new DelegatingFilterProxy(this.targetBeanName, getWebApplicationContext()) {  
        @Override  
        protected void initFilterBean() throws ServletException {  
            // Don't initialize filter bean on init()  
        }  
    };  
}
```
可以得出这样的结论：DelegatingFilterProxyRegistrationBean只是简单地向ServletContext中添加了一个DelegatingFilterProxy过滤器（Filter）。




