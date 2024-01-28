---
layout: post
title: "[SpringBoot] SecurityFilterChain"
author: qwlake
categories: Spring
tags: springboot3 security filter
---

# What is SecurityFilterChain?

Filter do some process what you want before Controller. SecurityFilter is do something before other filters. In my case I need some processes in SecurityFilter phase. So all of filters are in SecurityFilterChain. If you don’t need security things, just put your filters in normal steps.

# Codes

```kotlin
@EnableWebSecurity
@EnableGlobalMethodSecurity(securedEnabled = true)
@Configuration(proxyBeanMethods = false)
class AuthorizationServerConfig {

    @Bean
    @Throws(Exception::class)
    fun authorizationServerSecurityFilterChain(
        http: HttpSecurity,
        authenticationSecurityFilter: AuthenticationSecurityFilter,
        exceptionHandleFilter: ExceptionHandleFilter,
        apiTranIdInjectionFilter: ApiTranIdInjectionFilter,
        contentCachingServletFilter: ContentCachingServletFilter,
        dataApiLogFilter: DataApiLogFilter,
        acceptApiFilter: AcceptApiFilter,
    ): SecurityFilterChain {
        http
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()
            .authorizeRequests().anyRequest().permitAll().and()
            .addFilterAfter(authenticationSecurityFilter, BasicAuthenticationFilter::class.java)
            .addFilterBefore(acceptApiFilter, AuthenticationSecurityFilter::class.java)
            .addFilterBefore(apiTranIdInjectionFilter, AcceptApiFilter::class.java)
            .addFilterBefore(exceptionHandleFilter, ApiTranIdInjectionFilter::class.java)
            .addFilterBefore(dataApiLogFilter, ExceptionHandleFilter::class.java)
            .addFilterBefore(contentCachingServletFilter, DataApiLogFilter::class.java)
            .httpBasic().disable()
            .csrf().disable()
            .formLogin().disable()
        return http.build()
    }
}
```

This structure is legacy. So if you use newer version than me, migrate to that. And this code is little hard to see so here is simple version.

```kotlin
BasicAuthenticationFilter -> 
contentCachingServletFilter -> 
dataApiLogFilter -> 
exceptionHandleFilter -> 
apiTranIdInjectionFilter -> 
acceptApiFilter -> 
authenticationSecurityFilter
```

* `BasicAuthenticationFilter` : Default Spring Security filter.
* `contentCachingServletFilter` : Caching request body. Caution that it takes different ways to caching on http Content-type.
* `dataApiLogFilter` : Save requests and response information(like api-path, request-ip, response http status, etc) on database.
* `exceptionHandleFilter` : Handling exceptions in after process.
* `apiTranIdInjectionFilter` : Set some information on response header.
* `acceptApiFilter` : Distinguish whether the api-path can be processed or whether the request came from an IP that matches the api-path.
* `authenticationSecurityFilter` : Process JWT tokens.
