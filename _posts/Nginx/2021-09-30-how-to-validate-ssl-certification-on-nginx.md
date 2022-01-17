---
layout: post
title: "[Nginx] nginx ssl certificate 검증을 위한 인증서 세팅"
author: qwlake
categories: Nginx
tags: mtls ssl
---

nginx에서 ssl 인증서를 검증하기 위해서는 아래의 조건이 필요

1. 검증된 루트 인증서들의 모음이 필요하다.
2. `루트 인증서`, `체인 인증서`, `도메인 인증서` 가 하나의 인증서로 묶여야 한다.

## 검증된 루트 인증서를 구하는 방법

자바의 루트 인증서 모음을 가져온다.

`$JAVA_HOME\lib\security\cacerts`

해당 `cacerts` 파일을 pem 파일로 변환

```bash
echo -n changeit |keytool -importkeystore -srckeystore cacerts \
  -destkeystore cacerts.p12 -deststoretype PKCS12 -storepass changeit
openssl pkcs12 -in cacerts.p12 -out cacerts.pem -passin pass:changeit
```

cf. [Export cacerts JKS to PEM format](https://gist.github.com/ajarv/55170a8250128efe24d1b9caa927a816)

## 인증서들을 하나의 인증서로 합치는 방법

```bash
cat client.crt client_chain.crt clint_root.crt > new_client.crt
```

- 순서 중요
- 결과 파일에서 각 인증서마다 줄바꿈이 제대로 들어갔는지 체크 요망

## nginx 설정

```bash
server {
    listen                  443 ssl;

    access_log              /var/log/nginx/access.log;
    error_log               /var/log/nginx/error.log;

    ssl_certificate         /etc/ssl/server.crt;
    ssl_certificate_key     /etc/ssl/server.key;
    ssl_protocols           TLSv1.3;
    ssl_session_cache       shared:SSL:1m;
    ssl_session_timeout     10m;
    ssl_ciphers             HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers   on;

    ssl_client_certificate  /etc/nginx/client_certs/cacerts.pem;
    ssl_verify_client       optional_no_ca;
    ssl_verify_depth        2;

    location / {
        proxy_set_header    SSL_Client_Verify $ssl_client_verify;
        proxy_set_header    SSL_Client_Escaped_Cert $ssl_client_escaped_cert;

        proxy_pass          http://host.docker.internal:9091;
    }
}
```

Point

- 인증서를 포함하지 않아도 proxy_pass로 넘긴다. 단, ssl_client_verify 에는 `NONE` 이 들어간다.
- 인증서가 존재한다면 ssl_client_certificate에 주어진 루트 인증서를 기반으로 검증
    - 검증에 실패한다면 ssl_client_verify 에 `FAILED: {reason}` 형식으로 값 세팅
    - 검증 성공시 `SUCCESS` 세팅
- 어떠한 경우에도 ssl_client_escaped_cert 필드에는 도메인 인증서가 세팅된다. 단, 인증서가 존재하지 않는다면 NONE.

## 요청 날리기

```bash
curl https://localhost:8443/test \
  --key client.key \
  --cert new_client.crt \
  --pass {your_password_if_you_have_it}
```