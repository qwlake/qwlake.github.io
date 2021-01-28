---
layout: post
title: "[Server] Server shutdown problem"
subtitle: "Django memory leak with scrapy"
author: qwlake
categories: ETC
tags: Server Django Scrapy memory-leak
---

# Finding reasons

1. Add logging system to Django project. → Found Reason 1.
2. Add `AWS EC2 instance memory monitor` to `cloud watch`. → Found Reason 2.
[https://brunch.co.kr/@topasvga/615](https://brunch.co.kr/@topasvga/615)

## Reason 1.

```
TypeError: argument must be an int, or have a fileno() method.
```

[TypeError: argument must be an int, or have a fileno() method. · Issue #49 · ppcomp/knu-notice-server](https://github.com/ppcomp/knu-notice-server/issues/49)

## Reason 2.

Django Memory Leak

![image](https://user-images.githubusercontent.com/41278416/106122809-79baa680-619c-11eb-886f-79f1101d85d9.png)

# Direction

### Why memory leak?

메모리 누수가 발생하는 원인 찾기

cf. [https://medium.com/@elastic7327/운영중인-장고-지유니콘-앱백엔드-메모리-누수-문제-해결-production-django-gunicorn-backend-memory-leak-fix-feat-uwsgi-b6013e3e0514](https://medium.com/@elastic7327/%EC%9A%B4%EC%98%81%EC%A4%91%EC%9D%B8-%EC%9E%A5%EA%B3%A0-%EC%A7%80%EC%9C%A0%EB%8B%88%EC%BD%98-%EC%95%B1%EB%B0%B1%EC%97%94%EB%93%9C-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EB%88%84%EC%88%98-%EB%AC%B8%EC%A0%9C-%ED%95%B4%EA%B2%B0-production-django-gunicorn-backend-memory-leak-fix-feat-uwsgi-b6013e3e0514)

### If not possible?

임시방편으로 메모리 누수로 인한 커널패닉이 일어나기 전에 재부팅해주기