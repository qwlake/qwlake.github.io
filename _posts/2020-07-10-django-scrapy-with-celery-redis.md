---
layout: post
title: "[Django] Celery,Redis로 Scrapy 크롤링 주기적으로 하기"
subtitle: "+ Docker"
author: qwlake
categories: Django
tags: Django Docker Scrapy Celery Celery-Beat Redis
---

# 서론

우리 팀에서 Django로 구축한 서버는 Scrapy로 주기적으로 크롤링해서 데이터베이스에 저장하는 역할을 한다. 동시에 저장된 데이터를 달라는 요청이 들어오면 이것도 대응해야 한다. 만약 이 과정을 동기로 처리한다면 크롤링 하는 동안에는 요청에 응답하지 못하기 때문에 비동기로 처리하려고 한다. 또한, 주기적으로 작업(크롤링) 이벤트를 발생 시켜야 하는 상황이므로 Celery와 Redis 조합으로 이벤트를 스케쥴링 해보려 한다.

# Celery Redis?

Celery는 비동기 작업 큐이다. 이 큐 안에 있는 작업(Work)을 처리하기 위해서는 메세지 브로커가 플요하다. 내가 사용한 메세지 브로커는 Redis고, 이 Redis가 워커(Worker)를 관리한다. 그렇다면 Celery 큐에는 작업을 어떻게 넣어줄까? 나는 Celery-beat을 사용했다.

# Celery

## Celery 설치

Celery는 파이썬 기반 라이브러리이기 때문에 `pip`으로 설치한다.

```bash
pip install celery
```

`docker-compose.yml` 상에서 컨테이너는 다음과 같이 세팅해준다.

```yaml
version: '3'
    
services:

...

		celery:
        build: .
        container_name: celery01
        command: celery worker --app=crawling.tasks --loglevel=INFO
        volumes:
            - ./knu_notice:/src
        depends_on:
            - db
            - redis

    celery-beat:
        build: .
        container_name: celery-beat01
        command: celery beat --app=crawling.tasks --loglevel=INFO
        volumes:
            - ./knu_notice:/src
        depends_on:
            - db
            - redis
```

Django의 `settings.py`의 `INSTALLED_APPS`에 `celery`를 추가한다.

```python
INSTALLED_APPS = [
		...
    'celery',
    ...
]
```

## Celery 세팅

Django의 `settings.py` 에서 Celery의 메세지 브로커 서버(Redis)와 `BEAT_SCHEDULE` 을 명시해 준다.

```python
from celery.schedules import crontab, timedelta

CELERY_BROKER_URL = 'redis://redis:6379/0'
CELERY_RESULT_BACKEND = 'redis://redis:6379/0'
CELERY_BEAT_SCHEDULE = {
    'hello': {
        'task': 'crawling.tasks.crawling',
        'schedule': timedelta(seconds=20)
    }
}
```

위 코드는 20초 마다 `crawling` 앱(Django 앱)의 `tasks` 파일의 `crawling` 함수가 실행된다는 뜻이다. 스케쥴링 시간 간격은 `crontab`, `timedelta` 을 적절히 활용하면 된다.

## Task 생성

작업을 실행할 Django 앱 하나를 골라서(내 경우에는 `crawling`) 앱 폴더 안에 `tasks.py` 파일을 만들고 다음의 코드를 작성한다.

```python
import logging, os
from billiard.context import Process

# from django.conf import settings
from knu_notice.celery import app

import scrapy
from scrapy.crawler import CrawlerProcess
from .crawler.crawler.spiders.crawl_spider import CrawlSpiderSpider
from scrapy.settings import Settings

settings = Settings()
os.environ['SCRAPY_SETTINGS_MODULE'] = 'crawling.crawler.crawler.settings'
settings_module_path = os.environ['SCRAPY_SETTINGS_MODULE']
settings.setmodule(settings_module_path, priority='project')

def crawling_start():
    process = CrawlerProcess(settings)
    process.crawl(CrawlSpiderSpider)
    process.start()

@app.task
def crawling():
    proc = Process(target=crawling_start)
    proc.start()
    proc.join()
```

위에서 `crawling` 함수를 실행한다고 했는데, `crawling` 함수는 다시 `crawling_start` 함수를 서브 프로세스로 호출한다. 이 이유는 Redis로 생성된 Worker 프로세스에서 Scrapy를 동작시키면, Scrapy의 작업이 끝나도 Worker 프로세스가 종료되지 않는 문제가 있기 때문이다. 
그리고 파이썬 기본 Process를 쓰지 않고 `billiard` 를 사용했는데, 이는 파이썬 기본 Process 모듈이 서브 프로세스에서 또 자식 프로세스를 생성하지 못하는 특성 때문이다. 기본적으로 Worker 프로세스는 Django 프로세스가 만든 `서브 프로세스` 이다. 때문에 파이썬 기본 Process를 통해 프로세스를 생성하려 하면 다음과 같은 에러를 마주하게 될 것이다. `twisted.internet.error.ReactorNotRestartable`. 하지만 `billiard` 를 사용하면 프로세스 생성이 가능하다.

# Redis

## Redis 설치

Redis는 docker 이미지가 존재한다. 따라서 `docker-compose.yml` 파일에 다음과 같이 넣어 줬다.

```yaml
version: '3'
    
services:

...

		redis:
        image: redis:6.0-rc4
        container_name: redis01
        ports:
            - "6379:6379"
```

Django에서 redis를 사용하려면 pip을 이용한 설치도 해주어야 한다. (사실 없어도 되는 것 같은데 내 프로젝트에는 있어서 혹시나 해서 넣는다. 없어도 될 것 같긴 하다.)

```bash
pip install redis
```

# Scrapy

Scrapy는 별도의 프로젝트 구조가 있기 때문에 로컬 환경에서 파이썬 가상환경(`venv`) 를 만들고 그 안에서 Scrapy를 설치한 후, Scrapy 프로젝트를 생성해서 Django 프로젝트 안에 복사 붙여넣기로 넣는 방법을 추천한다.

## Scrapy 설치

```bash
pip install scrapy
```

## Scrapy 프로젝트 생성

```bash
scrapy startproject crawler
```

생성된 crawler 폴더를 Django의 앱 안에 넣는다. (인터넷을 보면 앱 안에 넣는 사람도 있고, 앱 폴더와 동일한 레벨에 넣는 사람도 있다. 편한대로 하면 된다.) `spiders` 폴더 안에 spider클래스(크롤링 코드가 적히는 클래스)를 선언할 파일을 만들고 다음과 같은 코드를 작성한다.

```python
import scrapy
from scrapy.linkextractors import LinkExtractor

class CrawlSpiderSpider(scrapy.Spider):
    name = 'crawl_spider'
    start_urls = ['https://www.naver.com']

    def parse(self, response):
        url_form = LinkExtractor(restrict_xpaths='/html/body/div[2]/div[3]/div[2]/div[2]/div[1]/div[1]/div/div/div/div/a',attrs='href')
        urls = url_form.extract_links(response)

        for item in urls:
            scrapyed_info = {
                'link' : item[0].url,
            }
            yield scrapyed_info
```

위 코드는 예시이다. 작동 되는지 확인하지 않았다! 네이버의 뉴스 기사 배너 링크를 가져와 상대 주소를 실제 주소로 바꾸고 변수에 저장한 후에 `yield` 한다. [yield](https://dojang.io/mod/page/view.php?id=2412)란? 

## Scrapy settings

Scrapy는 크롤링 후에 `pipelines` 를 통해 크롤링된 데이터를 가공, 저장한다. 참고로 `pipelines`은 비동기로 작동하기 때문에 무거운 데이터 처리나 저장(IO 오버헤드 발생) 작업 등은 spider보단 `pipelines` 안에서 하는 것이 좋다.

다음은 `pipelines.py` 이다. Django 모델 구조를 사용해 데이터베이스에 저장하는 모습이다.

```python
from crawling.models import *

class CrawlerPipeline:
    def process_item(self, item, spider):
        cse = Cse(
            link = item['link'],
        )
        cse.save()
        return item
```

파이프라인을 사용하기 위해서는 파이프라인의 이름을 settings에 적어줘야 한다.

```python
BOT_NAME = 'crawler'
SPIDER_MODULES = ['crawling.crawler.crawler.spiders']
NEWSPIDER_MODULE = 'crawling.crawler.crawler.spiders'
...
ROBOTSTXT_OBEY = False
...
ITEM_PIPELINES = {
   'crawling.crawler.crawler.pipelines.CrawlerPipeline': 300,
}
...
```

그리고 이 settings를 사용하기 위해서는 crawler 프로세스를 작동시킬때 이 settings를 사용한다고 명시해 주어야 한다. 이는 위에서 `tasks.py` 파일을 작성할때 같이 해주었다.

# 끝

이제 Django 프로젝트를 실행시키면 Scrapy가 20초에 한 번씩 실행될 것이다. 포스팅하면서 빼먹은 부분이 있을 수도 있으니 안 되면 댓글 부탁합니다~