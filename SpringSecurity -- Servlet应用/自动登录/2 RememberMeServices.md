
# 身份认证成功
```java
/* ------------------------------- AbstractRememberMeServices ------------------------------- */
public void loginSuccess(HttpServletRequest request, HttpServletResponse response, Authentication successfulAuthentication) {  

	// 
    if (!rememberMeRequested(request, this.parameter)) {  
        ... // logger.debug()  
        return;  
	}  

	// protected abstract void 方法
    onLoginSuccess(request, response, successfulAuthentication);  
}
```

## 判断是否是rememberMe 

```java
/* ------------------------------- AbstractRememberMeServices ------------------------------- */
protected boolean rememberMeRequested(HttpServletRequest request, String parameter) {  
	// AbstractRememberMeServices.this.alwaysRemember:
	// - 默认初值 null
	// - 仅支持 set注入
    if (this.alwaysRemember) {  
        return true;  
    }  

    String paramValue = request.getParameter(parameter);  
    if (paramValue != null) {  
	    if (paramValue.equalsIgnoreCase("true") 
            || paramValue.equalsIgnoreCase("on")  
            || paramValue.equalsIgnoreCase("yes") 
            || paramValue.equals("1")) {  
            return true;  
        }  
    }  

    ... // logger.debug(...);  

	return false;
}
```

# 身份认证成功代表登录成功

```java
protected void onLoginSuccess(HttpServletRequest request, HttpServletResponse response, Authentication successfulAuthentication) {  

    String username = successfulAuthentication.getName();  

    ... // logger.debug()  

	// PersistentRememberMeToken: 对四个属性的简单封装(提供get方法)
	// - username(String)
	// - series(String)
	// - tokenValue(String)
	// - date(Date)
    PersistentRememberMeToken persistentToken = new PersistentRememberMeToken(
	    username, 
	    generateSeriesData(),  
        generateTokenData(),
		new Date()
	);  

    try {  
	    // PersistentTokenRepository#createNewToken(): 向自身添加Token
	    // - JdbcTokenRepositoryImpl     向数据库中添加PersistentRememberMeToken信息
	    // - InMemoryTokenRepositoryImpl 向内存中添加PersistentRememberMeToken信息
        this.tokenRepository.createNewToken(persistentToken);  
        addCookie(persistentToken, request, response);  
    }  
    catch (Exception ex) {  
        ... // logger.error()  
    }  
}
```

## 创建 PersistentRememberMeToken

1）
```java
protected String generateSeriesData() {  
    byte[] newSeries = new byte[this.seriesLength];  
    this.random.nextBytes(newSeries);  
    return new String(Base64.getEncoder().encode(newSeries));  
}
```
2）
```java
protected String generateTokenData() {  
    byte[] newToken = new byte[this.tokenLength];  
    this.random.nextBytes(newToken);  
    return new String(Base64.getEncoder().encode(newToken));  
}
```

# PersistentTokenRepository向自身添加Token信息

# 向响应（response）中添加Cookie

```java
private void addCookie(PersistentRememberMeToken token, HttpServletRequest request, HttpServletResponse response) {  
    setCookie(
	    new String[] {
			token.getSeries(), 
		    token.getTokenValue()
		}, 
		getTokenValiditySeconds(), 
		request,  
        response
    );  
}
protected void setCookie(String[] tokens, int maxAge, HttpServletRequest request, HttpServletResponse response) {  

    String cookieValue = encodeCookie(tokens);  
    Cookie cookie = new Cookie(this.cookieName, cookieValue); 

	// 
    cookie.setMaxAge(maxAge);  
    // 
    cookie.setPath(getCookiePath(request));  
    // 
    if (this.cookieDomain != null) {  
        cookie.setDomain(this.cookieDomain);  
    }  
    // 
    if (maxAge < 1) {  
        cookie.setVersion(1);  
    }  
    // 
    cookie.setSecure((this.useSecureCookie != null) ? this.useSecureCookie : request.isSecure()); 
    //  
    cookie.setHttpOnly(true);  

    response.addCookie(cookie);  
}
```