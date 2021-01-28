---
layout: post
title: "[MSA] 크롤링 모듈 MS(Micro Service)로 포팅해서 gRPC 서버로 만들기"
subtitle: "gRPC 예제 링크와 gRPC 서버, gRPC 클라이언트 실행방법"
author: qwlake
categories: MSA
tags: MSA gRPC Python
---

# gRPC

[https://grpc.io/docs/languages/python/quickstart/](https://grpc.io/docs/languages/python/quickstart/)

[https://grpc.io/docs/languages/python/basics/](https://grpc.io/docs/languages/python/basics/)

```bash
python -m grpc_tools.protoc -I ../protos --python_out=. --grpc_python_out=. ../protos/crawling.proto
```

# proto3

`proto3` 란, `protocol buffer` 의 축약어에 버전(3)을 붙인 단어이다. gRPC API를 정의하기 위해서는 proto3를 사용해야 한다. proto3는 다음의 특징을 갖고 있다.

> 프로토콜 버퍼는 데이터 구조 스키마와 프로그래밍 인터페이스를 정의할 수 있는 인터페이스 정의 언어(IDL)로서 언어 중립적일 뿐만 아니라 플랫폼 중립적이기도 합니다. 이 IDL은 바이너리 통신 형식과 텍스트 통신 형식을 모두 지원할 뿐만 아니라 플랫폼에 따라 여러 가지 다른 통신 프로토콜과 호환됩니다.

[https://developers.google.com/protocol-buffers/docs/proto3](https://developers.google.com/protocol-buffers/docs/proto3)

# 실행

### 서버

1. 도커 빌드(첫 실행 시)
```bash
docker build --tag grpc-crawling .
```

2. 도커 실행
```bash
docker run -it \
-v /$(pwd)/crawling:/home/crawling/ \
-p 50053:50053 \
grpc-crawling bash
```

3. 실행
```bash
python server.py
```

### 클라이언트(터미널 하나 더 띄워서)

1. 가상환경 설치(첫 실행 시)
```bash
python -m venv .env
```

2. 가상환경 켜기
```bash
source .env/Scripts/activate
```

3. 패키지 설치(첫 실행 시)
```bash
pip install -r config/requirements.txt
```

4. 폴더 이동
```bash
cd crawling
```

5. 실행
```bash
python client.py
```

# Issue

**도커 toolbox 버그**

[https://stackoverflow.com/questions/35315996/how-do-i-mount-a-docker-volume-while-using-a-windows-host](https://stackoverflow.com/questions/35315996/how-do-i-mount-a-docker-volume-while-using-a-windows-host)

**billiard.Manager.dict() 에 딕셔너리 벨류로 못 넣음. <class 'billiard.managers.DictProxy'>**

[https://stackoverflow.com/questions/49210359/update-dictproxy-from-multiple-processes-does-not-work](https://stackoverflow.com/questions/49210359/update-dictproxy-from-multiple-processes-does-not-work)

# python ORM

[http://sanghun.xyz/2016/11/python에서-postgres-db-연결해서-쿼리-조회하기/](http://sanghun.xyz/2016/11/python%EC%97%90%EC%84%9C-postgres-db-%EC%97%B0%EA%B2%B0%ED%95%B4%EC%84%9C-%EC%BF%BC%EB%A6%AC-%EC%A1%B0%ED%9A%8C%ED%95%98%EA%B8%B0/)

[https://www.learndatasci.com/tutorials/using-databases-python-postgres-sqlalchemy-and-alembic/](https://www.learndatasci.com/tutorials/using-databases-python-postgres-sqlalchemy-and-alembic/)

# ~~데이터베이스~~

```bash
docker run -it \
-v /$(pwd)/config:/run/secrets \
-p 5432:5432 \
-e POSTGRES_DB=postgres \
-e POSTGRES_USER=postgres \
-e POSTGRES_PASSWORD_FILE=/run/secrets/postgres-passwd \
postgres:12.2 bash
```