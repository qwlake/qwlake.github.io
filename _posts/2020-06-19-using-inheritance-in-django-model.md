---
layout: post
title: "[Django] 장고 모델에 상속 사용하기"
subtitle: "비슷한 모델 여러 개 생성, 상속으로 해결하기"
author: qwlake
categories: Django
tags: Django Model Inheritance OntoOneField
---

# 서론

비슷한 Django 모델을 여러 개 만들어야할 때가 있다. Python의 상속을 통해 이를 해결해보고자 한다.

# 어떤 경우에 사용할까?

Django는 모델 하나가 데이터베이스 테이블 하나를 구성한다. 예를 들어 다음과 같은 모델이 있다고 가정하자.

```python
from django.db import models

class Model1(models.Model):
    title = models.CharField(max_length=100)
    link = models.CharField(max_length=200)

class Model2(models.Model):
    title = models.CharField(max_length=100)
    link = models.CharField(max_length=200)
```

각각의 모델마다 테이블을 분리할 필요가 있지만, 각 테이블이 똑같은 필드를 갖는다면 위와 같은 구조는 관리적 측면이나 코드상으로 좋지 않다. 특히 이 모델이 수십 개라면 더 그렇다.

# 모델에 상속 사용 방법

나는 일단 부모 모델을 만들고, 이 모델을 각 모델에 상속시켰다.

```python
from django.db import models

class ModelSupervisor(models.Model):
    title = models.CharField(max_length=100)
    link = models.CharField(max_length=200)

class Model1(ModelSupervisor):
    pass

class Model2(ModelSupervisor):
    pass
```

이렇게 하면 `Model1` , `Model2` 모델은 각각 `title` , `link` 필드를 갖는다.

## 모델을 불러올 때

`ModelSupervisor` 모델 구조를 갖는(상속 받는) 모델들을 불러오고 싶으면 `views.py`에서 다음과 같이 사용 가능하다.

```python
models = ModelSupervisor.objects.all()
```

만약 위와 같은 방식을 사용하지 않고 모든 모델을 불러오고자 한다면, 개별 모델에 대해 각각 모두 불러온 뒤에 합쳐야 하는 과정이 필요하다. 실제로 두 방식을 비교했을때, `개별 모델 호출 후 합치는 방식`이 `상속 방식` 보다 약 1.5배의 시간이 더 걸렸다.

## 내부 구조

`migrations` 작업 후에 자동적으로 생성되는 `migrations` 폴더의 initial 파일은 다음과 같다. `Model1` , `Model2` 모델을 생성할 때에는 `ModelSupervisor` 의 내용을 `OneToOneField` 로 가져오는 것을 볼 수 있다. 이는 Django의 User 모델을 만들 때, 기본 User 모델을 가져와 `OneToOneField` 로 매핑하는 것과 같은 방식이다. 

```python
from django.db import migrations, models
import django.db.models.deletion

class Migration(migrations.Migration):

    initial = True

    dependencies = [
    ]

    operations = [
        migrations.CreateModel(
            name='ModelSupervisor',
            fields=[
                ('title', models.CharField(max_length=100)),
                ('link', models.CharField(max_length=200)),
            ],
        ),
        migrations.CreateModel(
            name='Model1',
            fields=[
                (
                    'modelsupervisor_ptr', 
                    models.OneToOneField(
                        auto_created=True, 
                        on_delete=django.db.models.deletion.CASCADE, 
                        parent_link=True, 
                        primary_key=True, 
                        serialize=False, 
                        to='app_name.ModelSupervisor')),
            ],
            bases=('app_name.modelsupervisor',),
        ),
        migrations.CreateModel(
            name='Model2',
            fields=[
                    'modelsupervisor_ptr', 
                    models.OneToOneField(
                        auto_created=True, 
                        on_delete=django.db.models.deletion.CASCADE, 
                        parent_link=True, 
                        primary_key=True, 
                        serialize=False, 
                        to='app_name.ModelSupervisor')),
            ],
            bases=('app_name.modelsupervisor',),
        ),
    ]
```

# 진보된(?) 방법

나의 경우에는 필드가 똑같은 모델을 수십 개 만들어야 했다. 그런데 이 수십 개를 일일이 `[models.py](http://models.py)` 에 적어줄 수는 없었다. 그래서 나는 편법을 쓰기로 했다.

```python
from django.db import models
from app_name.data import data

class ModelSupervisor(models.Model):
    title = models.CharField(max_length=100)
    link = models.CharField(max_length=200)

"""
class Main(ModelSupervisor):
    pass
"""
# 위와 같은 형식의 모델이 data로부터 자동 생성
for model_name in data:
    txt = f"""
class {model_name}(ModelSupervisor):
    pass
"""
    exec(compile(txt,"<string>","exec"))
```

`String` 형식으로된 모델의 껍떼기를 저장하고, 클래스 이름에는 외부에서 불러온 모델 이름을 매핑한다. 그리고 이를 `exec` 으로 실행시켜, 실제 모델 클래스 코드가 존재하는 것처럼 만들었다. 

실행하기 전까지는 실제로 모델 코드가 존재하는 것이 아니니 좋은 코드라고 할 수는 없을 것이다. 이에 대해서는 Cool한 방법을 더 연구해봐야겠다.

# 끝