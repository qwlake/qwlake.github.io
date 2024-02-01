---
layout: post
title: "[TroubleShooting] Complete Service Outage"
author: qwlake
categories: ETC
tags: TroubleShooting Firewall Spring RestTemplate
---

# **상황**

어느날 백엔드 api의 response가 모두 404로 나가는 장애가 보고됨

# **문제점**

모든 api가 정상동작 하지 않으므로 서비스 전체 먹통

# **내가 생각했던 원인**

* 특정 api 호출시 간헐적으로 무기한 대기에 빠짐

  * 에러 로그에서 A사를 호출하는 rest template 코드를 공통적으로 발견했기 때문
* 특정 api 호출이 백엔드 시스템 전체로 영향을 끼친 이유

  1. rest template의 pool request timeout은 적용되어 있지 않음
  2. 일부 스레드가 http client pool에 있는 자원을 모두 점유중이라면 다른 스레드는 http client pool의 자원을 얻기 위해 무기한 대기
  3. tomcat thread 모두가 http client pool 자원을 얻기 위해 대기하게 되고 더이상 요청을 처리할 수 있는 thread가 남지 않음

# **원인 검증**

미봉책으로 rest template에 connection TTL을 설정한 후 상황이 해결됨

# **결과**

- 3일 뒤 다른 팀원이 근본적인 문제로 방화벽 문제를 제기
- 이를 정보보호팀에서 확인한 결과 도메인에 연결된 아이피가 변경될 경우 발생하는 이슈로 확인됨
- 방화벽 설정을 변경하여 근본적인 문제 해결
