---
layout: post
title: "백준 17351번 - 3루수는 몰라"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon Python DP
---

[https://www.acmicpc.net/problem/17351](https://www.acmicpc.net/problem/17351)

문자열 'MOLA'를 세는 임시 문자열과 개수가 저장될 DP 배열을 만든다.

문제를 해결하기 위해서는 제공된 입력 문자 N*N을 2중 for문으로 탐색한다.

제공된 입력 문자 한 칸을 탐색할 때마다 DP 배열에서 똑같은 위치에 해당하는 곳에 처음 언급했던 '임시 문자열'과 개수를 저장한다.

예를 들어 다음과 같이 입력 문자가 주어지면

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c8a0955a-4532-4cc0-be69-59988c1725d5/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c8a0955a-4532-4cc0-be69-59988c1725d5/Untitled.png)

다음과 같이 DP 배열을 준비한다.

```python
[[['',0], ['',0], ['',0], ['',0]],
 [['',0], ['',0], ['',0], ['',0]],
 [['',0], ['',0], ['',0], ['',0]],
 [['',0], ['',0], ['',0], ['',0]]]
```

한 칸씩 진행할 때마다 문자열 칸에는 MOLA를 저장하고, 숫자 칸에는 지금까지 나온 MOLA의 최대 개수를 저장한다.

모든 과정을 마친 DP 배열은 다음과 같이 나온다.

```python
[[['M',0],      ['',0], ['',0], ['',0]],
 [['MO',0],     ['',0], ['',0], ['',0]],
 [['MOL',0],    ['',0], ['',0], ['',0]],
 [['',1],       ['',1], ['',1], ['',1]]]
```

'MOL' 아래쪽에 있는 칸에는 MOLA가 다 채워졌기 때문에 문자열을 ''으로 초기화하고 MOLA 카운트를 1 증가 시킨다.

다음은 파이썬 전체 코드이다.

```python
import sys
N = int(input())
a = ["X"+sys.stdin.readline().split()[0] for _ in range(N)]
a.insert(0, "X"*N)
mola = "MOLA"
dp = [[["", 0] for _ in range(N+1)] for _ in range(N+1)]
for i in range(1,N+1):
    for j in range(1,N+1):
        t1, t2 = dp[i][j-1], dp[i-1][j]
        s1 = t1[0] + a[i][j]
        s2 = t2[0] + a[i][j]
        t1_cnt, t2_cnt = t1[1], t2[1]
        idx1 = "MOLA".find(s1)
        idx2 = "MOLA".find(s2)
        if s1 == "MOLA":
            t1_cnt = t1_cnt + 1
            s1 = ""
        if s2 == "MOLA":
            t2_cnt = t2_cnt + 1
            s2 = ""
        if idx1 is not 0:
            s1 = ""
            if a[i][j] == 'M':
                s1 = "M"
        if idx2 is not 0:
            s2 = ""
            if a[i][j] == 'M':
                s2 = "M"
        if t1_cnt < t2_cnt:
            dp[i][j] = [s2, t2_cnt]
        elif t1_cnt > t2_cnt:
            dp[i][j] = [s1, t1_cnt]
        else:
            dp[i][j] = [s2, t2_cnt] if len(s1) < len(s2) else [s1, t1_cnt]
print(dp[N][N][1])
```

[https://github.com/qwlake/study-algorithm/blob/master/Python/baekjoon/n17351/n17351.py](https://github.com/qwlake/study-algorithm/blob/master/Python/baekjoon/n17351/n17351.py)