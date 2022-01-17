---
layout: post
title: "[Django] 장고 Rest API 서버 https 프로토콜 적용하기 (feat. nginx)"
author: qwlake
categories: Django
tags: Model Inheritance OntoOneField
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

당신이 ssl 설정을 적용하려 한다면 배포를 한다는 뜻이고, 이는 곧 web server를 사용하고 있다는 뜻일 것이다. 설마 배포하는데 WAS(Django)만 달랑 배포하는 일은 없길 바란다. 그 이유는 [여기](https://velog.io/@woo00oo/Web-Server%EC%99%80-WAS) 를 참조.

https는 제일 앞단인 web server에만 적용시키면 된다. 어자피 web server와 was간의 통신은 외부를 거치는게 아니고 내부간의 통신일테니.

`nginx/default.conf` 파일(nginx 설정 파일)을 다음과 같이 바꾼다. 

```
server {
    listen 80;

    location / {
         return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl;

    ssl_certificate     /etc/nginx/conf.d/django.crt;
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

굳이 web server 없이 쓰겠다면 아래와 같이 진행하면 된다.

1. `django-sslserver` 설치
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
    python manage.py runsslserver --certificate django.crt --key django.key
    ```

# 끝
