---
layout: post
title: "[Scrapy] exceptions.RuntimeError: doWrite called on a twisted.internet.tcp.Port 에러"
subtitle: "포트 할당 문제"
author: qwlake
categories: Scrapy
tags: Scrapy twisted parallel port Error
---

# 증상

다수의 Scrapy Spider 객체를 동시에 호출하여 사용할 때 발생하는 문제이다. 여기서 동시 호출이라 하면, Celery를 통해 task를 쌓아 두고 실행할 때 동시에 실행되는 task를 두 개 이상으로 하는 경우를 말한다. 이 외에도 멀티프로세스나 여러 Spider를 생성해서 동시에 실행시키는 경우가 있겠다. 내 상황에서는 Celery task 안에서 서브프로세스를 만들고 그 안에서 spider를 동작시키는 구조이다.

# 원인

각각의 Spider 객체는 크롤링을 하기 위해서 웹소켓 포트를 할당받는다. 그런데 Spider 객체가 여러 개라고 해서 여러 포트를 할당받는 것이 아니라는게 문제다. 

조금 더 깊이 들어가면, Scrapy는 `twisted` 라는 라이브러리를 베이스로 동작한다. 이는 파이썬의 멀티프로세싱을 상속한 라이브러리로써, 다중 웹 통신을 지원한다. 여기서 우리가 Scrapy를 사용하면, twisted가 서브 프로세스를 하나 생성하고 포트를 하나 할당한다. 이 때, celery woker나 billiard를 통해 여러 spider 객체를 생성 및 호출하면, 이는 모두가 처음 twisted로부터 할당받은 포트를 사용한다.

때문에 포트를 사용할 수 없다는 에러가 뜨는 것이다.

# 해결책

### 1. Spider 객체를 생성 및 호출할 때 twisted를 사용한다.

제일 근본적인 해결책이다. 동시 처리할 spider 마다 새로운 twisted 프로세스와 포트를 할당해주면 된다. 즉, spider를 생성 관리할 서브프로세스 라이브러리로 지금까지는 billiard를 사용했지만, twisted multiprocessing을 사용하는 것이다.

아래 예시가 있긴 하지만 twisted multiprocessing 관련 공식 문서 등의 자료가 너무 빈약해 다른 방법을 찾아보기로 했다.

[Multiprocessing of Scrapy Spiders in Parallel Processes](https://stackoverflow.com/a/31104744)

### 2. Telnet Console 비활성화하기.

내가 사용한 방법이고, 가장 쉽다. Scrapy `settings.py` 에 다음의 문장을 추가하면 된다. 아마 초기에는 주석 처리되어 있을 것이다. 출처: [port error in scrapy](https://stackoverflow.com/a/17491706)

```python
EXTENSIONS = {
   'scrapy.telnet.TelnetConsole': None
}
```


`Telnet Console` 은 Scrapy 실행 프로세스를 검사하고 제어하는 역할을 한다. 구체적인 정보를 얻고 싶었으나 공식 문서에는 이정도만 설명되어 있다. 

참고로, Telnet Console은 전송 계층 보안을 제공하지 않기 때문에, 로컬에서의 사용이나 보안 연결(VPN, SSH 등)을 통해 사용하는 것이 아니라면 Telnet Consol의 사용을 비추천하고 있다.

# 끝