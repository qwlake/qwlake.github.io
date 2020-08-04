---
layout: post
title: "Django 프로젝트 AWS에 배포하기"
subtitle: "+ Docker, Nginx, PostgreSQL"
author: qwlake
categories: Django AWS Docker
tags: Django AWS EC2 gunicorn Nginx PostgreSQL Docker Docker-compose
---

# 서론

본 프로젝트는 원래 멋쟁이 사자처럼 7기 중앙 해커톤의 결과물이다. 때문에 프로젝트는 배포 목적도 아니었고, 윈도우에서 작업하던 것이라 위와 같은 스택을 사용할 일이 없었다. 하지만 애써 만들어 놓은 사이트 묵히기도 아깝고, 코드만 달랑 보여주면 이게 어느 정도의 완성도를 갖춘 사이트인지, 무슨 사이트인지 모르기 때문에 AWS에 배포하기로 생각했다.

 - 프로젝트 GitHub: [https://github.com/qwlake/eot](https://github.com/qwlake/eot)
 - 배포한 사이트 주소: [http://resume-make.shop/](http://resume-make.shop/)

​

# 작업 전 상황

기존의 작업환경은 다음과 같았다.

 - 윈도우10 64bit
 - Django 2.2.2
 - Python 3.7
 - SQLight3(Django 기본 db)

또한 웹서버 세팅(Nginx)과 wsgi 세팅(gunicorn)이 안 되어 있다.

​

# 목표

1. 배포 시에 다중 접속을 받아낼 수 있는 서버 세팅으로 바꾸기
2. Django 기본 db인 SQLight를 걷어내고 PostgreSQL로 바꾸기
3. AWS에 프로젝트를 옮기기 위한 Docker 이미지 만들기
4. AWS에 Docker 이미지와 프로젝트 코드 올리고 배포

1번의 이유는 확장성 때문이다.

2번의 이유는 Docker를 한 번 써보고 싶기도 했고, AWS에서 처음부터 개발하는 것이 아닌, 기존 프로젝트를 옮기는 것이라 도커를 테스트해볼 수 있는 기회이기도 했다. *[도커와 컨테이너를 꼭 사용해야 하는 이유](http://www.ciokorea.com/news/39829)*

3번의 이유는 Django(장고)는 웹 어플리케이션이기 때문이다. 웹 어플리케이션과 웹 서버는 다르다. 웹 어플리케이션(예 Django)은 python(파이썬) 등의 언어를 이용해서 여러 가지 처리를 할 수 있지만, 외부로부터의 다중 접속을 받지 못한다. 웹 서버(예 Nginx)는 웹 어플리케이션과는 반대다. 여기서 하나가 더 필요한데 웹 서버로부터 받은 요청을 웹 어플리케이션에 전달할 수 있는 미들웨어(예 gunicorn)가 필요하다. *[웹 서버?, WSGI, 웹 프레임워크?](https://qkr0990.github.io/development/2017/08/29/note.html)*


# 시작하기 전에

윈도우에서 작업폴더를 하나 만들자.

이 안에 docker-compose.yml 과 Dockerfile 을 생성할 것이고, 장고 프로젝트 소스코드와 Nginx 설정 파일도 둘 것이다.

작업폴더를 생성했으면 그 안에 config 폴더와 src 폴더를 만들자.

그리고 **코드를 복사 붙여넣기 했는데 안 되면 직접 타이핑 하자**.

![image](https://user-images.githubusercontent.com/41278416/85567257-aee1a200-b66b-11ea-9c5d-95bbceb3923c.png)

> 코드 복붙이 안 되는 상황. 안 될 땐 직접 타이핑 하거나 깃허브에서 복사해 사용하자.

# 1. 배포 시에 다중 접속을 받아낼 수 있는 서버 세팅으로 바꾸기

앞서 말했듯 배포를 목적으로 하기 때문에 Nginx와 gunicorn을 설정해 줄 것이다.

config 폴더 안에 nginx 폴더를 생성하고 그 안에 `nginx.conf` 파일을 생성해준다. 이 파일은 Nginx의 설정파일이다.

​
### **config/nginx/nginx.conf**

```
server {
    listen 80;
    location / {
        proxy_set_header Host $host:$server_port;
        proxy_pass http://web:80;
        proxy_redirect off;
    }
    location /static/ {
        alias /src/.static_root/;
    }
    location /media/ {
        alias /src/media/;
    }
    access_log /var/log/nginx/8000_access.log;
    error_log /var/log/nginx/8000_error.log;
}
```

각 라인에 대한 설명

**listen 80;** nginx 컨테이너에서 외부로 향하는 포트

**location** 서버 호스트의 기본 주소 뒤에 붙는 위치 주소 정보(예, /는 root 디렉토리)

**proxy_set_header Host $host:$server_port;** 호스트의 주소와 포트를 고정시킨다. 만약 이 설정이 없으면 Django 프로젝트는 호스트 주소와 포트가 무엇인지 모르기 때문에 proxy_pass로 들어오는 주소를 사용한다.

**proxy_pass http://web:80;** 루트 디렉토리로 접근했을 경우 web 컨테이너의 80번 포트로 안내한다. web는 [docker-compose.yml](https://www.notion.so/e09f922dbed14a358abe00e235bd6ae5#ba7eb71442d043c39dd756baa1d7fafa) 에서 더 자세히 다룬다.

​

# 2. Django 기본 db인 SQLight를 걷어내고 PostgreSQL로 바꾸기

장고 프로젝트의 `settings.py`에서는 기본적으로 다음과 같이 세팅돼있다.

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': 'db',
    }
}
```

이것을 다음과 같이 PostgreSQL로 바꿔주자.

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'db',
        'USER': 'admin',
        'PASSWORD': 'admin',
        'HOST': 'db',
        'PORT': '5432',  # 5432는 PostgreSQL의 기본포트이다
    }
}
```

​

# 3. AWS에 배포를 쉽게 하기 위한 Docker 이미지 만들기

AWS에 올릴 도커 이미지를 만들기 위해서는 Dockerfile 하나 만으로는 안 된다. 왜냐하면 dockerhub에 있는 이미지 중에는 내가 정확히 원하는 이미지가 없다.

나는 Django 프레임워크에 웹 서버 세팅(gunicorn, Nginx)을 붙이고 싶고, (내 프로젝트의 경우) LibreOffice를 사용해야 하고 추후 확장성을 위해서 ubuntu bash 쉘을 사용할 수 있으면 좋겠다. 그리고 PostgreSQL은 덤이다.

**Ubuntu 이미지를 다운 받아 거기에 다 설치하면 되는 거 아닌가?**

물론 가능하다. 하지만 다른 것들도 이미지가 존재하는데 굳이 우분투 이미지에 설치하는 시간까지 늘일 필요는 없다. 게다가 관리상 힘들다.

그렇다면 여러 개의 이미지를 어떻게 관리할까? 해답은 docker-compose다. *[docker-compose로 개발 환경 구성하기](https://www.44bits.io/ko/post/almost-perfect-development-environment-with-docker-and-docker-compose#%EB%8F%84%EC%BB%A4-%EC%BB%B4%ED%8F%AC%EC%A6%88%EB%A1%9C-%EA%B0%9C%EB%B0%9C-%ED%99%98%EA%B2%BD-%EA%B5%AC%EC%84%B1%ED%95%98%EA%B8%B0)*

나의 경우에는 우분투 이미지를 베이스로 Django, Nginx, PostgreSQL 이미지를 얹었다. 작업폴더의 src 폴더에 장고 프로젝트를 넣는다. 이후 작업폴더에 다음의 두 파일을 만든다.

### **docker-compose.yml**

주석으로 자세하게 설명해 놓았다. 위에서부터 차근차근 보길 바란다. 단, 주석을 달고 실행해봤는데 실행이 안 된다. syntax 오류인 듯한데 어느 부분이 잘못됐는지 모르겠다. **코드 복사가 필요하면 다음 링크에서 복사해 사용하자.** [https://github.com/qwlake/eot/blob/master/docker-compose.yml](https://github.com/qwlake/eot/blob/master/docker-compose.yml)

```docker
version: "3"                                                # docker-compose 버전이다
services:                                                   
    nginx:                                                  # 서비스명
        nginx:1.17.10                                       # nginx:1.17.10 이미지를 사용
        container_name: ng01                                # 컨테이너명
        ports:                                              # 포트 개방 "hostPort:dockerPort"으로 포워딩한다
            - "80:80"                                       
        volumes:                                            # 볼륨 마운트. 호스트의 파일을 사용하려면 필요하다
            - ./src:/src                                    # 소스코드 마운트
            - ./config/nginx/nginx.conf:/etc/nginx/conf.d/default.conf # nginx 세팅 마운트
        depends_on:                                         # 'web'컨테이너가 구동되어야만 실행된다
            - web                                           

    web:                                                    
        build: .                                            # 메인으로 빌드될 Dockerfile을 적는다. 지금은 현재 폴더에 있다고 알린다
        container_name: dg01                                
        command: bash -c "                                  # Dockerfile을 빌드 후 실행했을때 자동으로 입력되는 명령어다
            python3 manage.py collectstatic --no-input &&   # collectstatic을 실행했을때 기존에 이미 파일이 있는 경우 그래도 실행하냐고 물어본다. 이 때 입력을 방지하기 위해 --no-input 옵션을 사용한다
            python3 manage.py makemigrations &&             
            python3 manage.py migrate &&                    
            gunicorn project.wsgi:application -b 0:80"      # Django의 일반적인 runserver가 아닌 gunicorn을 사용해서 서버를 구동한다. 포트는 0.0.0.0:80으로 바인딩한다. 모든 아이피에서의 접속을 허용하고 80번 포트를 연다는 의미
        depends_on:                                         
            - db                                            
        volumes:                                            
            - ./src:/src                                    

    db:                                                     
        image: postgres:12.2                                
        container_name: ps01                                
        environment:                                        # PostgreSQL의 환경설정이다.
            POSTGRES_DB: db                                 
            POSTGRES_USER: admin                            
            POSTGRES_PASSWORD: admin
```

​

### **Dockerfile**

```docker
### 우분투 초기 설정 ###
# ubuntu 이미지 18.04 버전을 베이스 이미지로 한다
FROM ubuntu:18.04

# 우분투에서 다운로드 속도가 느리기 때문에 다운로드 서버를 바꿔주었다
RUN sed -i 's@archive.ubuntu.com@mirror.kakao.com@g' /etc/apt/sources.list

# apt 업그레이트 및 업데이트
RUN apt-get -y update && apt-get -y dist-upgrade

# apt-utils dialog : 우분투 초기 설정 / libpq-dev : PostgreSQL 의존성
RUN apt-get install -y apt-utils dialog libpq-dev

# 리브레오피스 설치 (내가 만든 사이트 내부 구동에 필요하다. 여러분은 굳이 다운 받지 않아도 된다.)
RUN apt-get install -y libreoffice

# 리브레오피스 언어 패치
RUN apt-get install -y language-selector-gnome fonts-noto-cjk

# pdf를 img로 변환하기 위한 패키지 (내가 만든 사이트 내부 구동에 필요하다. 여러분은 굳이 다운 받지 않아도 된다.)
RUN apt-get install poppler-utils

### python 설치 ###
# pip dev 설치
RUN apt-get install -y python3-pip python3-dev

# 파이썬에서 콘솔 출력이 느릴 경우 다음과 같이 환경 변수를 설정해준다.
ENV PYTHONUNBUFFERED=0

# Django 기본 언어를 한국어로 설정하면 파이썬 기본 인코딩과 충돌되어 한글 출력, 입력시에 에러가 난다.
# 따라서 파이썬 기본 인코딩을 한국어를 사용할 수 있는 utf-8으로 설정한다.
ENV PYTHONIOENCODING=utf-8

# pip setuptools 업그레이드
RUN pip3 install --upgrade pip
RUN pip3 install --upgrade setuptools

### 실행 환경 구축 ###
# 컨테이너 내부에 config(설정파일)가 들어갈 폴더 생성
RUN mkdir /config

# 호스트에 있는 requirements.txt 파일을 컨테이너 내부로 복사해준다.
ADD /config/requirements.txt /config/

# requirements.txt에 있는 파이썬 패키지 설치
RUN pip3 install -r /config/requirements.txt

### 작업 디렉토리 ###
# Django 소스코드가 들어갈 폴더 생성
RUN mkdir /src;

# 작업 디렉토리 src로 변경
WORKDIR /src
```

Dockerfile 같은 경우는 명령어 뒤에 주석을 달면 빌드 시에 에러가 난다.

docker-compose.yml을 빌드 하면 Dockerfile을 가져와 베이스로 삼아 컨테이너를 구성하고 그 위에 nginx, postgreSQL을 얹는 개념이다.

​

Dockerfile에는 파이썬 설치만 적혀 있고 내가 필요한 파이썬 패키지는 적어 놓지 않았다. 내가 필요한 파이썬 패키지는 cmd 창이나 bash 상에서 `pip freeze > requirements.txt` 명령어로 `requirements.txt` 파일에 쉽게 저장할 수 있다. 그 안에 내가 앞으로 추가적으로 필요할 예정인 패키지를 추가한다.

```python
gunicorn==20.0.4
psycopg2==2.8.4
```

gunicorn은 도커 이미지로 붙이는 것이 아니라 파이썬 패키지로 설치한다. 만약 지금 당신의 환경에 Django가 없다면 다음과 같이 장고도 추가해 주도록 하자.

```python
Django==2.2.3
gunicorn==20.0.4
psycopg2==2.8.4
```

이렇게 준비한 `requirements.txt` 파일을 config폴더를 생성하고 그 안에 위치시킨다. 따라해보자

docker-compose build위 명령으로 docker-compose를 빌드 하면 이미지가 3개 생성된다.

![image](https://user-images.githubusercontent.com/41278416/85567300-b6a14680-b66b-11ea-81d0-9d6948296d2e.png)
> 생성된 이미지들

docker-compose up위 명령어로 생성된 이미지를 기반으로 컨테이너들을 한꺼번에 생성하고 실행한다.

![image](https://user-images.githubusercontent.com/41278416/85567311-b86b0a00-b66b-11ea-9232-0abcd79712bd.png)
> 생성되어 실행 중인 컨테이너들

여기까지 진행한다면 도커 이미지 생성은 끝이다.

잘 따라 했다면 다음과 같은 구성이 될 것이다.


![image](https://user-images.githubusercontent.com/41278416/85567316-b99c3700-b66b-11ea-8644-36a8be4f6d6e.png)

---

다음은 내가 주로 쓰는 유용한 명령어다.

`docker-compose up --builddocker-compose` 빌드와 실행을 한꺼번에 할 수 있다.

`docker-compose down` 실행 중인 docker-compose 컨테이너들을 종료시키기

`docker-compose down --volumedocker-compose` 종료와 더불어 각 컨테이너에 마운트 된 볼륨도 삭제한다.

`docker ps -a` 모든(종료된, 실행 중인) 개별 컨테이너 출력

`docker images` 도커 이미지 출력

`docker exec -it <컨테이너명or컨테이너id> <명령어>` 실행 중인 컨테이너에 접속하여 명령어를 전달한다.

나는 주로 `docker exec -it <ubuntu 컨테이너명> bash`를 사용해 django가 실행 중인 우분투 컨테이너에 접속한다.

`-it` 옵션 없이 실행하면 명령어만 전달하고 끝이다. 컨테이너에 접속하여 무언가 하려면 `-it` 옵션으로 해당 컨테이너에 접속하여야 한다.

실행 중인 컨테이너를 종료하지 않고 빠져나오려면 다음의 키를 누르면 된다.

`ctrl + p + q` 컨테이너에 접속 중일 때 컨테이너를 종료하지 않고 빠져나온다.

이외에도 유용한 명령어가 많지만 내가 주로 쓰는 것들은 이상과 같다.

더 알아보고 싶으면 `docker --help` 나 `docker-compose --help`를 쳐보도록 하자.

​

# 4. AWS에 Docker 이미지와 프로젝트 코드 올리고 배포

방금까지 만든 이미지에는 장고 소스코드(src 폴더)가 포함되지 않는다.

Dockerfile의 `ADD /config/requirements.txt /config/` 처럼 src 폴더도 이미지 내부로 복사해준다면 코드도 이미지화되어 버전 관리를 할 수 있다. 다만 코드를 이미지화시키는 경우에는 ADD 명령어 말고 보통 COPY를 쓴다고 한다.

아무튼 소스코드가 이미지 안에 없는 관계로 AWS에 배포할 때에는 이미지와 소스코드 모두를 옮겨 주어야 한다. 이때 사용하는 방법으로는 scp가 있다. 

*[AWS scp,로컬에서 EC2 서버로 파일 전송](https://www.sallys.space/blog/2017/11/28/aws-scp/)*

하지만 나는 작업폴더를 github에 올려서 관리할 것이기 때문에 깃으로 옮기도록 하겠다. 그럼 먼저 AWS에 EC2 인스턴스를 생성하자.

​

### **AWS EC2 인스턴스 생성**

a. AWS(Amazon Web Service)에 회원가입 후 로그인한다.

b. 로그인 후 AWS 상단의 네비게이션바에서 지역(Region)을 서울로 바꾼다.

![image](https://user-images.githubusercontent.com/41278416/85567330-bbfe9100-b66b-11ea-9768-d04b6d2bb565.png)

c. EC2 인스턴스 대시보드로 이동

![image](https://user-images.githubusercontent.com/41278416/85567344-be60eb00-b66b-11ea-9861-2aec525b11d8.png)

d. 인스턴스 시작

![image](https://user-images.githubusercontent.com/41278416/85567359-c02aae80-b66b-11ea-8c4b-066000aa639e.png)

f. `Ubuntu Server 18.04 LTS (HVM), SSD Volume Type` 선택

![image](https://user-images.githubusercontent.com/41278416/85567377-c28d0880-b66b-11ea-90b4-db646b1b19a7.png)

g. 프리티어 사용 가능한 `t2.micro` 선택 후 `검토 및 시작` 클릭

![image](https://user-images.githubusercontent.com/41278416/85567384-c456cc00-b66b-11ea-9ac2-92d7f99b2656.png)

h. 다음 페이지에서 우측 하단의 `시작하기` 버튼 누르고, `새 키 페어 생성` 클릭

![image](https://user-images.githubusercontent.com/41278416/85567673-0849d100-b66c-11ea-8015-294b3f47d99e.png)

i. `키 페어 이름`을 적절히 적은 후 `키 페어 다운로드`를 한 후에 `인스턴스 시작` 클릭.

**이 때 다운받은 키 페어가 있어야 인스턴스에 접속할 수 있다.**

![image](https://user-images.githubusercontent.com/41278416/85567678-0a139480-b66c-11ea-85a7-539c35b6bb62.png)

---

### Elastic IP (탄력적 IP) 할당

인스턴스 생성은 끝났다. 이 인스턴스에는 자동으로 IP 주소가 부여되는데, 이는 인스턴스를 재부팅할 때마다 바뀐다. 때문에 고정 IP 주소를 할당해 주겠다. 주의해야할 점이 있는데 탄력적 IP를 생성해 두고 **인스턴스에 연결하지 않으면 요금이 부과**된다.

a. EC2 대시보드에서 좌측 메뉴에 `탄력적 IP` 클릭

![image](https://user-images.githubusercontent.com/41278416/85567690-0b44c180-b66c-11ea-98bc-cd4dc4906e33.png)

b. `탄력적 IP 주소 할당` 클릭

![image](https://user-images.githubusercontent.com/41278416/85567697-0c75ee80-b66c-11ea-8369-23cab4119ec8.png)

c. `Amazon의 IPv4 주소 풀` 체크 후 우측 하단 `할당` 클릭

![image](https://user-images.githubusercontent.com/41278416/85567710-0e3fb200-b66c-11ea-81c0-061560096854.png)

d. 할당 후에 다시 탄력적 IP 홈으로 돌아오면 Actions의 `탄력적 IP 주소 연결` 클릭

![image](https://user-images.githubusercontent.com/41278416/85567716-0f70df00-b66c-11ea-9f0e-c640e1668067.png)

e. 인스턴스를 방금 생성한 인스턴스로 선택하고, 프라이빗 IP 주소도 선택합니다. 이후 우측 하단의 연결 클릭

![image](https://user-images.githubusercontent.com/41278416/85567722-10a20c00-b66c-11ea-8247-c3fe905b962b.png)

---

### **보안 그룹 설정**

탄력적 IP 연결은 끝이다. 이제 외부에서 인스턴스로 접근하기 위한 80번 포트를 열어주겠다. 기본적으로 22번 포트(ssh)는 활성화 되지만 웹사이트를 위한 기본 포트(80번은)는 막혀있다. EC2 대시보드의 좌측 메뉴에서 `보안 그룹`으로 이동한다.

a. 아까 EC2 인스턴스를 만들면서 같이 생성된 보안 그룹을 클릭해 들어간다.

![image](https://user-images.githubusercontent.com/41278416/85567733-126bcf80-b66c-11ea-9edb-79652f61f4af.png)

b. `Edit inbound rules`를 클릭

![image](https://user-images.githubusercontent.com/41278416/85567742-139cfc80-b66c-11ea-831e-839d3132f4bb.png)

c. `Add rule` 클릭

![image](https://user-images.githubusercontent.com/41278416/85567751-14ce2980-b66c-11ea-9403-1c8f27000fbc.png)

d. Type 속성의 `Custom TCP`를 눌러서 `http`를 타이핑해서 검색 후 클릭

![image](https://user-images.githubusercontent.com/41278416/85567761-1697ed00-b66c-11ea-8b48-c40a9b010771.png)

e. Source 속성에서 `0.0.0.0/0` 클릭

![image](https://user-images.githubusercontent.com/41278416/85567766-17c91a00-b66c-11ea-9752-f96f40449791.png)

f. 우측 하단의 `Save rules`를 눌러 종료

![image](https://user-images.githubusercontent.com/41278416/85567777-1992dd80-b66c-11ea-8d75-7eed0b80a318.png)

---

### **SSH 사용하여 인스턴스 접속**

AWS 설정은 끝났다. 생성한 인스턴스에 접속하자. 접속은 먼저 SSH로 할건데,

나중에 jupyter-notebook를 설치해서 웹에서 터미널로 접속하거나([https://youtu.be/LoYpXoBJPMc](https://youtu.be/LoYpXoBJPMc)),

ubuntu-desktop을 설치해서 gui 환경으로 접속할 수도 있다([Ubuntu Server에 Desktop GUI 설치하기](https://snowdeer.github.io/linux/2017/08/04/install-gui-desktop-on-ubuntu/)).

SSH 접속을 위해선 `ppk`(Putty Private Key) 파일이 필요하다. 하지만 우리가 아까 인스턴스를 생성하면서 다운로드 받은 파일은 `pem` 파일이다. 이 `pem` 파일을 `ppk` 파일로 바꿔주자.

a. `PuTTYgen` 실행 (putty 설치하면 같이 설치된다. [putty 다운로드](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html))

b. `Load` 클릭

![image](https://user-images.githubusercontent.com/41278416/85567783-1ac40a80-b66c-11ea-9088-a30cd36d01e8.png)

c. All Files로 검색 설정을 바꾼 후에 아까 다운로드 받은 `pem` 파일 선택 후 열기

![image](https://user-images.githubusercontent.com/41278416/85567797-1dbefb00-b66c-11ea-9a79-04982aac9b39.png)

d. `Save private key`클릭

![image](https://user-images.githubusercontent.com/41278416/85567817-23b4dc00-b66c-11ea-9652-6e4ca6a90fab.png)

e. PuTTY 실행. 관리자 권한으로 실행하지 않으면 권한 에러가 날 수 있다.

f. 좌측의 카테고리에서 Connection > SSH > Auth 클릭 후 Browse 클릭 후에 방금 생성한 `ppk` 파일 열기

![image](https://user-images.githubusercontent.com/41278416/85567829-257e9f80-b66c-11ea-909e-dcb09f5d1864.png)

g. 다시 카테고리의 Session으로 이동해 `Host Name`에 아까 탄력적 IP에서 생성한 IP를 적고 하단의 Open 클릭

![image](https://user-images.githubusercontent.com/41278416/85567841-27e0f980-b66c-11ea-9854-bf0bc4879384.png)

h. 예를 눌러 접속하고 `ubuntu`로 로그인한다

![image](https://user-images.githubusercontent.com/41278416/85567853-29aabd00-b66c-11ea-9d51-1556bedba89c.png)

i. 접속한 모습

![image](https://user-images.githubusercontent.com/41278416/85567865-2c0d1700-b66c-11ea-94b7-d47602f623dd.png)

---

### **샘플 Django 프로젝트 실행**

접속 후 docker를 설치하고 git에서 프로젝트를 다운로드하여 실행해보자. 제일 먼저 접속된 SSH상에서 다음의 명령어를 입력한다. 혹시 에러가 난다면 다시 입력해보자.

```
sudo apt-get update
sudo apt-get -y dist-upgrade
sudo apt-get install docker
sudo apt-get -y install docker-compose
```

dist-upgrade 중간에 이런 화면이 두 번 나올텐데, 나오면 엔터를 눌러 주도록 하자

![image](https://user-images.githubusercontent.com/41278416/85567882-30393480-b66c-11ea-9f1e-5cf0c35f6bff.png)

모두 끝나면 다음 명령어로 깃허브에서 프로젝트를 다운 받는다. 이 주소는 내가 만든 테스트 장고 프로젝트로 여러분이 그대로 사용해도 된다.

```
sudo git clone https://github.com/qwlake/example-docker-django.git
```

프로젝트 폴더로 이동

```
cd example-docker-django
```

docker 이미지 생성 및 컨테이너 실행 (웹 서버 시작)

```
sudo docker-compose up --build
```

![image](https://user-images.githubusercontent.com/41278416/85567897-33342500-b66c-11ea-84ae-b126bca0e025.png)

이제 AWS에서 받은 탄력적 IP(내 경우는 [http://15.165.211.41/](http://15.165.211.41/))로 들어가면 위와 같은 화면을 볼 수 있다. 페이지는 간단한 Bootstrap 예제를 넣어놨다. 여러분 마음대로 수정해 보도록 하자.

# Reference

 - [https://inma.tistory.com/125](https://inma.tistory.com/125)
 - [https://ruddra.com/posts/docker-django-nginx-postgres/](https://ruddra.com/posts/docker-django-nginx-postgres/)