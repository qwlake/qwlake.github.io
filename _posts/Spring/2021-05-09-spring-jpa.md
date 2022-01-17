---
layout: post
title: "[Spring] JPA"
author: qwlake
categories: Spring
tags: jpa
---

# JPA

[김영한님 Jpa 책 정리 블로그](https://parkhyeokjin.github.io/category/jpa.html)

- 프록시( proxy )와 지연로딩

    [[Spring JPA] 프록시( proxy )와 지연로딩](https://victorydntmd.tistory.com/210)

    > 즉시 로딩은 어떤 엔티티를 조회 했을 때 그 엔티티와 연관된 엔티티가 join이 일어나서 같이 조회 되는 것이고,
    지연 로딩은 엔티티가 실제로 사용되는 시점까지 기다리다가 그제서야 엔티티 조회되는 것을 의미합니다.

- 영속성 (Persistence Context)

    [(JPA) Entity와 EntityManager와 EntityManagerFactory](https://perfectacle.github.io/2018/01/14/jpa-entity-manager-factory/)

    - EntityManagerFactory & EntityManager

        EntityManagerFactory: 생성 비용이 매우 크나 Thread-safe하다.

        EntityManager: 생성 비용이 작으나 Thread-safe하지 않다.

        EntityManagerFactory를 하나 생성하고 이를 통해 EntityManager를 여러 개 생성한다.

    - 영속성 컨텍스트 특징
        - 1차 캐시

            영속성 컨텍스트는 내부에 캐시를 갖고 있는데 이게 1차 캐시

            엔티티 생성 후 EM에 persist() 하게 되면 이 영역에 저장됨

            em.find()를 통해 데이터 조회 시 1차 캐시에서 먼저 찾고 없으면 DB에서 찾은 후 1차 캐시에 보관 후 반환

        - 동일성 보장

            아래 코드를 통해 얻은 두 엔티티 객체는 동일한 객체이다.

            ```java
            Member member1 = em.find(Member.class, "jpa1");
            Member member2 = em.find(Member.class, "jpa1");
            System.out.println(member1 == member2); //결과 == true
            ```

        - 쓰기지연
            1. 트랜잭션이 커밋되기 직전까지 모든 쿼리문은 연속성 콘텍스트 내부의 쓰기 지연 SQL 저장소에 저장된다.
            2. 트랜잭션이 커밋되는 순간 모든 쿼리가 한 번에 날아간다.
            3. 만약 롤백을 해야 하는 상황이 발생하면 이후 쿼리는 더이상 날리지 않는다.
        - 변경감지 (Dirty Checking)

            [JPA 변경 감지와 스프링 데이터](https://medium.com/@SlackBeck/jpa-%EB%B3%80%EA%B2%BD-%EA%B0%90%EC%A7%80%EC%99%80-%EC%8A%A4%ED%94%84%EB%A7%81-%EB%8D%B0%EC%9D%B4%ED%84%B0-2e01ad594b82)

            - 스프링 프레임워크 같은 컨테이너 환경에서 사용하는 경우 엔티티 매니저가 여러 개 존재할 수 있고, 변경 감지는 엔티티 매니저별로 수행한다.
            - 단, 같은 스레드(Thread)에서 스프링 데이터가 제공하는 레포지토리들은 하나의 엔티티 매니저를 공유한다. ?
            - 엔티티 매니저의 DB 반영시점: 1. 트랜잭션 커밋 2. 엔티티 매니저의 flush() 호출
        - 플러시

            플러시는 영속성 컨텍스트 영역의 엔티티를 데이터베이스에 반영(commit) 하는 명령이다. 플러시 하는 방법은 아래 3가지로 사용 할 수 있다.

            - em.flush() 직접 호출(거의 사용하지 않음.)
            - tx.commit() 트랜잭션 커밋 시 플러시(flush) 자동 호출
            - JPQL 쿼리 실행 시 플러시(flush) 자동 호출
            - 플러시 모드 옵션
                - FlushModeType.AUTO : 커밋이나 쿼리를 실행할 때 플러시 실행
                - FlushModeType.COMMIT : 커밋 실행시에만 플러시 실행

- 프록시 (Proxy)

    - 지연로딩이란

        엔티티 조회시 프록시 객체가 조회됨. 해당 객체로부터 값을 꺼내 사용할 때 쿼리 조회 시작. 연관된 객체가 존재할 경우 따로따로 SELECT함

    - 즉시로딩이란

        조회에 필요한 모든 테이블을 left outer join 하여 SELECT 쿼리 즉시 실행. (외래키가 null이 허용될 때)

        하지만 `@JoinColumn(name="category_no", nullable=false)` 처럼 nullable을 false로 설정하면 inner join으로 동작한다.

- 프록시 사용 방법

    ```java
    public void findMemberAndTeacher(String id){
        Member member = em.find(Member.class, id);  //즉시 로딩
    }
        
    public void findMemberAndTeacher(String id){
        Member member = em.getReference(Member.class, id);  //프록시 엔티티 반환(로딩 X)
        System.out.println("회원 이름 : " + member.getName());  //실제 사용 될때 로딩(SQL 수행)
    }
    ```

    ![image](https://user-images.githubusercontent.com/41278416/130638526-efa18857-39e2-4387-8ef1-6de82b23dc64.png)


- 기본 전략
    - @ManyToOne, @OneToOne : FetchType.EAGER (즉시로딩)
    - @OneToMany, @ManyToMany : FetchType.LAZY (지연로딩)


- N+1 Problem
    - EAGER 조회 시 연관된 테이블 모두 조인하여 가져옴 → N+1 번의 조인 연산 발생
    - LAZY 조회 시 연관된 테이블마다 SELECT 연산 일어남 → N+1 번의 조회 연산 발생