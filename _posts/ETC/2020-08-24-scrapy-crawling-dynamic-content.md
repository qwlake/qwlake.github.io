---
layout: post
title: "[Scrapy] xpath, css selector 안 먹는 현상 해결"
subtitle: "Dynamic content 크롤링 하기"
author: qwlake
categories: ETC
tags: Scrapy scrapy-splash splash javascript dom xpath css-selector
---

# 서론

Scrapy를 사용하다보면 가끔 xpath와 css selector가 제대로 동작하지 않는 현상을 볼 수 있다. 분명 크롬에서 개발자모드를 켜서 가져온 경로는 제대로된 경로가 맞는데, Scrapy에서는 먹히지 않는다. 이는 우리가 크롬에서 보는 화면이 동적 콘텐츠(Dynamic contents)가 포함된 화면이기 때문이다.

# Dynamic content란?

![image](https://user-images.githubusercontent.com/41278416/91658496-9d04fb00-eb03-11ea-9725-5e9cb3805c9f.png)

우리가 웹 브라우저(크롬)을 통해 웹 사이트를 볼 때에는 동적 콘텐츠가 렌더링된 화면을 보는 것이다. 예를 들어 브라우저가 로드하는 원본 html 파일에는 `<div></div>` 만 존재한다면, Javascript를 통해 `<div>` 태그 안에 내용을 채워 넣는 것이다. 이 때 Javascript를 통한 렌더링이 되기 전 원본 html 파일이 위 사진에서 1번 단계이고, 브라우저에서 보는 화면은 2번 단계이다.

그런데 브라우저에서 **개발자 모드를 켜고 xpath를 복사한다면 2번 단계 기준으로 경로**가 나온다. 하지만 **Scrapy의 경우에는 원본 html 파일, 즉 1번 단계의 파일**을 가져오기 때문에, 내가 구한 **xpath 경로와 Scarpy가 가져온 파일과는 서로 맞지 않는 것**이다.

이와 관련해서 유의 사항으로 Scrapy 공식 문서에서 다음과 같이 안내하고 있다. [https://docs.scrapy.org/en/latest/topics/developer-tools.html#caveats-with-inspecting-the-live-browser-dom](https://docs.scrapy.org/en/latest/topics/developer-tools.html#caveats-with-inspecting-the-live-browser-dom) → 이와 같이 개발자 모드에서 Javascript 렌더링을 수동으로 off 시키고 xpath 등의 경로를 구할 수도 있겠지만, Dynamic content가 필요한 상황에서는 무의미하다.

# Dynamic content를 가져오기 위한 방법

따라서 Scrapy 공식 문서에서는 Dynamic content를 크롤링 하기 위해 여러 방안을 제시하고 있다. 먼저 가장 좋은 방법은 Data Source를 구하는 것이다.

## Finding Data Source

[https://docs.scrapy.org/en/latest/topics/dynamic-content.html#topics-finding-data-source](https://docs.scrapy.org/en/latest/topics/dynamic-content.html#topics-finding-data-source)

이 방법은 원본 위치에서 직접 데이터를 가져오는 것이다.  이 중에도 방법이 3가지로 나뉜다.

1. 가져오려는 데이터가 이미지, pdf 처럼 text-based 데이터가 아닌 경우
2. 데이터가 Javascript 코드로 정의되어 있거나, 외부 리소스에서 받아오는 경우
3. 그 외(크롤링 하려는 URL에서 모두 가져온 경우)

1번은 내가 어떤 데이터를 크롤링 하려는지에 따라 알게될 것이고, 2번과 3번이 어떤 기준일지 모를텐데 이는 [wgrep](https://github.com/stav/wgrep)와 같은 도구를 사용하려 해당 리소스의 URL을 찾을 수 있다.

결론적으로 1번과 2번의 경우는 해당 요청의 헤더값을 [재현](https://docs.scrapy.org/en/latest/topics/dynamic-content.html#topics-reproducing-requests)해서 Data Source를 구할 수 있다. 3번의 경우에는 원하는 데이터가 `<script>` 태그 안에 정의되어 있는 경우에는 [Javascript 코드 크롤링](https://docs.scrapy.org/en/latest/topics/dynamic-content.html#topics-parsing-javascript) 을 사용하면 된다. 단, 이 때 요청 헤더를 브라우저에서 보낼 때와 같게 해주어야 성공률이 높아진다.

## Pre-rendering JavaScript

[https://docs.scrapy.org/en/latest/topics/dynamic-content.html#pre-rendering-javascript](https://docs.scrapy.org/en/latest/topics/dynamic-content.html#pre-rendering-javascript)

Finding Data Source로 원하는 데이터를 가져올 수 있다면 좋겠지만, 데이터를 가져올 수 없거나, 그 과정이 귀찮거나, 크롤링하는데 걸리는 시간이 크게 중요하지 않다면 본 섹션에서 소개하는 방법을 사용하는 것이 확실한 방법이다.

위의 방법은 Javascript 렌더링을 피하고 원하는 데이터를 가져오는 것이었다면, 본 방법은 Javascript 렌더링이 된 후의 html을 가져와 크롤링하는 것이다. 따라서 자원(주로 메모리)과 시간 소모량이 비약적으로 증가하는 것을 감안해야 한다.

### scrapy-splash

[https://github.com/scrapy-plugins/scrapy-splash](https://github.com/scrapy-plugins/scrapy-splash)

기존에 구축되어 있는 Scrapy 크롤링 시스템에서 약간의 변경만으로 적용 가능하다. Javascript 렌더링을 위해 [Splash](https://github.com/scrapinghub/splash)를 사용하고, Splash가 가져온 Javascript rendered DOM 객체를 scrapy-splash가 받아서 Scrapy에 제공한다. Scrapy에서는 크롤링 요청에 따른 응답을 가져올 middleware로 기존의 것 대신에 scrapy-splash를 선택해 사용하는 구조이다. 나는 이 방법을 채택했고, 사용법은 위 링크에 자세히 나와 있다.

### headless-brwoser

[https://www.selenium.dev/](https://www.selenium.dev/) [https://github.com/clemfromspace/scrapy-selenium](https://github.com/clemfromspace/scrapy-selenium)

정말 최후의 방법이다. Selenium같은 헤드리스 브라우저를 사용해 크롤링하는 것이다. 기능이 강력하고 우리가 보는 화면과 거의 유사한 환경에서 데이터를 가져올 수 있다는 장점이 있다. 하지만 자원가 시간 소모량이 더욱 증가하니 되도록이면 피하는 것이 좋다.

# References

[https://docs.scrapy.org/en/latest/topics/developer-tools.html#caveats-with-inspecting-the-live-browser-dom](https://docs.scrapy.org/en/latest/topics/developer-tools.html#caveats-with-inspecting-the-live-browser-dom)

[https://docs.scrapy.org/en/latest/topics/dynamic-content.html](https://docs.scrapy.org/en/latest/topics/dynamic-content.html)

[https://github.com/scrapy-plugins/scrapy-splash](https://github.com/scrapy-plugins/scrapy-splash)

[https://blog.scrapinghub.com/2015/03/02/handling-javascript-in-scrapy-with-splash](https://blog.scrapinghub.com/2015/03/02/handling-javascript-in-scrapy-with-splash)