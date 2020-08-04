---
layout: post
title: "백준 9095번 - 1,2,3 더하기"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon Python DP
---

[https://www.acmicpc.net/problem/9095](https://www.acmicpc.net/problem/9095)

전형적인 DP 문제다.

0부터 높여가면서 이전에 거쳐온 숫자들 중에서 가장 많은 경우의 수를 고르고, 해당 숫자에 1을 증가시켜서 DP 배열에 저장한다.

얼핏 보면 피보나치와도 비슷하다.

```python
T = int(input())
n_list = [_ for _ in range(T)]
for i in range(T):
    n_list[i] = int(input())
n_result = [1, 2, 4]
 
for i in range(3, max(n_list)):
    n_result.append(n_result[i-3] + n_result[i-2] + n_result[i-1])
 
for i in n_list:
    print(n_result[i-1])
```

[https://github.com/qwlake/study-algorithm/blob/master/Python/baekjoon/n9095/n9095.py](https://github.com/qwlake/study-algorithm/blob/master/Python/baekjoon/n9095/n9095.py)