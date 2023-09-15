
```java
@Configuration(proxyBeanMethods = false)  
public class WebSecurityConfiguration implements ImportAware, BeanClassLoaderAware {

	// DEFAULT_FILTER_NAME = "springSecurityFilterChain"
	@Bean(name = AbstractSecurityWebApplicationInitializer.DEFAULT_FILTER_NAME)  
	public Filter springSecurityFilterChain() throws Exception {  
		// WebSecurityConfiguration.this.securityFilterChains:
		// - 类型 List<SecurityFilterChain>
		// - 默认初值 Collections.emptyList()
		// - 自动装配 @Autowired(required = false)
	    boolean hasFilterChain = !this.securityFilterChains.isEmpty();  
	    if (!hasFilterChain) {  
		    // 1. WebSecurity#addSecurityFilterChainBuilder():
		    // 简单地向 WebSecurity.this.securityFilterChainBuilders 中添加元素
		    // 2. WebSecurity#this.securityFilterChainBuilders: 
		    // - 默认初值 new ArrayList<SecurityBuilder<? extends SecurityFilterChain>>()
	        this.webSecurity.addSecurityFilterChainBuilder(
		        () -> {  
			        // this.httpSecurity: 自动装配 @Autowired(required = false)
			        this.httpSecurity.authorizeHttpRequests(
				        (authorize) -> authorize.anyRequest().authenticated()
				    ); 
			        this.httpSecurity.formLogin(Customizer.withDefaults());  
			        this.httpSecurity.httpBasic(Customizer.withDefaults());  
			        return this.httpSecurity.build();  
	            }
	        );  
	    }  

		// 
	    for (SecurityFilterChain securityFilterChain : this.securityFilterChains) {  
	        this.webSecurity.addSecurityFilterChainBuilder(() -> securityFilterChain);  
	    }

		// WebSecurityConfiguration.this.webSecurityCustomizers:
		// - 类型 List<WebSecurityCustomizer>
		// - 默认初值 Collections.emptyList()
		// - 自动装配 @Autowired(required = false)
	    for (WebSecurityCustomizer customizer : this.webSecurityCustomizers) {  
	        customizer.customize(this.webSecurity);  
	    }
	    // 返回FilterProxyChain实例对象, 注入到Spring容器中
	    return this.webSecurity.build();  
	}
}
```


# 附录

1）addSecurityFilterChainBuilder()
```java
public WebSecurity addSecurityFilterChainBuilder(SecurityBuilder<? extends SecurityFilterChain> securityFilterChainBuilder) {  
	// this.securityFilterChainBuilders 默认初值为 new ArrayList<SecurityBuilder<? extends SecurityFilterChain>>()
    this.securityFilterChainBuilders.add(securityFilterChainBuilder);  
    return this;
}
```