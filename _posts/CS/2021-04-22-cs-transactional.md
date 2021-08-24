---
layout: post
title: "[CS] Transactional"
author: qwlake
categories: CS
tags: transaction
---

# Transactional

Transactional을 구현하려면 정의된 비즈니스 로직 앞 뒤에 트랜잭션의 시작(begin)과 끝(commit)을 추가해 주어야 한다. 이를 Spring에선 AOP로 구현하고 있으며, 일반적인 AOP 방법은 2가지.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dd4b30d2-0c7c-430c-82f8-e3ac216f97fb/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dd4b30d2-0c7c-430c-82f8-e3ac216f97fb/Untitled.png)

# AOP 구현 방법

## JDK Proxy (Interface based)

- `spring.aop.proxy-target-class=true`
- 프록시 객체를 생성하고 이 객체로 우리가 만든 메서드를 감싸서 실행한다.
- Spring의 기본 동작 방식이다.
1. 인터페이스가 구현된 클래스에만 사용할 수 있다.
2. 대상 클래스가 thread-safe한 경우 생성된 프록시도 thread-safe하다.
3. Advice/Pointcut 및 TargetSource가 직렬화 가능하다면 프록시도 직렬화가 가능하다.

## CGLib Proxy (class based)

- `spring.aop.proxy-target-class=false`
- CGLIB는 byte code를 조작하여 코드를 생성하는 기법이다.
- Spring Boot의 기본 동작 방식이다.
1. CGLIB도 Dynamic Proxy와 마찬가지로 원본 클래스의 thread-safe 특징을 따른다.
2. 부모 인터페이스가 없는 클래스에도 적용할 수 있다.
3. 상속을 활용한다. (상속이 불가능한 final/private 키워드는 aspect가 적용되지 않는다)

## 사용 예

Transactional 태그가 붙은 메서드를 실행할 때 RuntimeException이 발생한다면 해당 메서드에서 실행하던 모든 작업을 rollback한다.

```kotlin
@Service
class TestService(val urlsRepository: UrlsRepository) {
    @Transactional  // 태그가 없다면 에러가 발생하기 전까지의 작업은 db에 반영된다
    fun tran(): List<UrlsEntity> {
        val urlsEntity1 = UrlsEntity()
        val urlsEntity2 = UrlsEntity()
        urlsRepository.save(urlsEntity1)
        Assert.isTrue(false, "test1")  // 에러를 throw하기 때문에 아무것도 db에 저장되지 않는다
        urlsRepository.save(urlsEntity2)
        return listOf(urlsEntity1, urlsEntity2)
    }
}
```

# 기타 옵션

## readOnly

기본 Isolation Level은 DB의 default 속성을 따라가지만 별도로 명시해줄 수 있다. (`@Transactional(readOnly = true)`)

[`@Transactional(readOnly = true)`에대한 질문입니다. - 인프런 | 질문 & 답변](https://www.inflearn.com/questions/7185)

## propagation behavior

1. `TransactionDefinition.PROPAGATION_REQUIRED`
    1. 트랜잭션에서 트랜잭션을 또 호출할때 부모 트랜잭션을 활용
    2. 자식 트랜잭션에서 에러가 발생할 경우, `roll-back only` 를 true로 설정하고, 부모 트랜잭션에서 커밋하려고 할 때 `roll-back only` 가 마킹되어 있으므로 부모 트랜잭션도 롤백됨
    3. 자식 트랜잭션을 열 때 사용한 `TransactionDefinition` 이 적용되지 않음
        - ex. `ISOLATION_SERIALIZABLE`
2. `TransactionDefinition.PROPAGATION_REQUIRES_NEW`
    1. 자식 트랜잭션을 열 때 완전히 새로운 트랜잭션을 새로 만든다.
    2. 단점으로 connection pool의 connection을 한 개 더 차지한다.
    3. 단점으로 두 트랜잭션 사이에 데드락이 발생할 수 있다. 
    4. 단점으로 두 트랜잭션은 entity manage를 공유하지 않기 때문에 persistence context 역시 공유하지 않고, 이는 쿼리 실행의 비효율을 야기할 수 있다.

---

# 참고

[[JPA] 트랜잭션 테스트](https://lng1982.tistory.com/299)

[JDK- and CGLIB-based proxies](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-pfb-proxy-types)

[전체 정리](https://kwonnam.pe.kr/wiki/springframework/transaction)

[응? 이게 왜 롤백되는거지? - 참여 중인 트랜잭션이 실패하면 기본정책이 전역롤백](https://woowabros.github.io/experience/2019/01/29/exception-in-transaction.html)

[Spring Transaction 사용 시 주의할 점](https://suhwan.dev/2020/01/16/spring-transaction-common-mistakes/)