---
layout: post
title: "백준 2669번 - 직사각형 네개의 합집합의 면적 구하기"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon KOI Python
---

[https://www.acmicpc.net/problem/2669](https://www.acmicpc.net/problem/2669)

그냥 간단하게 2차배열을 만들고 전부 0으로 채운다.

입력으로 직사각형의 좌표가 들어오면, 좌표 안에 해당하는 2차배열의 칸들을 1로 채운다.

전부 채우고 1의 개수를 구하면 이것이 답이다.

파이썬 코드

```python
import sys
a = [[0]+list(map(int, sys.stdin.readline().split())) for i in range(4)]
total = 0
mat = [[0 for _ in range(101)] for _ in range(101)]
for r in a:
    for i in range(r[1], r[3]):
        for j in range(r[2], r[4]):
            if mat[i][j] == 0:
                total += 1
                mat[i][j] = 1
print(total)
```

[https://github.com/qwlake/study-algorithm/blob/master/Python/baekjoon/KOI1996/n2669/n2669.py](https://github.com/qwlake/study-algorithm/blob/master/Python/baekjoon/KOI1996/n2669/n2669.py)