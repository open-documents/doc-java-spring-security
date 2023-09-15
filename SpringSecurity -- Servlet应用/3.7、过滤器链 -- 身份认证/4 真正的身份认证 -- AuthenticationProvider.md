
# 一、AnonymousAuthenticationProvider

AnonymousAuthenticationProvider是默认情况下子ProviderManager中唯一的真正执行认证的AuthenticationProvider。

下面来看它的认证方法authenticate()。
```java
/* ---------------------------- AnonymousAuthenticationProvider ---------------------------- */
public Authentication authenticate(Authentication authentication) throws AuthenticationException { 

	// 不支持认证传进来的Authentication参数的种类直接返回
    if (!supports(authentication.getClass())) {  
        return null;  
    }  


    if (this.key.hashCode() != ((AnonymousAuthenticationToken) authentication).getKeyHash()) {  
      throw new BadCredentialsException(this.messages.getMessage("AnonymousAuthenticationProvider.incorrectKey",  
            "The presented AnonymousAuthenticationToken does not contain the expected key"));  
   }  
   return authentication;  
}
```


# 二、DaoAuthenticationProvider

DaoAuthenticationProvider是默认情况下父ProviderManager中唯一的真正执行认证的AuthenticationProvider。下面来看它进行身份验证的部分。
```java
/* --------------------- AbstractUserDetailsAuthenticationProvider --------------------- */
public Authentication authenticate(Authentication authentication) throws AuthenticationException { 
	... // Assert.isInstanceOf(只支持验证 UsernamePasswordAuthenticationToken)

	// 从 Authentication中获取Username, 即 Authentication#getName()
    String username = determineUsername(authentication);  
    boolean cacheWasUsed = true;  
    UserDetails user = this.userCache.getUserFromCache(username);  
    if (user == null) {  
        cacheWasUsed = false;
        try {  
        	// -------------------------------------------------------------------------
			// ----------------------------- 非常重要的方法 -----------------------------
			// -------------------------------------------------------------------------
	        // protected abstract方法, 获取UserDetails对象
            user = retrieveUser(username, (UsernamePasswordAuthenticationToken) authentication);  
        }
        catch (UsernameNotFoundException ex) {  
            ... // logger.debug(...)  
            if (!this.hideUserNotFoundExceptions) {  
	            throw ex;  
            }  
            throw new BadCredentialsException(...);  
        }  
        ... // Assert.notNull(user, ...);  
    }  

	// 下面是针对UserDetails做的最重要的3道认证:
	// - 

    try {  
	    // 第1道认证
	    // AbstractUserDetailsAuthenticationProvider#this.preAuthenticationChecks:
	    // 类型 UserDetailsChecker, 默认初值 new DefaultPreAuthenticationChecks()
	    // 仅支持set注入
        this.preAuthenticationChecks.check(user);  
        // 第2道认证: protected abstract方法
        additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken) authentication);  
    }  
    catch (AuthenticationException ex) {  
        if (!cacheWasUsed) {  
            throw ex;  
        }  
        // There was a problem, so try again after checking  
        // we're using latest data (i.e. not from the cache)      
        cacheWasUsed = false;  
        user = retrieveUser(username, (UsernamePasswordAuthenticationToken) authentication);  
        this.preAuthenticationChecks.check(user);  
        additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken) authentication);  
    }  

	// 第3道认证
    this.postAuthenticationChecks.check(user);  


    if (!cacheWasUsed) {  
        this.userCache.putUserInCache(user);  
    }  
    Object principalToReturn = user;  
    if (this.forcePrincipalAsString) {  
        principalToReturn = user.getUsername();  
    }  
    return createSuccessAuthentication(principalToReturn, authentication, user);  
}
```

## （重要）获取username对应的UserDetails -- retrieveUser()
```java
/* ---------------------------- DaoAuthenticationProvider ---------------------------- */
protected final UserDetails retrieveUser(String username, UsernamePasswordAuthenticationToken authentication) throws AuthenticationException {  
    prepareTimingAttackProtection();  
    try {
	    // -----------------------------------------------------------------------------
		// ----------------------------- 非常重要的一行代码 -----------------------------
		// -----------------------------------------------------------------------------
		// 根据username获取UserDetails对象, 用于后续的身份认证
	    // 1. DaoAuthenticationProvider.this.getUserDetailsService(): 返回 this.userDetailsService
	    // 2. DaoAuthenticationProvider.this.userDetailsService: 类型 UserDetailsService, 仅支持set注入
        UserDetails loadedUser = this.getUserDetailsService().loadUserByUsername(username);  
        if (loadedUser == null) {
            throw new InternalAuthenticationServiceException(  
               "UserDetailsService returned null, which is an interface contract violation");  
        }  
        return loadedUser;  
    }  
    catch (UsernameNotFoundException ex) {  
        mitigateAgainstTimingAttack(authentication);  
        throw ex;
    }  
    catch (InternalAuthenticationServiceException ex) {  
        throw ex;  
    }  
    catch (Exception ex) {  
        throw new InternalAuthenticationServiceException(ex.getMessage(), ex);  
    }  
}
```


## 第1道认证 -- 

## （重要）第2道认证 -- additionalAuthenticationChecks()

```java
/* ---------------------------- DaoAuthenticationProvider ---------------------------- */
protected void additionalAuthenticationChecks(UserDetails userDetails, UsernamePasswordAuthenticationToken authentication) throws AuthenticationException {

	// 用户输入的密码不能为空
    if (authentication.getCredentials() == null) {  
        this.logger.debug("Failed to authenticate since no credentials provided");  
        throw new BadCredentialsException(...);  
    }

	// 获取用户输入的密码
    String presentedPassword = authentication.getCredentials().toString();  
    // 将用户输入的密码presentedPassword 和 通过username得到的密码(可能从数据库查询)进行对比
    // DaoAuthenticationProvider#this.passwordEncoder:
    // - 类型 PasswordEncoder, 默认情况下是 DelegatingPasswordEncoder
    // - 仅支持 set注入
    if (!this.passwordEncoder.matches(presentedPassword, userDetails.getPassword())) {  
        this.logger.debug("Failed to authenticate since password does not match stored value");  
        throw new BadCredentialsException(...);  
    }
}
```

## 第3道认证 -- 


## 创建Authentication对象 -- createSuccessAuthentication()

在成功认证后，需要返回Authentication，用于后续的权限检查工作。
```java
/* --------------------- AbstractUserDetailsAuthenticationProvider --------------------- */
protected Authentication createSuccessAuthentication(Object principal, Authentication authentication, UserDetails user) {  

    UsernamePasswordAuthenticationToken result = UsernamePasswordAuthenticationToken.authenticated(
	    principal,  
        authentication.getCredentials(), 
        // AbstractUserDetailsAuthenticationProvider#this.authoritiesMapper:
        // - 将一组GrantedAuthority映射成另外一组GrantedAuthority
        // - 类型 GrantedAuthoritiesMapper
        // - 默认初值 new NullAuthoritiesMapper(), 将GrantedAuthority按原样映射
        // - 仅支持set注入
        this.authoritiesMapper.mapAuthorities(user.getAuthorities())
    );  
    result.setDetails(authentication.getDetails());  

    ... // logger.debug()

    return result;  
}
```