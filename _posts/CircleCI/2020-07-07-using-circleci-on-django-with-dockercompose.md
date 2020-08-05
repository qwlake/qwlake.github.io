---
layout: post
title: "[CircleCI] 장고 프로젝트에 Circle CI 적용하기"
subtitle: "+ Django, docker-compose"
author: qwlake
categories: CircleCI
tags: CircleCI Django Docker docker-compose
---

# 서론

지난번에 Django 프로젝트에 pytest를 붙인 적이 있다 ([[Django] pytest를 사용한 장고 테스트 환경 구축](https://qwlake.github.io/django/2020/06/05/building-django-test-environment/)). 이를 이용한 지속적 통합(CI) 환경을 구축해 보고자 한다.

# CI(Continuous Integration)란?

CI가 눈에 익지 않다면 이것은 어떤가. CI/CD. 많이 봤을 것이다. CI/CD는 Redhat의 DevOps 가이드에서 다음과 같이 말하고 있다.

> CI/CD는 애플리케이션 개발 단계를 자동화하여 애플리케이션을 보다 짧은 주기로 고객에게 제공하는 방법입니다.

그 중에서 CI는 다음을 의미한다.

> CI/CD의 "CI"는 개발자를 위한 자동화 프로세스인 지속적인 통합(Continuous Integration)을 의미합니다.

따라서 "지속적인 통합"을 자동화한다는 것인데, 이는 Git에서 영감을 받을 수 있다. 우리는 보통 프로젝트를 수정한 후에 이를 저장하기 위해 형상관리 툴인 Git 등에 Push한다. 이 때, 새로 올라온 버전을 Git에 연동되어 있는 CI 툴이 자동으로 빌드, 테스트 후에 병합한다.

![image](https://user-images.githubusercontent.com/41278416/89420540-d0a36e00-d76d-11ea-9139-be7ebc35020e.png)

> [https://www.redhat.com/ko/topics/devops/what-is-ci-cd](https://www.redhat.com/ko/topics/devops/what-is-ci-cd)

# Circle CI

주로 사용하는 CI 툴에는 Circle CI, Travis CI 등이 있다. 나는 Circle CI를 사용하기로 했다. Circle CI는 유료 버전도 존재하지만, 가볍게 쓰기에는 무료 버전도 괜찮다.

## 시작

Circle CI 회원가입 후에 깃허브 프로젝트를 Circle CI에 등록하자. [https://app.circleci.com/](https://app.circleci.com/) 이 사이트를 방문에서 Projects 탭으로 가면 깃허브에 있는 여러 레포지토리가 뜰 것이다. 그 중 CI를 원하는 레포지토리의 `Set Up Project` 버튼을 누른다.

![image](https://user-images.githubusercontent.com/41278416/89420517-c8e3c980-d76d-11ea-81b6-3bce1872cd69.png)


누르면 아래와 같이 나올텐데, 여기서 CI를 위한 설정을 해줄 수 있다. 사진에 보이는 Hello World를 클릭하면 빌드할 수 있는 여러가지 언어가 나오고 클릭하면 샘플을 보여준다. 샘플을 토대로 수정해서 바로 `Add Config` 버튼을 눌러 시작해도 되고, `Use Existing Config` 버튼을 통해 원하는 파일을 올려도 된다.

![image](https://user-images.githubusercontent.com/41278416/89420523-ca14f680-d76d-11ea-95e8-e595d34d1222.png)


`Add Config` 버튼을 누르면 깃허브에 config 파일을 추가한 `circleci-project-setup` 브랜치가 생성되고, 다음과 같이 빌드가 시작된다. 

![image](https://user-images.githubusercontent.com/41278416/89420525-cb462380-d76d-11ea-9d56-ee7daaa84768.png)


## Config 설정하기

`config.yml` 을 설정하기 위해서는 다음의 가이드를 보자. 잘 설명되어 있다.

[https://circleci.com/docs/2.0/sample-config/](https://circleci.com/docs/2.0/sample-config/)

```yaml
version: 2.1

# Define the jobs we want to run for this project
jobs:
  build:
    docker:
      - image: circleci/<language>:<version TAG>
    steps:
      - checkout
      - run: my-command
  test:
    docker:
      - image: circleci/<language>:<version TAG>
    steps:
      - checkout
      - run: my-command

# Orchestrate our job run sequence
workflows:
  build_and_test:
    jobs:
      - build
      - test:
          requires:
            - build
```

`jobs` 에서는 Circle CI에서 무엇을 실행할 것인지 작업을 지정해준다. 위 예제에서는 `build` 와 `test` 를 작업으로 넣어줬다. `workflows` 에서는 앞서 지정한 작업을 어떻게 실행할건지를 명시한다.

서론에서 언급한 pytest를 `test` 부분에 넣으면, Circle CI를 위한 테스트 환경을 따로 구성하지 않고 기존 시스템을 그대로 사용할 수 있다.

### docker-compose

단, docker-compose를 빌드하기 위해서는 신경 써야할 부분이 있다. 보통 Circle Ci에서 레포지토리를 빌드하는 방법은 해당 프로젝트의 docker image를 지정해주고, 해당 image를 통해 빌드하는 방법을 사용한다. 그런데 docker-compose로 구성한 레포지토리는 docker image를 지정할 수가 없기 때문에 Executor Type으로 docker가 아닌 machine으로 지정해 주어야 한다. [https://circleci.com/docs/2.0/executor-types/#using-machine](https://circleci.com/docs/2.0/executor-types/#using-machine)

# 끝