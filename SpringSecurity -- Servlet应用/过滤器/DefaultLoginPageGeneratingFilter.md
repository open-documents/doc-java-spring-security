
```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)  
      throws IOException, ServletException {  
    doFilter((HttpServletRequest) request, (HttpServletResponse) response, chain);  
}  
  
private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)     throws IOException, ServletException {  

    boolean loginError = isErrorPage(request);  

    boolean logoutSuccess = isLogoutSuccess(request);  

    if (isLoginUrlRequest(request) || loginError || logoutSuccess) {  
        String loginPageHtml = generateLoginPageHtml(request, loginError, logoutSuccess);  
        response.setContentType("text/html;charset=UTF-8");  
        response.setContentLength(loginPageHtml.getBytes(StandardCharsets.UTF_8).length);  
        response.getWriter().write(loginPageHtml);  
        return; 
	}  

    chain.doFilter(request, response);  
}
```