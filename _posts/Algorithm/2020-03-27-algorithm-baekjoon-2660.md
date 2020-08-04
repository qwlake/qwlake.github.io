---
layout: post
title: "백준 2660번 - 회장뽑기"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon KOI Python
---

[https://www.acmicpc.net/problem/2660](https://www.acmicpc.net/problem/2660)

너비우선탐색으로 모든 회원들의 관계를 파악한다.

관계의 친밀도를 저장하는 2차 배열을 두고 각각의 인덱스가 회원의 번호를 나타내게 하여 인덱스를 이용해 회원들의 친밀도를 한 번에 볼 수 있다.

```python
import sys

n = int(input())
a = [[0 for _ in range(n+1)] for _ in range(n+1)]

while True:
    t = list(map(int, sys.stdin.readline().split()))
    if t[0] == -1:
        break
    a[t[0]][t[1]] = 1
    a[t[1]][t[0]] = 1

def find(idx):
    s = {idx}
    q = [(idx, 0)]
    m = 0
    while len(q) != 0:
        p, d = q.pop(0)
        m = d
        for i, e in enumerate(a[p]):
            if e == 1 and i not in s:
                q.append((i, d+1))
                s.add(i)
    return m

score_dic = dict()
for i in range(1,n+1):
    score_dic[i] = find(i)
sort = sorted(score_dic.items(), key=lambda x: x[1])
count = n
can = ""
for i, e in enumerate(sort):
    if e[1] != sort[0][1]:
        count = i
        break
    else:
        can += str(e[0]) + " "
print(sort[0][1], count)
print(can.strip())
```

[https://github.com/qwlake/study-algorithm/blob/master/Python/baekjoon/KOI1997/n2660/n2660.py](https://github.com/qwlake/study-algorithm/blob/master/Python/baekjoon/KOI1997/n2660/n2660.py)