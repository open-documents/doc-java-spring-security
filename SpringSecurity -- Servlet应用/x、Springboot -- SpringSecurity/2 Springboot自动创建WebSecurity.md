
```java
@Configuration(proxyBeanMethods = false)  
public class WebSecurityConfiguration implements ImportAware, BeanClassLoaderAware {

	@Autowired(required = false)  
	public void setFilterChainProxySecurityConfigurer(
		ObjectPostProcessor<Object> objectPostProcessor,
        ConfigurableListableBeanFactory beanFactory
    ) throws Exception {  

	    this.webSecurity = objectPostProcessor.postProcess(new WebSecurity(objectPostProcessor));  

		... // debug

		// 从Spring容器中获取类型为 WebSecurityConfigurer 的所有Bean
	    List<SecurityConfigurer<Filter, WebSecurity>> webSecurityConfigurers = 
		    new AutowiredWebSecurityConfigurersIgnoreParents(beanFactory).getWebSecurityConfigurers();  

		// 对 
	    webSecurityConfigurers.sort(AnnotationAwareOrderComparator.INSTANCE);  

	    Integer previousOrder = null;  
	    Object previousConfig = null;  
	    for (SecurityConfigurer<Filter, WebSecurity> config : webSecurityConfigurers) {  
	        Integer order = AnnotationAwareOrderComparator.lookupOrder(config);  
	        if (previousOrder != null && previousOrder.equals(order)) {  
	            throw new IllegalStateException(...);  
	        }  
	        previousOrder = order;  
	        previousConfig = config;  
	    }  

		// 利用用户提供的WebSecurityConfigurer接口实现类自定义 Springboot自动创建的WebSecurity
	    for (SecurityConfigurer<Filter, WebSecurity> webSecurityConfigurer : webSecurityConfigurers) {  
	        this.webSecurity.apply(webSecurityConfigurer);  
	    }  
	    this.webSecurityConfigurers = webSecurityConfigurers;  
	}
}
```

# Springboot自动注册ObjectPostProcessor

```java
@Configuration(proxyBeanMethods = false)  
@Role(BeanDefinition.ROLE_INFRASTRUCTURE)  
public class ObjectPostProcessorConfiguration {
	@Bean  
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)  
	public ObjectPostProcessor<Object> objectPostProcessor(AutowireCapableBeanFactory beanFactory) {  
	    return new AutowireBeanFactoryObjectPostProcessor(beanFactory);  
	}
}
```