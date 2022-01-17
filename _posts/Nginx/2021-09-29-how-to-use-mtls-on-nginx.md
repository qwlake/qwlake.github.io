---
layout: post
title: "[Nginx] mTLS 샘플 인증서 발급 및 검증"
author: qwlake
categories: Nginx
tags: mtls ssl
---

# 테스트용 인증서 발급

### ca 인증서 발급

```bash
openssl req \
  -newkey rsa:4096 \
  -x509 \
  -keyout ca.key \
  -out ca.crt \
  -days 30 \
  -nodes \
  -subj "/CN=my_ca"
```

### 서버 인증서 발급

```bash
openssl req \
  -newkey rsa:4096 \
  -keyout server.key \
  -out server.csr \
  -nodes \
  -days 30 \
  -subj "/CN=localhost"

openssl x509 \
  -req \
  -in server.csr \
  -out server.crt \
  -CA ca.crt \
  -CAkey ca.key \
  -CAcreateserial \
  -days 30
```

### 클라이언트 인증서 발급

```bash
openssl req \
  -newkey rsa:4096 \
  -keyout client.key \
  -out client.csr \
  -nodes \
  -days 30 \
  -subj "/CN=client/serialNumber=110111-7107728"

openssl x509 \
  -req \
  -in client.csr \
  -out client.crt \
  -CA ca.crt \
  -CAkey ca.key \
  -CAcreateserial \
  -days 30
```

# nginx 세팅

## nginx install (docker)

```yaml
# docker-compose.yml

version: '3.8'

services:
  nginx:
    image: nginx:1.16.1
    ports:
      - "8443:443"
    volumes:
      - ./default.conf:/etc/nginx/conf.d/default.conf # nginx 설정 파일 경로
      - ./server/:/etc/ssl/ # 서버 인증서 폴더 경로
      - ./ca/ca.crt:/etc/nginx/client_certs/ca.crt # ca 인증서 경로
```

```bash
$ docker-compose up --build
```

## nginx 설정

```
# default.conf

server {
    listen                  443 ssl;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    ssl_certificate         /etc/ssl/server.crt;
    ssl_certificate_key     /etc/ssl/server.key;
    ssl_protocols           TLSv1.2 TLSv1.3;

    ssl_client_certificate  /etc/nginx/client_certs/ca.crt;
    ssl_verify_client       on;
    ssl_verify_depth        2;

    location / {
        if ($ssl_client_verify != SUCCESS) { return 403; }

        proxy_set_header     SSL_Client_Issuer $ssl_client_i_dn;
        proxy_set_header     SSL_Client $ssl_client_s_dn;
        proxy_set_header     SSL_Client_Verify $ssl_client_verify;
        proxy_set_header     SSL_CLIENT_ESCAPED_CERT $ssl_client_escaped_cert;  # urlencoded된 client 인증서를 헤더에 삽입

        proxy_pass           http://host.docker.internal:9090;
    }
}
```

- `host.docker.internal` : docker 내부에서 로컬을 바라볼 때 사용하는 주소

# client에서 nginx로 요청 보내기

## postman 사용

### SETTINGS에서 ca, client 인증서 등록 (아래 사진 참조)

![image](https://user-images.githubusercontent.com/41278416/149715657-d72ee72e-8a2d-473e-8d85-2c2ea1daa7c2.png)

### 요청 보내기

포스트맨에서 알아서 헤더에 인증서 삽입해주므로 헤더 세팅 필요 없음

![image](https://user-images.githubusercontent.com/41278416/149715672-153a51c6-f9af-4829-9595-ca7854723dd5.png)

### 요청 세부

![image](https://user-images.githubusercontent.com/41278416/149715689-e0faa47d-a002-4089-92c5-aa2c5671677a.png)

## curl 이용

```bash
curl https://localhost:8443/test \
  --cacert ca.crt \
  --key client.key \
  --cert client.crt
```

# WAS에서 헤더에 삽입된 인증서 뽑아내기

```kotlin
@GetMapping("/test")
fun test(req: HttpServletRequest): String {
    val rawClientCert = req.getHeader("ssl_client_escaped_cert")
    return rawClientCert
}
```

# What to do next

rawClientCert를 `X509Certificate` 타입으로 변환시킨 다음, 인증서 안에 있는 항목을 뽑아낼 수 있다.