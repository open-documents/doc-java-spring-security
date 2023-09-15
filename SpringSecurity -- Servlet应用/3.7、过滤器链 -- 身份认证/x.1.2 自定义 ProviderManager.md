
因为 
- ProviderManager对象实例由 AuthenticationManagerBuilder 创建。
- AuthenticationManagerBuilder 继承了 AbstractConfiguredSecurityBuilder。

所以 AbstractConfiguredSecurityBuilder 提供方法来改变每个创建的 ProviderManager实例对象。

# 一、自定义 ProviderManager内部的AuthenticationProvider

由于 AuthenticationManagerBuilder 创建 ProviderManager对象实例 时，ProviderManager实例对象内部的 AuthenticationProvider列表是由  AuthenticationManagerBuilder.this.authenticationProviders属性。
```java
/* ------------------------- AuthenticationManagerBuilder ------------------------- */
protected ProviderManager performBuild() throws Exception {  
	... 
    ProviderManager providerManager = new ProviderManager(
	    this.authenticationProviders,  
        this.parentAuthenticationManager
    );  
    ... 
}
```
所以需要使用建造者配置者（SecurityConfigurer）来对AuthenticationManagerBuilder.this.authenticationProviders属性进行配置。
## 1）userDetailsService()
```java
/* ------------------------- AuthenticationManagerBuilder ------------------------- */
public <T extends UserDetailsService> DaoAuthenticationConfigurer<AuthenticationManagerBuilder, T> userDetailsService(T userDetailsService) throws Exception {  
    this.defaultUserDetailsService = userDetailsService;  
    return apply(new DaoAuthenticationConfigurer<>(userDetailsService));  
}
```
可以看到使用的是DaoAuthenticationConfigurer来对AuthenticationManagerBuilder.this.authenticationProviders属性进行配置。下面来看DaoAuthenticationConfigurer是如何改变这个属性的。

通过下面的过程向AuthenticationManagerBuilder添加了一个配置者DaoAuthenticationConfigurer。
```java
/* ------------------------- AuthenticationManagerBuilder ------------------------- */
private <C extends UserDetailsAwareConfigurer<AuthenticationManagerBuilder, ? extends UserDetailsService>> C apply(C configurer) throws Exception {  
    this.defaultUserDetailsService = configurer.getUserDetailsService();  
    return super.apply(configurer);  
}
/* ------------------------- AbstractConfiguredSecurityBuilder ------------------------- */
public <C extends SecurityConfigurerAdapter<O, B>> C apply(C configurer) throws Exception {  
    configurer.addObjectPostProcessor(this.objectPostProcessor);  
    configurer.setBuilder((B) this);  
    add(configurer);  
    return configurer;
}
```
下面继续看DaoAuthenticationConfigurer是如何配置AuthenticationManagerBuilder的。
```java
/* --------------------------- AbstractDaoAuthenticationConfigurer ---------------------------
 * Description: 该类是DaoAuthenticationConfigurer的父类
 */
public void configure(B builder) throws Exception {  
    this.provider = postProcess(this.provider);  
    builder.authenticationProvider(this.provider);  
}
/* ------------------------- AuthenticationManagerBuilder ------------------------- */
public AuthenticationManagerBuilder authenticationProvider(AuthenticationProvider authenticationProvider) {  
    this.authenticationProviders.add(authenticationProvider);  
    return this;
}
```
