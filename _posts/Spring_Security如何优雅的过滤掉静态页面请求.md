---
title: Spring Security 如何优雅的过滤掉静态页面请求
layout: post
category: Spring
date: 2020-06-30 18:00:00
---

>在最近的开发过程中遇到一个问题，某个java服务采用spring security开启了认证，但是同时又需要采用swagger生成在线api文档，如果从这些请求中过滤掉swagger的请求成了一个问题。

想到的方法一,用正则过滤：
```
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable();
        http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);
        http.authorizeRequests()
                .antMatchers("/xxx/!?(doc.html|v2/api-docs|springfox.js|swagger-ui.html|swagger-resources|webjars)**").authenticated();
        http.addFilterBefore(authenticationFilter(), UsernamePasswordAuthenticationFilter.class);
        http.exceptionHandling().authenticationEntryPoint(new ComIdAuthenticationEntryPoint());
    }
    // ...
}
```

更优雅的方法二，把Swagger的请求当做静态页面请求处理：
```
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    private List<String> PERMIT_URL_LIST = ImmutableList.of(
            "/xxx/doc.html*",
            "/xxx/v2/api-docs/**",
            "/xxx/springfox.js*",
            "/xxx/swagger-ui.html*",
            "/xxx/swagger-resources/**",
            "/xxx/webjars/**");

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable();
        http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);
        http.authorizeRequests()
                .antMatchers("/xxx/**").authenticated();
        http.addFilterBefore(authenticationFilter(), UsernamePasswordAuthenticationFilter.class);
        http.exceptionHandling().authenticationEntryPoint(new ComIdAuthenticationEntryPoint());
    }

    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers(PERMIT_URL_LIST.toArray(new String[0]));
    }
    // ...
}
```
