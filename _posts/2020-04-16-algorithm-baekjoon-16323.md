---
layout: post
title: "백준 16323번 - Intergalactic Bidding"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon Python
---

[https://www.acmicpc.net/problem/16323](https://www.acmicpc.net/problem/16323)

영어 번역만 잘 하면 쉬운 문제다.

주어진 입찰 금액을 입찰자들이 제시한 금액으로 채운다.

입찰자들이 제시한 금액을 선택했을때 합이 주어진 입찰 금액과 같아야 한다.

1. 먼저 입찰자들을 제시 금액 내림차순으로 정렬한 후에

2. 남은 입찰 금액보다 i번째 입찰자의 제시 금액이 작다면 해당 입찰자를 저장한다.

3. 저장된 입찰자의 수와 입찰자 목록을 출력한다.

다음은 파이썬 전체 코드이다.

```python
import sys

n, s = map(int, sys.stdin.readline().split())
arr = list()
for _ in range(n):
    name, b = map(str, sys.stdin.readline().split())
    arr.append((name, int(b)))
arr.sort(key=lambda x: -x[1])
ret = list()
for name, b in arr:
    if b <= s:
        s -= b
        ret.append(name)
    if s <= 0:
        break
if s == 0:
    print(len(ret))
    for r in ret:
        print(r)
else:
    print(0)
```

[https://github.com/qwlake/study-algorithm/blob/master/Python/baekjoon/n16323/n16323.py](https://github.com/qwlake/study-algorithm/blob/master/Python/baekjoon/n16323/n16323.py)