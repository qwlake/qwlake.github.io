---
layout: post
title: "[Spring] 내장DB(Embedded mariadb) mariaDB4j 실행 관련 에러 (feat. openssl)"
author: qwlake
categories: Spring
tags: embedded mysql mariadb error
---

# 내장 mariadb 설치

`testImplementation*("ch.vorburger.mariaDB4j:mariaDB4j:2.5.3")`

# 설치 후 실행(Junit test)하는데 에러

**현재 맥 계열(intel, silicon)에서만 발견한 문제이다. (리눅스는 확인 못해봤고, 윈도우에서는 정상동작)**
```
...
dyld[30098]: Library not loaded: /usr/local/opt/openssl/lib/libssl.1.0.0.dylib
  Referenced from: /private/var/folders/09/zcbzjdzj4bv5dclns9v6jzsh0000gn/T/MariaDB4j/base/bin/my_print_defaults
  Reason: tried: '/usr/local/opt/openssl/lib/libssl.1.0.0.dylib' (mach-o file, but is an incompatible architecture (have 'arm64', need 'x86_64')), '/usr/local/lib/libssl.1.0.0.dylib' (no such file), '/usr/lib/libssl.1.0.0.dylib' (no such file), '/opt/homebrew/Cellar/openssl@1.0/1.0.2u/lib/libssl.1.0.0.dylib' (mach-o file, but is an incompatible architecture (have 'arm64', need 'x86_64')), '/usr/local/lib/libssl.1.0.0.dylib' (no such file), '/usr/lib/libssl.1.0.0.dylib' (no such file)
Installing MariaDB/MySQL system tables in '/private/var/folders/09/zcbzjdzj4bv5dclns9v6jzsh0000gn/T/MariaDB4j/data/60735' ...
...
```

# 해결방법

openssl1.0 버전이 다음 위치에 필요 `/usr/local/opt/openssl`

이를 위해선 아래의 명령어를 실행하면 됨

```
brew install rbenv/tap/openssl@1.0
ln -sfn /usr/local/Cellar/openssl@1.0/1.0.2t /usr/local/opt/openssl
```

m1 사용자는 위 명령어에서 에러가 날텐데..

- 에러

    ```
    Last 15 lines from /Users/jungwoo/Library/Logs/Homebrew/openssl@1.0/03.make:
     ^
    x86_64cpuid.s:273:10: error: unknown token in expression
     cmpq $0,%rax
             ^
    x86_64cpuid.s:273:10: error: invalid operand
     cmpq $0,%rax
             ^
    x86_64cpuid.s:274:9: error: unknown token in expression
     cmoveq %rcx,%rax
            ^
    x86_64cpuid.s:274:9: error: invalid operand
     cmoveq %rcx,%rax
            ^
    make[1]: *** [x86_64cpuid.o] Error 1
    make: *** [build_crypto] Error 1
    
    If reporting this issue please do so at (not Homebrew/brew or Homebrew/core):
      https://github.com/rbenv/homebrew-tap/issues
    
    These open issues may also help:
    `brew install rbenv/tap/openssl@1.0` not working anymore https://github.com/rbenv/homebrew-tap/issues/1
    jungwoo@jungwoo-MacBookAir ~ % brew uninstall ./openssl@1.0.rb
    Error: No available formula with the name "./openssl@1.0.rb". Did you mean openssl@1.0?
    ```


openssl1.0 설치 자체는 이 [링크](https://github.com/rbenv/homebrew-tap/pull/2#issuecomment-930218375) 의 방법으로 설치가 가능하지만 mariaDB4j에서는 openssl1.0 뿐만 아니라 x86_64로 설치된 파일을 요구하기 때문에 다른 방법을 찾아야 한다. 

현재 알려진 방법으로는 x86_64 버전의 `libssl.1.0.0.dylib` 를 구해서 `/usr/local/opt/openssl/lib` 위치에 넣는 것이다. 다음의 링크를 참조 바란다. [https://github.com/vorburger/MariaDB4j/issues/411#issuecomment-963983334](https://github.com/vorburger/MariaDB4j/issues/411#issuecomment-963983334)