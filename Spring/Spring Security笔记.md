# Spring Security笔记

## 概要

Spring 是非常流行和成功的 Java 应用开发框架，Spring Security是 Spring 家族中的 成员。Spring Security 基于 Spring 框架，提供了一套 Web 应用安全性的完整解决方案。 正如你可能知道的关于安全方面的两个主要区域是“认证”和“授权”（或者访问控 制），一般来说，Web 应用的安全性包括用户认证（Authentication）和用户授权 （Authorization）两个部分，这两点也是 Spring Security 重要核心功能。 

- [x] 用户认证指的是：验证某个用户是否为系统中的合法主体，也就是说用户能否访问该系统。用户认证一般要求用户提供用户名和密码。系统通过校验用户名和密码来完成认 证过程。通俗点说就是系统认为用户是否能登录 。
- [x] 用户授权指的是验证某个用户是否有权限执行某个操作。在一个系统中，不同用户 所具有的权限是不同的。比如对一个文件来说，有的用户只能进行读取，而有的用户可以 进行修改。一般来说，系统会为不同的用户分配不同的角色，而每个角色则对应一系列的 权限。通俗点讲就是系统判断用户是否有权限去做某些事情。



###  权限管理中的相关概念

**主体** 

英文单词：principal 

使用系统的用户或设备或从其他系统远程登录的用户等等。简单说就是谁使用系统谁就是主体。 

**认证** 

英文单词：authentication 权限管理系统确认一个主体的身份，允许主体进入系统。简单说就是“主体”证 明自己是谁。 笼统的认为就是以前所做的登录操作。 

**授权**

英文单词：authorization 将操作系统的“权力”“授予”“主体”，这样主体就具备了操作系统中特定功 能的能力。



## Shiro和Spring Security

### Spring Security

**SpringSecurity 特点：** 

1. 和 Spring 无缝整合。 
2. 全面的权限控制。 
3. 专门为 Web 开发而设计。 旧版本不能脱离 Web 环境使用。 新版本对整个框架进行了分层抽取，分成了核心模块和 Web 模块。单独引入核心模块就可以脱离 Web 环境。 
4.  重量级。

### Shiro

Apache 旗下的轻量级权限控制框架。 

特点： 

1.  轻量级。Shiro 主张的理念是把复杂的事情变简单。针对对性能有更高要求 的互联网应用有更好表现。
2. 通用性。 好处：不局限于 Web 环境，可以脱离 Web 环境使用。 缺陷：在 Web 环境下一些特定的需求需要手动编写代码定制。

相对于 Shiro，在 SSM 中整合 Spring Security 都是比较麻烦的操作，所以，Spring Security 虽然功能比 Shiro 强大，但是使用反而没有 Shiro 多（Shiro 虽然功能没有 Spring Security 多，但是对于大部分项目而言，Shiro 也够用了）。 自从有了 Spring Boot 之后，Spring Boot 对于 Spring Security 提供了自动化配置方 案，可以使用更少的配置来使用 Spring Security。 

因此，一般来说，常见的安全管理技术栈的组合是这样的： 

- [x] SSM + Shiro 
- [x] Spring Boot/Spring Cloud + Spring Security 

以上只是一个推荐的组合而已，如果单纯从技术上来说，无论怎么组合，都是可以运行 的。

## 基本原理

SpringSecurity 本质是一个过滤器链： 

启动可以获取到过滤器链： 

- [x] org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter
- [x] org.springframework.security.web.context.SecurityContextPersistenceFilter org.springframework.security.web.header.HeaderWriterFilter
- [x] org.springframework.security.web.csrf.CsrfFilter org.springframework.security.web.authentication.logout.LogoutFilter（对登陆之后进行拦截）
- [x] org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter（对登陆进行拦截，验证账号密码是否正确）
- [x] org.springframework.security.web.authentication.ui.DefaultLoginPageGeneratingFilter
- [x] org.springframework.security.web.authentication.ui.DefaultLogoutPageGeneratingFilter
- [x] org.springframework.security.web.savedrequest.RequestCacheAwareFilter
- [x] org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter
- [x] org.springframework.security.web.authentication.AnonymousAuthenticationFilter
- [x] org.springframework.security.web.session.SessionManagementFilter org.springframework.security.web.access.ExceptionTranslationFilter（异常过滤器）
- [x] org.springframework.security.web.access.intercept.FilterSecurityInterceptor（方法级别的过滤器，位于过滤链的最底端）

如果spring单独使用spring securty，需要在application.yml中配置DelegatingFilterProxy，但是springboot已经帮我们配置好了

## spring security认证

设置登录用户名密码的几种方式

第一种：通过配置文件

第二种：通过配置类（需要继承WebSecurityConfigurerAdapter）

第三种：自定义编写实现类（继承UsernamePasswordAuthenticationFilter类，可以通过数据库进行认证）

## 配置不需要认证的路径

ignore是完全绕过了spring security的所有filter，相当于不走spring security

```
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers("/css/**");
        web.ignoring().antMatchers("/js/**");
        web.ignoring().antMatchers("/fonts/**");
    }
}
```

permitall没有绕过spring security，其中包含了登录的以及匿名的。

```
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .antMatchers("/css/**", "/js/**","/fonts/**").permitAll()
                .anyRequest().authenticated();
    }
}
```























###