---
layout: post
title: "[Django] pgAdmin 적용과 Nginx 설정"
subtitle: "+ Docker"
author: qwlake
categories: Django
tags: Django Docker PostgreSQL pgAdmin Nginx
---

# 서론

우리팀의 Django 서버의 데이터베이스는 PostgreSQL을 사용한다. 이 데이터베이스를 조금 더 쉽게 관리, 모니터링하기 위해서 pgAdmin을 설치하려 한다. 또한 우리는 docker로 관리되고 있는 상황이라 이 과정에서 pgAdmin으로 접속하기 위한 Nginx 설정도 다뤄보려 한다. 

# pgAdmin?

[https://www.pgadmin.org/faq/#1](https://www.pgadmin.org/faq/#1)

> pgAdmin은 PostgreSQL 및 EnterpriseDB의 EDB Advanced Server와 같은 파생 관계형 데이터베이스를위한 관리 도구입니다. 웹 또는 데스크톱 응용 프로그램으로 실행될 수 있습니다. 제공되는 기능에 대한 자세한 내용은 기능 및 스크린 샷 페이지를 참조하십시오.

간단히 말해서 PostgreSQL을 위한 데이터베이스 관리 도구이다. 웹 또는 설치형 프로그램으로 실행 가능하다.

# pgAdmin 설치

나는 docker 이미지를 띄워서 사용할 것이기 때문에 `docker-compose.yml`에 다음과 같이 pgadmin을 추가해준다.

```yaml
version: '3'
    
services:
    pgadmin:
        image: dpage/pgadmin4:4.21
        container_name: pgadmin01
        environment:
            - PGADMIN_DEFAULT_EMAIL=postgres
            - PGADMIN_DEFAULT_PASSWORD=postgres
            - PGADMIN_LISTEN_PORT=5050
        depends_on:
            - db
    ...
```

사실 설치라고 할 것도 없는게 docker 이미지를 가져와 쓰는거라.. 이게 끝이다. pgAdmin 페이지 접속 계정 아이디(이메일)과 주소는 위에서 보다시피 postgres로 지정해 주었다. 이는 로그인해서 변경할 수 있다. 그리고 접속 포트는 `5050`이다. 도커 컨테이너를 띄운 후에 [http://localhost:5050](http://localhost:5050) 으로 접속한다면 pgAdmin에 접속할 수 있다.

# Nginx 설정

자 우리는 Nginx가 있으니 여기도 건드려줘야 한다. 또한 나처럼 AWS 등에 배포해놓고 쓴다면 프록시 헤더도 고정시켜줘야 한다. Nginx 관련 설정이 들어있는 파일(nginx.conf, default.conf, etc)을 열어서 다음과 같이 작성한다.

```
# Django
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

# pgAdmin
server {
    listen 5050;

    access_log /var/log/nginx/5050_access.log;
    error_log /var/log/nginx/5050_error.log;

    location / {
        proxy_set_header Host $host:$server_port;
        proxy_pass http://pgadmin:5050;
        proxy_redirect off;
    }
}
```

Nginx상에서 서버는 두 대로 인지한다. 하나는 Django, 하나는 pgAdmin. 

`listen 5050;` : 접속 주소의 5050번 포트는 pgAdmin에서 처리한다는 뜻이다.

`access_log`, `error_log` : 해당 포트를 통해 들어온 요청은 승인시 `access_log` 에 저장되고, 각종 오류로 인해 접속 실패했을 경우에는 `error_log` 에 저장된다.

`location /` : 접속 주소의 기본 주소(root 주소, [http://localhost:5050/](http://localhost:5050/))로 접속했을때 괄호 안의 내용으로 처리한다는 뜻이다.

`proxy_pass http://pgadmin:5050;` : 5050번 포트의 기본 주소로 접속했을때 pgadmin의 5050번 포트로 안내(프록시)한다.

`proxy_set_header Host $host:$server_port;` : 이 설정은 프록시를 통해 다른 호스트를 요청할때 호스트의 헤더를 고정하는데 쓰인다. 만약 이를 해주지 않는다면, 프록시 헤더는 위의 `proxy_pass`에서 가져온 `http://pgadmin:5050` 로 지정될 것이다. 하지만 이는 컨테이너를 지정한 것이지 실제 주소는 아니다. 따라서 현재 사용하고 있는 주소와 포트로 고정시켜야 한다.

# 끝