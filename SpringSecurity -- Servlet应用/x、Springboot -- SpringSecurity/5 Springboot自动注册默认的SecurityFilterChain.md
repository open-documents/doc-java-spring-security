
```java
@Configuration(proxyBeanMethods = false)  
@ConditionalOnWebApplication(type = Type.SERVLET)  
class SpringBootWebSecurityConfiguration {

	@Configuration(proxyBeanMethods = false)  
	// 这个条件很重要, 决定了如果用户自己指定了SecurityFilterChain 那么该SecurityFilterChain就不自动注册
	@ConditionalOnDefaultWebSecurity          
	static class SecurityFilterChainConfiguration {  
	    @Bean  
	    @Order(SecurityProperties.BASIC_AUTH_ORDER)  
	    SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {  
	        http.authorizeHttpRequests((requests) -> requests.anyRequest().authenticated());  
	        http.formLogin(withDefaults());  
	        http.httpBasic(withDefaults());  
	        return http.build();
	    }  
	}
}
```
