---
layout: post
title: "[Spring] Spring Boot 에서 gRPC로 크롤링 MS 호출하기"
subtitle: "IntelliJ 설치부터 스프링부트 개념과 도커를 통한 실행까지"
author: qwlake
categories: Spring
tags: Spring gRPC Docker
---

# IntelliJ

[https://freehoon.tistory.com/147](https://freehoon.tistory.com/147)

# Spring Boot 시작하기

[https://start.spring.io/](https://start.spring.io/)

[https://spring.io/guides/gs/spring-boot/](https://spring.io/guides/gs/spring-boot/)

### actuator

[https://jeong-pro.tistory.com/160](https://jeong-pro.tistory.com/160)

# To Docker

[https://spring.io/guides/gs/spring-boot-docker/](https://spring.io/guides/gs/spring-boot-docker/)

[https://medium.com/@gaemi/spring-boot-과-docker-with-jib-657d32a6b1f0](https://medium.com/@gaemi/spring-boot-%EA%B3%BC-docker-with-jib-657d32a6b1f0)

### Dockerfile 빌드

```bash
mkdir -p build/dependency && (cd build/dependency; jar -xf ../libs/*.jar)
```

```bash
$ docker build -t spring-server .
```

### Build a Docker Image with Gradle

**build image**

```bash
gradlew bootBuildImage --imageName=spring-server
```

**run**

```bash
docker run -p 8080:8080 -t spring-server
```

**run with environment**

```bash
docker run -e "SPRING_PROFILES_ACTIVE=prod" -p 8080:8080 -t spring-server

or

docker run -e "SPRING_PROFILES_ACTIVE=dev" -p 8080:8080 -t spring-server
```

**debug**

```bash
docker run -e "JAVA_TOOL_OPTIONS=-agentlib:jdwp=transport=dt_socket,address=5005,server=y,suspend=n" -p 8080:8080 -p 5005:5005 -t spring-server
```

#### Push to docker hub

```bash
docker tag spring-server qwlake/spring-server:{tagname}
docker push qwlake/spring-server:{tagname}
```

#### Pull from docker hub

```bash
docker pull qwlake/spring-server:tagname
```

### What java.security.egd option is for?

[https://stackoverflow.com/questions/58991966/what-java-security-egd-option-is-for](https://stackoverflow.com/questions/58991966/what-java-security-egd-option-is-for)

# Database

```bash
docker-compose -f db.yml up
```

[https://www.javaguides.net/2019/01/springboot-postgresql-jpa-hibernate-crud-restful-api-tutorial.html](https://www.javaguides.net/2019/01/springboot-postgresql-jpa-hibernate-crud-restful-api-tutorial.html)

# Issue

Consider defining a bean named 'entityManagerFactory' in your configuration.

# Spring Boot (개념 참고)

**Spring MVC 구조의 처리 과정을 설명해보시오. (MVC process)**

[https://jeong-pro.tistory.com/96](https://jeong-pro.tistory.com/96)

**Spring singleton pattern**

[https://medium.com/webeveloper/싱글턴-패턴-singleton-pattern-db75ed29c36](https://medium.com/webeveloper/%EC%8B%B1%EA%B8%80%ED%84%B4-%ED%8C%A8%ED%84%B4-singleton-pattern-db75ed29c36)

**IoC Inversion of Control**

객체 생성 삭제 등에 관한 모든 권한을 스프링에 넘기고 난 필요한 것만 호출

**DI Dependency Injection**

싱글톤 패턴으로 생성된 빈 객체를 주입 → autowired로 가져와 사용

**Dispatcher-Servlet**