---
layout: post
title: "백준 1931번 - 회의실배정"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon Python Greedy
---

[https://www.acmicpc.net/problem/1931](https://www.acmicpc.net/problem/1931)

정답률은 낮지만 쉬운 문제다.

회의가 빨리 끝나는 것부터 배정하면 된다.

회의가 빨리 끝나는 것이 무엇인지 판단하려면

1. 정렬해두고 뽑거나

2. 회의를 뽑을 때마다 전체를 탐색하는 방법

이 있겠다.

2번 방법은 시간복잡도가 N^2이기 때문에 패스하고 1번 방법을 쓰겠다.

데이터 입력은 (1, 4), (3, 5)와 같이 두 수가 세트이다.

이 데이터를 특정 인덱스 기준으로 정렬하려면 C에서는 직접 다 짜야겠지만,

파이썬에서는 코드 한 줄이면 된다.

```python
import sys
In = sys.stdin.readline
 
N = int(In())
c_list = [(*map(int, In().split()),) for _ in range(N)]
c_list = sorted(c_list, key=lambda c: (c[1], c[0]))
idx = 0
cnt = 0
for c in c_list:
    if idx <= c[0]:
        idx = c[1]
        cnt += 1
print(cnt)
```

[https://github.com/qwlake/study-algorithm/blob/master/Python/baekjoon/n1931/n1931.py](https://github.com/qwlake/study-algorithm/blob/master/Python/baekjoon/n1931/n1931.py)