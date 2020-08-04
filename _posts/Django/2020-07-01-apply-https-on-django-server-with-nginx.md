---
layout: post
title: "[Django] 장고 Rest API 서버 https 프로토콜 적용하기"
subtitle: "nginx가 있을 때는 어떡하지?"
author: qwlake
categories: Django
tags: Django Model Inheritance OntoOneField
---

# 서론

클라이언트(어플)에서 서버(Django)와 통신할 때 통신내용의 보안을 강화하고자 https 프로토콜을 사용하고자 한다.

# https란?

깊게 파면 글이 너무 길어지니 짧게 하겠다. 우리가 이전에 주로 사용했던 http 프로토콜은 암호화되어 있지 않기 때문에 중간에 누가 패킷을 가로챈다면 내용을 볼 수 있다. 당장 여러분의 컴퓨터에 와이어샤크만 깔아도 패킷 내용을 볼 수 있다. 

누군가는 `Get`말고 `Post` 방식을 사용하면 내용을 숨길 수 있다고 하는데, 그렇지 않다. 내용이 주소에서 Request의 Body로 이동했을 뿐이다. 패킷을 가로채면 내용을 볼 수 있다는 사실은 변함없다.

이를 보안하기 위해 http에 SSL 프로토콜을 사용한 것이 https이다.

# SSL 인증서

https를 사용하기 위해서는 SSL 인증서가 필요하다. 이 인증서를 통해 암호화를 할 수 있기 때문이다.(정확히는 클라이언트에게 공개키를 전달하는 역할을 한다) SSL 인증서는 공인된 인증기관(CA)에서 발급받을 수도 있고, 사설 인증서를 발급받거나 직접 만들 수 있다. 

단, 공인된 CA로부터 인증서를 발급받기 위해서는 도메인 주소가 필요하다. 그런데 나의 경우에는 사용자가 직접 서버를 방문하는 것이 아닌, 클라이언트에서 서버와 통신하는 것이므로, 도메인 주소가 필요 없다. 오직 나는 암호화만 필요하기 때문에 SSL 인증서를 직접 만들어 사용하기로 했다.

## 인증서 생성

인증서는 `.key` 파일(개인키 역할)과 `.crt` 파일(공개키 역할)로 구분된다. 상황에 따라 둘을 합친 `.pem` 파일을 사용하기도 한다.

먼저 key 파일을 생성한다.

```bash
openssl genrsa 1024 > django.key
```

키 파일을 갖고 crt 파일을 생성한다.

```bash
openssl req -new -x509 -nodes -sha256 -days 365 -key django.key > django.crt
```

생성된 두 파일을 Django 프로젝트 폴더 내부로 옮긴다. 나는 nginx를 사용하기 때문에 Django 프로젝트 외부에 설정 파일을 모아둔 곳으로 옮겼다. 상황에 맞게 하자.

# https 적용

## nginx가 있을 때

난 처음에는 nginx가 있어도 Django에 `sslserver` 를 설치해야 하는줄 알았다. 그런데 그게 아니고 제일 앞단, 그러니까 nginx에만 https 설정을 해주면 되는 것이었다. 꽤 삽질을 많이 한 부분이다. Django는 그냥 그대로 가면 된다.

`nginx/default.conf` 파일(nginx 설정 파일)을 다음과 같이 바꾼다. 코드상에서 80번 포트를 `listen` 하고 있는 server가 기존 것이다. 그 아래에 443번 포트를 `listen` 하고 있는 server가 https 접속을 받는 것이다(참고로 http 접속시 기본 포트는 80번, https 접속시 기본 포트는 443번으로 들어온다).

나는 일단 아래와 같이 http와 https 둘 다 살려두었다. 추후에 80번 포트로 접속해도 https 프로토콜로 대응하는쪽으로 변경할 예정이다.

```
server {
    listen 80;

    access_log /var/log/nginx/80_access.log;
    error_log /var/log/nginx/80_error.log;

    location / {
        proxy_set_header Host $host:$server_port;
        proxy_pass http://web:8080;
        proxy_redirect off;
    }
    location /static/ {
        alias /home/knu_notice/.static_root/;
    }
}

server {
    listen 443 ssl;

    ssl_certificate     /etc/nginx/conf.d/django.cert;
    ssl_certificate_key /etc/nginx/conf.d/django.key;
    
    access_log          /var/log/nginx/443_access.log;
    error_log           /var/log/nginx/443_error.log;

    location / {
        proxy_set_header Host $host:$server_port;
        proxy_pass http://web:8080;
        proxy_redirect off;
    }
    location /static/ {
        alias /home/knu_notice/.static_root/;
    }
}
```

나와 같이 `docker-compose` 를 사용중이라면 다음과 같이 포트도 열고 `.key` , `.crt` 파일도 볼륨 마운트를 해주어야 할 것이다.

```docker
version: '3'
    
services:
    nginx:
        image: nginx:1.17.10
        container_name: nginx01
        ports:
            - "80:80"
            - "443:443"
        volumes:
            - ./knu_notice:/home/src
            - ./config/nginx/:/etc/nginx/conf.d/

    web:
        build: .
        container_name: web01
        command: bash -c "
            python manage_dev.py collectstatic --no-input &&
            python manage_dev.py makemigrations && 
            python manage_dev.py migrate &&
            gunicorn knu_notice.wsgi -b 0:8080"
        volumes:
            - ./knu_notice:/home/src
```

## nginx가 없을 때

1. `django-sslserver` 설치
    - `requirements.txt` 를 쓴다면 다음을 해당 파일에 추가한다.

```
django-sslserver==0.22
```

- 그게 아니라면 다음과 같이 `django-sslserver` 를 설치한다.

```bash
pip install django-sslserver
```

2. `settings.py` 의 `INSTALLED_APPS` 에 `sslserver` 를 추가한다.

```python
INSTALLED_APPS = [
    ....
    'sslserver',
]
```

3. 다음의 명령어로 실행시키면 된다.

```bash
python manage.py runsslserver --certificate django.cert --key django.key
```

# 끝