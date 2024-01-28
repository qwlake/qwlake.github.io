---
layout: post
title: "[SpringBoot] Api validation using ip and its TestCode"
author: qwlake
categories: Spring
tags: springboot3 junit5 testcode
---
In my previous post, there is `AcceptApiFilter` . It uses `ApiUrlValidationService` . So this post find out how it works and how test it. Testing is so important in this case. Because if someone added controller but didn’t add new controller’s information in validation service, all new requests are would be are denied. TestCode will be executed when pipeline building, so it prevents human errors.

# Codes

```kotlin
@Component
class AcceptApiFilter(
    private val apiUrlValidationService: ApiUrlValidationService,
): OncePerRequestFilter() {

    override fun doFilterInternal(
        request: HttpServletRequest,
        response: HttpServletResponse,
        chain: FilterChain
    ) {
        val path = request.requestURI
        val requestIp = HttpUtil.getClientIpAddress(request)

        if (apiUrlValidationService.apiValidation(path, requestIp)) {
            chain.doFilter(request, response)
        } else {
            throw NotFoundEndpointException("")
        }
    }
}
```

```kotlin
@Service
class ApiUrlValidationService(
    private val availableApiUrls: List<ApiCode>,
    private val internalApiUrls: List<String>,
    private val nonCheckingApiUrls: List<String>,
    private val internalIpAddressMatcherManager: InternalIpAddressMatcherManager,
) {

    @Value("\\${mydata.version}")
    val currentSupportVersion: Int = 0
    @Value("\\${mydata.min-version}")
    val minimumSupportVersion: Int = 0

    companion object {
        private val versionPattern: Pattern = Pattern.compile("/v(\\\\d+)/")
    }

    fun apiValidation(path: String, requestIp: String): Boolean {
        val matcher = AntPathMatcher()

        val availableApiMatch = availableApiUrls.any {
            matcher.match("/**${it.resource}", path)
        }
        if (availableApiMatch) {
            if (!isPossibleVersion(path)) {
                throw NotAllowedVersionException("available version range: $minimumSupportVersion ~ $currentSupportVersion")
            } else {
                return true
            }
        }

        val internalApiMatch = internalApiUrls.any {
            matcher.match("$it/**", path)
        }
        if (internalApiMatch) return internalIpAddressMatcherManager.acceptable(requestIp)

        val nonCheckingApiMatch = nonCheckingApiUrls.any {
            matcher.match("$it/**", path)
        }
        return nonCheckingApiMatch
    }

    private fun isPossibleVersion(path: String): Boolean {
        val version = extractVersion(path) ?: return true
        return version in minimumSupportVersion..currentSupportVersion
    }

    private fun extractVersion(apiPath: String): Int? {
        val versionMatcher: Matcher = versionPattern.matcher(apiPath)
        return if (versionMatcher.find()) {
            versionMatcher.group(1).toInt()
        } else {
            null
        }
    }
}
```

`apiValidation` check first whether request is valid api and valid api-version. And then check whether api-path is internal api. If it is, check request-ip whether internal-ip. Finally check if the api doesn't check anything.
