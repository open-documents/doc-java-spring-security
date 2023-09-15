

```java
public final class AnyRequestMatcher implements RequestMatcher {  
	public static final RequestMatcher INSTANCE = new AnyRequestMatcher();  
    private AnyRequestMatcher() {}  
    @Override  
    public boolean matches(HttpServletRequest request) {  
        return true;  
    }
}
```

