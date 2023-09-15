
导入依赖：
```xml

```

SpringSecurity提供了应用业务安全的2个方面：
- 认证（Authentication）
- 授权（Authorization）


# 版本更新

Spring Security 5.0 版本

1）弃用了 `WebSecurityConfigurerAdapter`，主要原因（来自大模型回答）如下：

1. 重复配置：`WebSecurityConfigurerAdapter` 提供了很多默认的配置，但在很多情况下，用户需要重写这些默认配置。这使得 `WebSecurityConfigurerAdapter` 变得冗余，因为用户需要了解每个方法默认的配置是什么，然后根据需要进行修改。
2. 不符合 Spring Boot 的约定优于配置（Convention over Configuration）原则：Spring Boot 的设计理念是约定优于配置，尽可能地减少用户的配置。而 `WebSecurityConfigurerAdapter` 需要用户去重写很多方法，与这个设计理念相悖。
3. 配置的不一致性：由于 `WebSecurityConfigurerAdapter` 提供了太多的方法，使得每个方法的默认行为难以被用户所理解。这导致在升级 Spring Security 版本时，用户可能会遇到配置不一致的问题。

因此，Spring Security 5.0 版本弃用了 `WebSecurityConfigurerAdapter`，推荐使用更具体、更简洁的 `SecurityFilterChain` 进行配置。同时，Spring Security 也提供了一些更高级的注解，如 `@EnableWebSecurity` 和 `@AuthenticationPrincipal` 等，使得配置更加简洁和灵活。