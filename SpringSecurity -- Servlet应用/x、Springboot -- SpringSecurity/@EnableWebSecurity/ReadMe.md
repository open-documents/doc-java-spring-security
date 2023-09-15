
@EnableWebSecurity注解的定义如下。
```java
@Import({ WebSecurityConfiguration.class, 
		  SpringWebMvcImportSelector.class, 
		  OAuth2ImportSelector.class,  
          HttpSecurityConfiguration.class }
)  
@EnableGlobalAuthentication  
public @interface EnableWebSecurity {
	boolean debug() default false;
}
```

和所有@EnableXxx一样，用于导入该框架的基础组件。

在SpringBoot中，如果