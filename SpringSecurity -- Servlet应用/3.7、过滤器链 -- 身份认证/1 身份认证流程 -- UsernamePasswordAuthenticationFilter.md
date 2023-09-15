
参考文档：https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html

身份认证总体流程如下：



ProviderManager实现了AuthenticationManager接口，用于完成身份认证功能，但是它自身并不实现身份认证功能，而是将身份认证功能交由内部的一组AuthenticationProvider实现类完成。下面是ProviderManager内部对于该部分属性的定义。
```java
// ProviderManager#this.providers 仅支持构造函数注入
private List<AuthenticationProvider> providers = Collections.emptyList();
```

AuthenticationProvider接口用于真正完成身份认证功能。



AbstractAuthenticationProcessingFilter是用于处理认证请求的过滤器的父类。

目前Spring Security只提供了一个子类：UsernamePasswordAuthenticationFilter

首先看该过滤器的定义。
```java
public class UsernamePasswordAuthenticationFilter extends AbstractAuthenticationProcessingFilter{}
```

# doFilter()

首先看UsernamePasswordAuthenticationFilter中最重要的doFilter()逻辑，doFilter()由其父类提供。
```java
/* --------------------------- AbstractAuthenticationProcessingFilter --------------------------- */
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {  
    doFilter((HttpServletRequest) request, (HttpServletResponse) response, chain);  
}
private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {  

	// 因为该过滤器主要做认证处理, 因此如果判断请求不需要做认证处理, 则直接执行过滤器链中的下一个过滤器
	// 使用 this.requiresAuthenticationRequestMatcher#match(HttpServletRequest) 判断当前请求是否需要做认证
	// this.requiresAuthenticationRequestMatcher: RequestMatcher类型, 支持set注入和构造器注入
    if (!requiresAuthentication(request, response)) {  
        chain.doFilter(request, response);  
        return; 
	}  
    try {  
	    // attemptAuthentication(): public abstract方法
        Authentication authenticationResult = attemptAuthentication(request, response);  
        if (authenticationResult == null) {  
            // return immediately as subclass has indicated that it hasn't completed  
            return;  
        }  
        // AbstractAuthenticationProcessingFilter.this.sessionStrategy:
        // - 类型 SessionAuthenticationStrategy, 允许插件式地处理认证后关于Session相关的逻辑
        // - 默认初值 new NullAuthenticatedSessionStrategy(), 不做任何处理
        this.sessionStrategy.onAuthentication(authenticationResult, request, response);  
        // Authentication success  
        // AbstractAuthenticationProcessingFilter.this.continueChainBeforeSuccessfulAuthentication:
        // - 默认初值 false
        // - 仅支持 set注入
        if (this.continueChainBeforeSuccessfulAuthentication) {  
            chain.doFilter(request, response);  
        }  
        successfulAuthentication(request, response, chain, authenticationResult);  
    }  
    catch (InternalAuthenticationServiceException failed) {  
        ... // logger.error()  
        unsuccessfulAuthentication(request, response, failed);  
    }  
    catch (AuthenticationException ex) {  
        // Authentication failed  
        unsuccessfulAuthentication(request, response, ex);  
    }  
}
```
# 是否需要进行身份认证 -- requiresAuthentication()

```java
/* --------------------------- AbstractAuthenticationProcessingFilter --------------------------- */
protected boolean requiresAuthentication(HttpServletRequest request, HttpServletResponse response) {  

    if (this.requiresAuthenticationRequestMatcher.matches(request)) {  
        return true;  
    }  

    ... // logger.trace()

    return false;  
}
```

# 认证 -- attemptAuthentication()

再看
```java
public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {  
	// 该过滤器只处理POST请求
	// this.postOnly: 默认true, 支持setPostOnly()方法
    if (this.postOnly && !request.getMethod().equals("POST")) {  
        throw new AuthenticationServiceException(...);  
    }  
    // 获取请求中键为UsernamePasswordAuthenticationFilter#this.usernameParameter的参数
    // UsernamePasswordAuthenticationFilter#this.usernameParameter: 默认为 "username", 支持setUsernameParameter()方法
    String username = obtainUsername(request);  
    username = (username != null) ? username.trim() : "";  
    // 获取请求中键为UsernamePasswordAuthenticationFilter#this.passwordParameter的参数
    // UsernamePasswordAuthenticationFilter#this.passwordParameter: 默认为 "password", 支持setPasswordParameter()方法
    String password = obtainPassword(request);  
    password = (password != null) ? password : ""; 

	// 利用 username 和 password 创建未认证的的AuthenticationToken
	// username -> principal
	// password -> credentials
    UsernamePasswordAuthenticationToken authRequest = 
    UsernamePasswordAuthenticationToken.unauthenticated(
	    username,  
        password
    );

	// -----------------------------------------------------------------------------
	// ----------------------------- 非常重要的一行代码 -----------------------------
	// -----------------------------------------------------------------------------
	// 设置 AuthenticationToken 的 details属性
    // Allow subclasses to set the "details" property  
    setDetails(request, authRequest);  

	// 
    return this.getAuthenticationManager().authenticate(authRequest);  
}
```
