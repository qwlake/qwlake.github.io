---
layout: post
title: "[Server] Web Server ↔ WAS 관계"
author: qwlake
categories: ETC
tags: Server WAS Web-server Python wsgi Spring
---

# Web Server

## 역할

1. 리버스 프록시
2. 로드 벨런스

## 효과

1. 보안성 향상
2. 성능 향상 - 진짜?

# Python을 서버로 쓰기 위해서는

WSGI가 필요함

- python은 다중 스레드를 원활히 지원하지 않기 때문
- wsgi가 이 부분을 맡아줌

WSGI는 리버스 프록시를 지원하지 않음 → Web Server 필요

- wsgi는 Web Server로부터의 요청을 WAS한테 넘겨준다.
- wsgi → WAS 통신 프로토콜은 wsgi 프로토콜

## 궁금증

1. Web Server 없이 Django만 가지고 사용 가능한가
2. wsgi만 붙이고 사용 가능한가?

# Spring을 서버로 쓰기 위해서는

그 자체로 그냥 써도 괜찮다

- Spring은 다중 스레드를 원활히 지원하기 때문
- 여러 스레드는 하나의 객체(싱글톤 객체)를 사용해서 작업을 한다.