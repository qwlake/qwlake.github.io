---
layout: post
title: "[Docker] Docker-compose auto-start after lauching AWS EC2 server"
author: qwlake
categories: Docker
tags: docker docker-compose 
---

1. Enable docker daemon
```
sudo systemctl enable docker
```

2.  Add `restart` option to `docker-compose.yml`
```
web:
    restart: always
    build: .
    container_name: web01
    command: bash -c "
        python manage_dev.py collectstatic --no-input &&
        python manage_dev.py makemigrations && 
        python manage_dev.py migrate &&
        gunicorn knu_notice.dev.wsgi -b 0:8080 --reload"
    volumes:
        - ./knu_notice:/home/knu_notice
    links:
        - redis
```