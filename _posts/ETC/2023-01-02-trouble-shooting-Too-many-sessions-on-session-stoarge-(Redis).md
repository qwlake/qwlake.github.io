---
layout: post
title: "[TroubleShooting] Too many sessions on session stoarge (Redis)"
author: qwlake
categories: ETC
tags: Session Redis TroubleShooting
---
# **상황**

어느날 프론트팀의 배포 이후 서버 로그에서 세션 수가 과도하게 많다는 warnning 발생

# **문제점**

세션 스토리지(Redis)의 메모리 사용량 지속적 증가

# **내가 생각했던 원인**

토큰 발급 및 검증 관련 api가 과도하게 호출되지 않는지 의심 -> 세션 수가 과도하게 많기 때문

# **원인 검증**

1. 모니터링 툴로 확인한 결과 토큰 관련 api 호출이 배포 이전 대비 최소 5배 이상 증가 확인
2. 자세한 증상 확인을 위해 request ip 체크
3. request ip 대비 토큰 관련 api 호출이 1:10 넘게 차이나고 있는 것을 확인
4. 로그를 통해 1초 내에 동일 ip로 토큰 api를 수십 번 호출한 이력 확인

# **결과**

해당 내용 프론트팀 전달, 이후 약 3시간 뒤 정상화
