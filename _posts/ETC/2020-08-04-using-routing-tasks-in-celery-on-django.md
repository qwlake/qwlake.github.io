---
layout: post
title: "[Celery] Django 프로젝트에서 Celery Worker 제한하기"
subtitle: ""
author: qwlake
categories: ETC
tags: Celery Django
---

서버에서 크롤링 규모를 키우다보니 CPU 자원이 딸리는 것을 느꼈다. 크롤링을 할 때마다 CPU 사용률 100%인 상황이 30분 이상 지속되었다. 때문에 클라이언트 요청도 제 때에 처리하지 못하는 상황이 발생했다. 

# Worker를 제대로 활용하자

나는 지금까지 100개의 사이트를 크롤링 해야 한다면, 하나의 Celery task에 100개 사이트를 크롤링 하라는 명령을 주었다. 어자피 비동기로 돌아가니 괜찮겠다 싶었지만, 오산이었다.

따라서 나는 해결책으로 각각의 task에 사이트를 하나씩 크롤링 하도록 변경했다. 사실 이렇게 했어야 한다.

Django 프로젝트의 views.py 파일이다. 기존에는 이렇게 했다면,

```python
@api_view(['GET'])
def init(request, pages):
    from crawling import tasks.crawling
    crawling.delay(pages)
    return Response(
        "Database initialized. All board notices are crawled.", 
        status=status.HTTP_200_OK
    )
```

이렇게 수정했다. (apply_async의 queue 옵션은 아래 Routing Tasks에서 설명한다)

```python
@api_view(['GET'])
def init(request, *arg, **kwarg):
    from crawling import tasks.crawling
    for i in range(100):
        crawling.apply_async(args=(kwarg['pages'], i), queue='slow_tasks')
    return Response(
        "Database initialized. All board notices are crawled.", 
        status=status.HTTP_200_OK
    )
```

# Routing Tasks

그리고 어느 작업 때문에 다른 작업을 놓치는 일이 없도록, 각 worker에 대해 작업 Queue를 설정해 주었다. 각 작업들을 알맞은 Queue로 보내는 것이 Routing Tasks 이다.

기존 celery.py 파일이 다음과 같았다면,

```python
from celery import Celery

app = Celery(
    'knu_notice',
    broker='redis://redis:6379/0',
    backend='redis://redis:6379/0',
)
```

여기에 사용할 Queue를 지정해준다.

```python
from celery import Celery
from kombu import Queue

app = Celery(
    'knu_notice',
    broker='redis://redis:6379/0',
    backend='redis://redis:6379/0',
)
app.conf.task_default_queue = 'default'
app.conf.task_queues = (
	Queue('slow_tasks', routing_key='slow.#'),
	Queue('quick_tasks', routing_key='quick.#'),
)
```

이제 task를 실행할 때에는 다음과 같이 Queue를 지정하여 실행하면 된다.

```python
crawling.apply_async(args=(kwarg['pages'], i), queue='slow_tasks')
```

```bash
celery worker --app=crawling.tasks -Q slow_tasks
```

# 참조

[https://docs.celeryproject.org/en/master/userguide/routing.html](https://docs.celeryproject.org/en/master/userguide/routing.html)

[https://iam.namjun.kim/celery/2018/09/09/celery-routing/](https://iam.namjun.kim/celery/2018/09/09/celery-routing/)

# 끝