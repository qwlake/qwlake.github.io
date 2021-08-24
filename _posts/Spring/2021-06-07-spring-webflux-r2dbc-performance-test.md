---
layout: post
title: "[Spring] Spring webflux(w/r2dbc) performance test"
subtitle: "webflux, tomcat, jdbc, r2dbc 를 교차로 조합하여 nGrinder로 성능을 비교한다"
author: qwlake
categories: Spring
tags: webflux r2dbc tomcat jdbc nGrinder
---

# 테스트

읽기 / 쓰기 모두 1k / 10k / 30k / 50k / 70k / 100k 건 에 대해 각각 총 소요시간 측정 (vuser: 10)

- 성능 테스트 대상
    - 읽기 비교 :
        - 단순 데이터 1건 조회
        - 데이터 30건 조회
    - 쓰기 비교 : 단순 데이터 1건 쓰기
- 비교 대상 군
    - Spring Boot w/Tomcat + JDBC
    - Spring Boot w/Tomcat + R2DBC
    - Spring Webflux + JDBC
    - Spring Webflux + R2DBC
    - Spring Boot w/Tomcat + 큐를 활용한 파일 쓰기

# 환경

- WAS, DB(docker mariadb:10.5.9), nGrinder(docker nGrinder:3.5.5-p) 모두 로컬에서 동작
- PC 사양:
    - cpu: 6core/12thread | 2.90GHz up to 4.30GHz
    - memory: 16Gb
- docker:
    - wsl2 engine 사용
    - 별도의 컴퓨팅 자원 제한 없음
- nGrinder:

    설정은 아래 사진처럼 사용. 테스트 스크립트와 Run Count만 바꿔가며 테스트 진행.

    ![image](https://user-images.githubusercontent.com/41278416/130641888-9850e695-86c5-451b-a3d2-047218a6b1ad.png)


# 결과

## 각주

![image](https://user-images.githubusercontent.com/41278416/130641957-6cde11b4-c0c5-4910-8fee-d7700ca07ab7.png)


가로축: 읽기/쓰기 요청 건 수

세로축: 소요시간 (lower is good)

### 단순 데이터 1건 조회 (특정 id에 해당하는 데이터 조회)

![image](https://user-images.githubusercontent.com/41278416/130642030-e913eaa5-ae51-4eb9-9dfa-2b88843c9cc3.png)


### 데이터 30건 조회 (테이블의 모든 데이터 조회)

![image](https://user-images.githubusercontent.com/41278416/130642114-a443d0ca-e612-4d5e-aed1-b3370227bfc3.png)


### 단순 데이터 1건 쓰기

![image](https://user-images.githubusercontent.com/41278416/130642170-5de5f95e-d36a-46ec-82bc-f708a4ca3cec.png)


- raw data

    단위: 초(s)

    - Spring Boot w/Tomcat + JDBC
        - 단순 데이터 1건 조회: 1k: 6 / 10k: 11 / 30k: 22 / 50k: 33 / 70k: 45 / 100k: 77
        - 데이터 30건 조회: 1k: 5 / 10k: 10 / 30k: 28 / 50k: 39 / 70k: 53 / 100k: 102
        - 단순 데이터 1건 쓰기: 1k: 4 / 10k: 20 / 30k: 46 / 50k: 71 / 70k: 101 / 100k: 135
    - Spring Boot w/Tomcat + R2DBC
        - 단순 데이터 1건 조회: 1k: 5 / 10k: 7 / 30k: 15 / 50k: 21 / 70k: 27 / 100k: 39
        - 데이터 30건 조회: 1k: 7 / 10k: 15 / 30k: 35 / 50k: 56 / 70k: 83 / 100k: 147
        - 단순 데이터 1건 쓰기: 1k: 6 / 10k: 15 / 30k: 35 / 50k: 56 / 70k: 77 / 100k: 105
    - Spring Webflux + JDBC
        - 단순 데이터 1건 조회: 1k: 5 / 10k: 13 / 30k: 26 / 50k: 36 / 70k: 52 / 100k: 65
        - 데이터 30건 조회: 1k: 5 / 10k: 11 / 30k: 28 / 50k: 39 / 70k: 54 / 100k: 91
        - 단순 데이터 1건 쓰기: 1k: 5 / 10k: 19 / 30k: 47 / 50k: 73 / 70k: 98 / 100k: 133
    - Spring Webflux + R2DBC
        - 단순 데이터 1건 조회 1k: 5 / 10k: 9 / 30k: 17 / 50k: 25 / 70k: 33 / 100k: 45
        - 데이터 30건 조회: 1k: 5 / 10k: 16 / 30k: 38 / 50k: 63 / 70k: 91 / 100k: 143
        - 단순 데이터 1건 쓰기: 1k: 5 / 10k: 15 / 30k: 37 / 50k: 59 / 70k: 77 / 100k: 111
    - Spring Boot w/Tomcat + 큐를 활용한 파일 쓰기
        - 단순 데이터 1건 쓰기: 1k: 5 / 10k: 8 / 30k: 16 / 50k: 21 / 70k: 29 / 100k: 39