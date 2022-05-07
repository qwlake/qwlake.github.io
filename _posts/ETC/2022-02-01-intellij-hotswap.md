---
layout: post
title: "[IntelliJ] 인텔리제이 핫스왑으로 개발 효율을 높여보자"
author: qwlake
categories: ETC
tags: IntelliJ Spring
---

# 핫스왑이란?

실행중인 app을 중단하지 않고 수정된 파일만을 compile 하여 reload하는 것

# Before use

1. gradle 파일에 `org.springframework.boot:spring-boot-devtools` 추가
2. `Run/Debug Configuration` → `Modify options` → `On 'Update' action` : `Hot swap classes and update trigger file if failed` 선택
3. `Preferences` → `Build, Execution, Deployment` → `Debugger` → `Hotswap` → `Reload classes after compilation` : `Always` 선택

# How to use

- 디버그 모드로 실행
- 파일 수정 후 `Cmd + F10` (Update application)
- *HowSwap limitations* 에 위배되지 않을 경우
  - app을 중단하지 않고 변경된 파일만 recompile한 후에 reload함
- 위배될 경우
  - app을 새로 다시 띄운다

>### HotSwap limitations
>
>Due to VM design, HotSwap has the following limitations:
>
>- it is only available if a method body is modified. Changing signatures is not supported.
>- adding and removing class members is not supported
>- if the modified method is already in the call stack, the changes will take effect only after the program exits the modified method. Until that moment, the method body remains unchanged, and the frame is marked as **obsolete**.
>
>If you want to remove the limitations imposed by the standard VM, you can use the Dynamic Code Evolution VM with unlimited support for reloading classes at runtime.

# 참고

[https://www.jetbrains.com/help/idea/updating-applications-on-application-servers.html](https://www.jetbrains.com/help/idea/updating-applications-on-application-servers.html)