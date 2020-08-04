---
layout: post
title: "백준 9251번 - LCS"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon Python DP LCS
---

[https://www.acmicpc.net/problem/9251](https://www.acmicpc.net/problem/9251)

최장 공통 부분 수열(LCS)문제다.

DFS를 써서 해봤는데 시간이 너무 오래 걸린다.

DP를 쓰는 것이 가장 무난하다.

행에는 첫 번쨰 입력 문자들을 배치하고, 열에는 두 번째 입력 문자들을 배치하여,

행과 열이 같은 문자라면 이전 상태에서의 최대값에 +1을 해준 값이 현재 DP의 값이다.

만약 행과 열이 다른 문자라면 이전에 저장해놓은 DP값이 현재의 값이 된다.

나는 위에서 2차 배열을 DP에 쓴다고 했고, 이는 왼쪽에서 오른쪽, 위에서 아래로 진행하므로,

현재 DP배열 위치에서 왼쪽이나 위쪽 값 중 큰 값을 가져온다.

현재 위치에서 왼쪽이나 위의 값이 이전 상태의 최대값이 되겠다.

```python
s1, s2 = ' '+input(), ' '+input()
dp = [[0] * (len(s2)) for _ in range(len(s1))]
for i in range(1, len(dp)):
    for j in range(1, len(dp[0])):
        if s1[i] == s2[j]:
            dp[i][j] = dp[i-1][j-1]+1
        else:
            dp[i][j] = max(dp[i-1][j], dp[i][j-1])
print(dp[-1][-1])
```

[https://github.com/qwlake/study-algorithm/blob/master/Python/baekjoon/n9251/n9251.py](https://github.com/qwlake/study-algorithm/blob/master/Python/baekjoon/n9251/n9251.py)