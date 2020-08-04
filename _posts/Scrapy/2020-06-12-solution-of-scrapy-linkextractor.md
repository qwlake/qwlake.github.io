---
layout: post
title: "[Scrapy] 스크래피 LinkExtractor 모든 링크 가져오지 못하는 버그"
subtitle: "중복된 링크 거르지 않고 모두 가져오기"
author: qwlake
categories: Scrapy
tags: Scrapy LinkExtractor Bug
---

# 서론

Scrapy를 사용해 크롤링을 하다 보면 링크를 크롤링하는 경우가 많다. 이 때 a태그 안의 href 값이 상대 주소로 되어 있는 경우가 있는데(ex ./mypage/payments) 이를 절대 주소로 바꿔주는 기능이 `LinkExtractor`이다.

# LinkExtractor 구조 뜯어보기

`LinkExtractor`를 사용할 때에는 다음의 경로에서 import한다.

```python
from scrapy.linkextractors import LinkExtractor
```

Github에서 저 위치로 들어가 보면 `__init__.py` , `lxmlhtml.py` 가 존재한다. 

따라서 `__init__.py` 파일을 보면 다음과 같이 `LxmlLinkExtractor`를 `LinkExtractor` 로 임포트하여 갖고 있는걸 확인할 수 있다. 즉, `LxmlLinkExtractor` 가 대외적으로는 `LinkExtractor` 역할을 하고 있는 것이다.

```python
# Top-level imports
from scrapy.linkextractors.lxmlhtml import LxmlLinkExtractor as LinkExtractor
```

## LxmlLinkExtractor

`LxmlLinkExtractor` 클래스의 함수로는 `__init__()` , `extract_links()` 가 있다. 우리가 주목해야할 것은 `extract_links()` 함수인데 이는 Scrapy 공식 문서에 다음과 같이 설명되어 있다.

> Returns a list of Link objects from the specified response.
Only links that match the settings passed to the **init** method of the link extractor are returned.
Duplicate links are omitted.

마지막 문장에서 알 수 있듯이 *'중복 된 링크는 생략되었습니다.'*  라고 한다. 하지만 이는 `LxmlLinkExtractor` 클래스의 객체를 생성할 때 `unique`옵션으로 할당할 수 있다. 

> unique (boolean) – whether duplicate filtering should be applied to extracted links.

그런데 `extract_links()` 함수에는 `unique` 옵션에 대한 처리 없이 무조건 unique한 리스트만 반환하도록 코드가 짜여 있다.

```python
class LxmlLinkExtractor(FilteringLinkExtractor):
    ...
    def extract_links(self, response):
        """Returns a list of :class:`~scrapy.link.Link` objects from the
        specified :class:`response <scrapy.http.Response>`.
        Only links that match the settings passed to the ``__init__`` method of
        the link extractor are returned.
        Duplicate links are omitted.
        """
        base_url = get_base_url(response)
        if self.restrict_xpaths:
            docs = [
                subdoc
                for x in self.restrict_xpaths
                for subdoc in response.xpath(x)
            ]
        else:
            docs = [response.selector]
        all_links = []
        for doc in docs:
            links = self._extract_links(doc, response.url, response.encoding, base_url)
            all_links.extend(self._process_links(links))
        return unique_list(all_links)
```

# 해결책

따라서 나는 `LxmlLinkExtractor` 를 재정의 해서 사용하기로 했다. 먼저 `[LinkExtractor.py](http://linkextractor.py)` 파일을 생성하고 그 안에 다음과 같이 적어 주었다. `extract_links()` 함수를 호출할 때 `omit` 옵션을 줄 수 있도록 수정했다. `omit` 옵션의 기본값은 `False` 이고, `True` 로 변경할 경우 중복된 링크들까지 모두 출력하도록 했다.

```python
from scrapy.linkextractors.lxmlhtml import LxmlLinkExtractor
from scrapy.utils.python import unique as unique_list
from scrapy.utils.response import get_base_url

class MyLinkExtractor(LxmlLinkExtractor):
    def __init__(
        self,
        allow=(),
        deny=(),
        allow_domains=(),
        deny_domains=(),
        restrict_xpaths=(),
        tags=('a', 'area'),
        attrs=('href',),
        canonicalize=False,
        unique=True,
        process_value=None,
        deny_extensions=None,
        restrict_css=(),
        strip=True,
        restrict_text=None,
    ):
        super().__init__(
            allow,
            deny,
            allow_domains,
            deny_domains,
            restrict_xpaths,
            tags,
            attrs,
            canonicalize,
            unique,
            process_value,
            deny_extensions,
            restrict_css,
            strip,
            restrict_text,
        )

    # Override extract_links()
    def extract_links(self, response, omit=True):
        """Returns a list of :class:`~scrapy.link.Link` objects from the
        specified :class:`response <scrapy.http.Response>`.
        Only links that match the settings passed to the ``__init__`` method of
        the link extractor are returned.
        Duplicate links are omitted or not.
        """
        base_url = get_base_url(response)
        if self.restrict_xpaths:
            docs = [
                subdoc
                for x in self.restrict_xpaths
                for subdoc in response.xpath(x)
            ]
        else:
            docs = [response.selector]
        all_links = []
        for doc in docs:
            links = self._extract_links(doc, response.url, response.encoding, base_url)
            all_links.extend(self._process_links(links))
        if omit:
            return unique_list(all_links)
        else:
            return all_links
```

그리고 기존에 `LinkExtractor` 를 사용하던 코드가 다음과 같았다면,

```python
from scrapy.linkextractors import LinkExtractor

url_forms = LinkExtractor(restrict_xpaths=self.url_xpath,attrs='href',unique=False)
links: List[str] = url_forms.extract_links(response)
```

이를 다음과 같이 바꿔준다.

```python
from .LinkExtractor import MyLinkExtractor

url_forms = MyLinkExtractor(restrict_xpaths=self.url_xpath, attrs='href')
links: List[str] = url_forms.extract_links(response, omit=False)
```

# 참고

이 버그는 Scrapy 개발자가 `bug` 태그를 붙여 놓기도 했다.

[https://github.com/scrapy/scrapy/issues/3798](https://github.com/scrapy/scrapy/issues/3798)

# 끝