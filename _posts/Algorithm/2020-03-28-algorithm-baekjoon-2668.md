---
layout: post
title: "백준 2668번 - 숫자고르기"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon KOI Python
---

[https://www.acmicpc.net/problem/2668](https://www.acmicpc.net/problem/2668)

이 문제에서는 유연하게 집합(set)을 활용할 수 있는 파이썬이 좋은 것 같다.

나는 숫자와 지금까지 합한 숫자들의 길이를 한꺼번에 집합에 넣어서 깊이우선탐색을 돌렸다.

```python
import sys
N = int(input())
a = [int(sys.stdin.readline()) for i in range(N)]
a.insert(0, 0)
sets = set()
for i in range(1, N+1):
    stack = [(i, [i])]
    while len(stack) != 0:
        s, s_list = stack.pop()
        if a[i] == s:
            sets.update(set(s_list))
            break
        for j in range(1, N+1):
            if a[j] == s:
                stack.append((j, s_list+[j]))
sets = sorted(list(sets))
print(len(sets))
for e in sets:
    print(e)
```

[https://github.com/qwlake/study-algorithm/blob/master/Python/baekjoon/KOI1996/n2668/n2668.py](https://github.com/qwlake/study-algorithm/blob/master/Python/baekjoon/KOI1996/n2668/n2668.py)