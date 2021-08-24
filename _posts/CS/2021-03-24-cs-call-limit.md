---
layout: post
title: "[CS] API Call limit"
subtitle: "api call limit 구현을 위한 여러 접근방법과 데이터 조작 방법"
author: qwlake
categories: CS
tags: -
---

# 자료조사
- Techniques for enforcing rate limits (비율 제한을 적용하는 기법)
    - **토큰 버킷**: [토큰 버킷](https://wikipedia.org/wiki/Token_bucket)은 롤링 및 누적 사용 예산을 *토큰*의 잔액으로 설정합니다. 이 기법은 서비스에 대한 모든 입력이 요청과 1:1로 대응하지는 않는다는 점을 전제로 합니다. 토큰 버킷은 토큰을 일정한 비율로 추가합니다. 서비스 요청이 이루어지면 서비스는 요청을 이행하기 위해 토큰을 인출하려고 합니다(토큰 수 감소). 버킷에 토큰이 없으면 서비스가 한계에 도달하고 백프레셔로 응답합니다. 예를 들면 GraphQL 서비스에서 단일 요청으로 인해 여러 작업이 하나의 결과로 구성될 수 있습니다. 이러한 작업은 각각 하나의 토큰을 사용할 수 있습니다. 이러한 방식으로 서비스는 비율 제한 기법을 요청에 직접 연결하는 대신 사용을 제한하는 데 필요한 용량을 추적할 수 있습니다.
    - **Leaky Bucket**: [Leaky Bucket](https://wikipedia.org/wiki/Leaky_bucket)은 토큰 버킷과 비슷하지만 비율이 버킷에서 떨어지거나 새는 양으로 제한됩니다. 이 기법은 시스템이 서비스를 수행할 수 있을 때까지 요청을 보유할 수 있는 어느 정도의 유한한 용량을 가지고 있음을 전제로 합니다. 여분은 단순히 경계를 넘어 흘러넘치고 삭제됩니다. 이러한 버퍼 용량 개념(Leacky Bucket의 사용이 필수는 아님)은 부하 분산기 및 디스크 I/O 버퍼처럼 서비스에 인접한 구성요소에도 적용됩니다.
    - **고정 기간**: 고정 기간 제한(예: 시간당 3,000개의 요청 또는 하루에 10개의 요청)은 쉽게 지정할 수 있지만, 기간의 경계에서 사용 가능한 할당량 재설정으로 급증할 수 있습니다. 예를 들어 시간당 3,000개의 요청 제한이 있는 경우, 한 시간의 첫 1분 동안 3,000개의 요청이 모두 발생할 수 있도록 허용하면 서비스에 문제가 발생할 수 있습니다.
    - **슬라이딩 기간**: 슬라이딩 기간은 고정 기간의 이점이 있지만, 롤링 기간은 집중을 완화합니다. Redis와 같은 시스템은 만료 키로 이 기법을 지원합니다.

[Rate-limiting strategies and techniques | Solutions | Google Cloud](https://cloud.google.com/solutions/rate-limiting-strategies-techniques#techniques-enforcing-rate-limits)

[서비스 가용성 확보에 필요한 Rate Limiting Algorithm에 대해 | Mimul Tech log](https://www.mimul.com/blog/about-rate-limit-algorithm/)

## Leaky Bucket

[Rate Limiting with NGINX and NGINX Plus](https://www.nginx.com/blog/rate-limiting-nginx/)

## Token Bucket

[vladimir-bukhtoyarov/bucket4j](https://github.com/vladimir-bukhtoyarov/bucket4j)

[Bucket4j - Token bucket 알고리즘을 이용한 Rate limit 라이브러리를 알아보자!!](https://baeji77.github.io/dev/java/Java-rate-limit-bukcet4j/#bucket4j)

[Rate limiting Spring MVC endpoints with bucket4j](https://golb.hplar.ch/2019/08/rate-limit-bucket4j.html)

## Redis

[Redis rate limiter in Spring Boot](https://www.javacodemonk.com/redis-rate-limiter-in-spring-boot-6455a677)

[Redis transaction (MULTI, EXEC)](https://redis.io/topics/transactions)

[고 처리량 분산 비율 제한기 - LINE ENGINEERING](https://engineering.linecorp.com/ko/blog/high-throughput-distributed-rate-limiter/)

## Lock

[자바의 락(Locks in Java)](https://parkcheolu.tistory.com/m/24)

[Java Concurrency Utilities Tutorials - java.util.concurrent](http://tutorials.jenkov.com/java-util-concurrent/index.html)

[Java Spin lock implementation](https://www.tech693.com/2018/08/java-spin-lock-implementation.html?m=1)

[Using a Mutex Object in Java | Baeldung](https://www.baeldung.com/java-mutex)

## Actor

[Alpakka Documentation](https://doc.akka.io/docs/alpakka/current/spring-web.html)

# 작업내역

1. bucket4j
    - in-memory 기반 call limit
2. redis
    1. key, value를 저장하면서 expire time을 1분으로 지정한다.
        - key → ip:mm (mm은 현재 분(minute))
        - value → count
3. mutex
    1. spin-lock (V1)
        - ReentrantLock 상속받아 구현
        - lock() 호출시 다른 곳에서 락이 걸려 있으면 while 루프를 돌며 대기 → 락이 풀리면 (!isLocked()) lock()을 호출한 스레드에서 lock을 선점
    2. syncronized (V2)
        - 메모리를 조작하는 구문을 syncronized로 묶어서 사용
    3. call-counter (V1, V2 공통)

        ip별로 다른 call count를 세기 위해서 CallCounter 클래스 사용

        - CallCounter는 `Map<String, List<?>>` 형식의 변수(이하 userCounts)를 필드로 갖는다
            - userCounts key: {IP}:{mm}형식. mm은 현재 분(minute)
            - userCounts value: (count, expire time) 형식. List, 크기 2 고정
                - 지금 보니 List가 아니라 객체에 데이터 담아서 사용해도 좋았을듯
        - CallCounter는 Scheduled 함수를 갖는다; removeCountAfterOneMinute
            - removeCountAfterOneMinute 함수는 1초마다 userCounts에 저장된 모든 데이터를 순회하면서 expire time을 꺼내 현재 시간과 비교한다.
            - expire time에서 60초가 지났으면 해당 key, value는 remove

# 구현코드
https://github.com/qwlake/oscar/tree/main/api-call-limit