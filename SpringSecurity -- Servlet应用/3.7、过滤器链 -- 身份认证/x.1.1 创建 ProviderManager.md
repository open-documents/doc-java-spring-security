
ProviderManager 由 AuthenticationManagerBuilder 创建。下面是 AuthenticationManagerBuilder 的定义。
```java
public class AuthenticationManagerBuilder  
	extends AbstractConfiguredSecurityBuilder<AuthenticationManager, AuthenticationManagerBuilder>  
    implements ProviderManagerBuilder<AuthenticationManagerBuilder> {}
```
从其父类 AbstractConfiguredSecurityBuilder 知道，创建的 ProviderManager实例对象 可以自定义。


从前面知道 ProviderManager 的功能主要由 ProviderManager内部的一组

```java
protected ProviderManager performBuild() throws Exception {  
    if (!isConfigured()) {  
        ... // logger.debug()  
        return null;
	}  

    ProviderManager providerManager = new ProviderManager(
	    this.authenticationProviders,  
        this.parentAuthenticationManager
    );  

    if (this.eraseCredentials != null) {  
        providerManager.setEraseCredentialsAfterAuthentication(this.eraseCredentials);  
    }  
    if (this.eventPublisher != null) {  
        providerManager.setAuthenticationEventPublisher(this.eventPublisher);  
    }  
    providerManager = postProcess(providerManager);  
    return providerManager;  
}
```

