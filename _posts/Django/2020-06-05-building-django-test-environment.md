---
layout: post
title: "[Django] pytest를 사용한 장고 테스트 환경 구축"
subtitle: "+ pytest Docker"
author: qwlake
categories: Django
tags: Django test pytest Docker
---

# 서론

Django 서버의 API 요청, Model 생성, 기능 작동 등을 테스트 해보려 한다.

# 테스트가 필요한 이유

우리는 기능을 추가할 때마다 기존 기능과 새로 추가된 기능이 모두 동작하는지 확인할 필요가 있다. 테스트 환경이 없다면, 이 때마다 개발자가 모든 기능을 일일이 열람하며 잘 작동하는지 확인해야 할 것이다. 하지만 테스트 환경을 잘 구축해 놓는다면 테스트 작동 명령어 입력 한 번으로 내가 짜 놓은 모든 테스트를 한 번에 할 수 있다. 혹은 [TDD](https://gmlwjd9405.github.io/2018/06/03/agile-tdd.html)를 채택하여 기능을 추가하기 전에 해당 기능을 테스트할 테스트 케이스를 먼저 만들어 놓고 개발을 시작할 수도 있겠다.

# 왜 Django 내장 테스트 기능이 아닌 pytest를 쓸까?

Django의 내장 테스트 기능은 느리기로 유명하다. 대안으로 python을 위한 테스트 라이브러리로는 `pytest`가 있고, Django를 위한 테스트 라이브러리로 `pytest-django`가 있다. 모두 `pytest-dev` 팀(?)에서 만들었고, 업데이트도 최근까지 이루어지고 있다. 따라서 나는 `pytest-django` 를 사용하기로 했다.

# 설치

`pytest-django` 의 공식 문서 중 [Getting started with pytest and pytest-django](https://pytest-django.readthedocs.io/en/latest/tutorial.html) 를 같이 보며 따라 하면 좋다.

다음의 명령어로 로컬의 가상 환경 등에 설치 하거나,

```bash
pip install pytest-django pytest-xdist model_mommy
```

다음의 예시처럼 `requirements.txt` 파일에 라이브러리를 추가시킨다.

```
pytest-django==3.8.0
pytest-xdist==1.32.0
model_mommy==1.5.1
```

- `pytest-xdist` 는 테스트시에 멀티 코어를 사용하고자 한다면 필요하다. 테스트 자체에는 필요가 없다.
- `model_mommy` 는 테스트를 위해 model 객체를 생성할 때 유용한 라이브러리이다. 물론 이것도 model 객체 생성 테스트를 하지 않는다면 필요 없다. 아래에서 더 다루겠다.

## pytest를 Django settings에 넣기

Django 프로젝트의 루트 디렉토리에 `pytest.ini` 파일을 생성하고 그 안에 다음의 내용을 삽입한다.

```
[pytest]
DJANGO_SETTINGS_MODULE = yourproject.settings
python_files = tests.py test_*.py *_tests.py
```

- `python_files`: 테스트에 사용할 테스트 케이스가 적힌 파일의 형식을 적어준다. 이 경우에는 `[tests.py](http://tests.py)` 파일이나 `test_` 로 끝나거나 `_test` 로 끝나는 파이썬 파일을 명시해 주었다.

## 테스트 파일 작성

Django 앱을 생성하면 `[tests.py](http://tests.py)` 파일이 생성될 것이다. 이 파일은 잘 쓰이지 않으니 삭제하고, 대신 `tests` 폴더를 생성한다. 생성된 폴더 안에 테스트 파일을 생성한다. 파일명은 `tests.py` `test_*.py` `*_tests.py` 중 하나의 형식을 따르면 된다. 생성 후에 다음과 같이 테스트 케이스를 작성한다.

### 파일을 작성하기 전에

1. 지금부터 작성할 테스트 케이스는 Django의 기본 테스트 기능을 사용할 때와 형식이 같다. 하지만 실행은 pytest로 한다는 점을 알아두자. 그리고 이를 다시 말하면 기존에 Django 기본 테스트 기능을 사용했더라도 pytest 설치 후 pytest로 실행만 한다면 pytest로 동작한다는 의미이다.
2. 테스트를 수항하기 위해서는 프로덕션 DB에 영향이 있으면 안 된다. 때문에 pytest에서는 각 테스트 클래스별로 트랜잭션을 분리하고, 글래스 내부의 테스트 케이스(함수)들 역시 서로 간에 분리되어 있다. 즉, 각 테스트 케이스(함수)가 끝나면 트랜잭션이 롤백 되어 초기화된다.
3. `assert` 의 의미를 알아두고 시작하자. [assert란?](https://wikidocs.net/21050)

### Model 테스트

model을 생성하는 간단한 테스트 케이스다. `model_mommy` 는 model의 각 속성별로 값을 일일이 넣어 주는 과정을 생략하고 간단하게 생성 테스트를 할 수 있게 도와준다. 

```python
from django.test import TestCase
from model_mommy import mommy

from yourapp import models

class TestModels(TestCase):
    def setUp(self):
        self.model_list = []
        for name, cls in models.__dict__.items():
            if isinstance(cls, type):
                self.model_list.append(mommy.make(cls))

    def test_create_models(self):
        for model in self.model_list:
            assert model
```

### API 테스트

`setUpTestData` 함수는 테스트 클래스가 처음 실행될때 초기 세팅을 위해 테스트 케이스보다 먼저 실행되는 함수이다. 테스트에는 포함되진 않으나 테스트를 위한 데이터를 삽입, 생성시킬 수 있다. 이와 같은 함수로 `setUp`, `setUpTestClass`가 있다. 차이점은 다음의 글에서 잘 다루고 있다. [Django Test Fixture: setUp, setUpClass and setUpTestData](https://medium.com/an-engineer-a-reader-a-guy/django-test-fixture-setup-setupclass-and-setuptestdata-72b6d944cdef)

```python
from django.test import TestCase, Client
from rest_framework import status

class TestDevices(TestCase):
    @classmethod
    def setUpTestData(cls):
        data = {
            'id':'test',
            'keywords':'keyw',
            'subscriptions':'main',
        }
        response = client.post('/accounts/device', data=data, content_type='application/json')

    def test_accounts_device_get(self):
        data = {
            'id':'test',
        }
        response = client.get('/accounts/device', data=data, follow=True)
        assert status.is_success(response.status_code)
```

# 실행

로컬 환경에서 직접 `python pytest` 명령어를 타이핑하여 테스트할 수도 있다. 나는 데이터베이스 연결을 위해 테스트용 docker-compose 파일을 만들었다. 다음과 같이 타이핑하여 파일을 `docker-compose.test.yml` 이라고 생성한다. 

```yaml
version: '3'
    
services:
    web:
        build: .
        container_name: web01
        command: bash -c "
            python -m pytest -n 2"
        volumes:
            - ./src:/home/src
            - /static:/static

    db:
        image: postgres:12.2
        container_name: db01
        ports:
            - "5432:5432"
        volumes:
            - ./config/postgres-passwd.txt:/run/secrets/postgres-passwd
        environment:
            - POSTGRES_DB=postgres
            - POSTGRES_USER=postgres
            - POSTGRES_PASSWORD_FILE=/run/secrets/postgres-passwd
```

이제 `docker-compose -f docker-compose.test.yml up` 을 타이핑하면 테스트를 할 수 있다.

# 끝