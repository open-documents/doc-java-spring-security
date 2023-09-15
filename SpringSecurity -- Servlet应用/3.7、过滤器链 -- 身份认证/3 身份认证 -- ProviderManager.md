
首先说明ProviderManager在身份认证中的总体功能：
- 将身份认证的具体实现交由内部的AuthenticationProvider接口实现类完成。
- 除此之外，每个ProviderManager有一个父ProviderManager，当子ProviderManager认证不成功时会尝试用父ProviderManager进行认证。注入到UsernamePasswordAuthenticationFilter中的是子ProviderManager。

默认情况下，子ProviderManager和父ProviderManager的providers都只包含一个元素：
- 子ProviderManager -- AnonymousAuthenticationProvider
- 父ProviderManager -- DaoAuthenticationProvider（更重要）

```java
/* ----------------------------- ProviderManager ----------------------------- */
public Authentication authenticate(Authentication authentication) throws AuthenticationException {  
    Class<? extends Authentication> toTest = authentication.getClass();  

    AuthenticationException lastException = null;  
    AuthenticationException parentException = null;  
    Authentication result = null;  
    Authentication parentResult = null;  
    int currentPosition = 0;  

	// 默认只有1个: AnonymousAuthenticationProvider
    int size = this.providers.size();  
    for (AuthenticationProvider provider : getProviders()) {  
        if (!provider.supports(toTest)) {  
            continue;  
        }  

		... // logger.trace()

		// 开始身份验证
        try {  
            result = provider.authenticate(authentication);  
            if (result != null) {  
	            copyDetails(authentication, result);  
	            break;
			}  
        }  
        catch (AccountStatusException | InternalAuthenticationServiceException ex) {  
            prepareException(ex, authentication);  
            // SEC-546: Avoid polling additional providers if auth failure is due to  
            // invalid account status         
            throw ex;  
        }  
        catch (AuthenticationException ex) {  
            lastException = ex;  
        }  
    }  

    if (result == null && this.parent != null) {  
        // Allow the parent to try.  
        try {  
            parentResult = this.parent.authenticate(authentication);  
            result = parentResult;  
        }  
        catch (ProviderNotFoundException ex) {  
            // ignore as we will throw below if no other exception occurred prior to  
            // calling parent and the parent         
            // may throw ProviderNotFound even though a provider in the child already         
            // handled the request  
		}  
        catch (AuthenticationException ex) {  
            parentException = ex;  
            lastException = ex;  
        }  
    }  

    if (result != null) {  
        if (this.eraseCredentialsAfterAuthentication && (result instanceof CredentialsContainer)) {  
            // Authentication is complete. Remove credentials and other secret data  
            // from authentication 
			((CredentialsContainer) result).eraseCredentials();  
        }  
        // If the parent AuthenticationManager was attempted and successful then it  
        // will publish an AuthenticationSuccessEvent      
        // This check prevents a duplicate AuthenticationSuccessEvent if the parent      
        // AuthenticationManager already published it      
        if (parentResult == null) {  
            this.eventPublisher.publishAuthenticationSuccess(result);  
        }  
  
        return result;  
    }  
  
    // Parent was null, or didn't authenticate (or throw an exception).  
    if (lastException == null) {  
        lastException = new ProviderNotFoundException(...);  
    }  
    // If the parent AuthenticationManager was attempted and failed then it will  
    // publish an AbstractAuthenticationFailureEvent    
    // This check prevents a duplicate AbstractAuthenticationFailureEvent if the  
    // parent AuthenticationManager already published it   
    if (parentException == null) {  
        prepareException(lastException, authentication);  
    }  
    throw lastException;  
}
```