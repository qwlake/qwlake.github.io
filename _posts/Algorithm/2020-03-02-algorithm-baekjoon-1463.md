---
layout: post
title: "백준 1463번 - 1로 만들기"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon Python DP
---

[https://www.acmicpc.net/problem/1463](https://www.acmicpc.net/problem/1463)

> 정수 X에 사용할 수 있는 연산은 다음과 같이 세 가지 이다.
X가 3으로 나누어 떨어지면, 3으로 나눈다.
X가 2로 나누어 떨어지면, 2로 나눈다.
1을 뺀다.
정수 N이 주어졌을 때, 위와 같은 연산 세 개를 적절히 사용해서 1을 만들려고 한다. 연산을 사용하는 횟수의 최솟값을 출력하시오.

처음에는 이 문제의 접근방식을 너비우선탐색(BFS)을 이용하여 푸려고 했다.

테스트 케이스에서는 잘 작동하였지만, 제출시에는 너무 많은 데이터로 인해 시간초과가 난다.

BFS로 작성했던 코드를 보자.

```python
def bfs(N):
    que = [1]
    cnt = 0
    cnt_dic = {1:0}
    visited = [False for _ in range(1000001)]
    while True:
        q = que.pop(0)
        cnt = cnt_dic[q] + 1
        temp = (q*3, q*2, q+1)
        if visited[N]:
            return cnt_dic[N]
        for i in range(3):
            if temp[i] <= N and not visited[temp[i]]:
                cnt_dic[temp[i]] = cnt 
                visited[temp[i]] = True
                que.append(temp[i])
# print(bfs(N))
```

전형적인 BFS 탐색이다.

입력 받은 정수 X에 대해 취할 수 있는 옵션은 3가지이다.

그런데 이 3가지 경우를 항상 탐색하니 시간이 오버될 수 밖에 없다.

그래서 다음과 같이 동적 프로그래밍(DP)를 사용해서 풀었다.

```python
def dp(N):
    cnt_list = [0 for i in range(N+1)]
    for i in range(2, N+1):
        cnt_list[i] = cnt_list[i-1] + 1
        if i % 2 == 0:
            cnt_list[i] = min(cnt_list[i], cnt_list[int(i//2)]+1)
        if i % 3 == 0:
            cnt_list[i] = min(cnt_list[i], cnt_list[int(i//3)]+1)
    return cnt_list[N]
print(dp(N))
```

0부터 입력 받은 수 N까지 각 숫자에 대한 카운트 수를 `cnt_list`에 저장한다.

N부터 2까지 거꾸로 내려오는 액션을 취하고 있고,

1을 뺐을 경우, 2로 나누어질 경우, 3으로 나누어질 경우 모두를 고려해 가장 적은 카운트 수를 cnt_list에 저장해 나간다.

코드 전체는 하단 링크에 있다.

[https://github.com/qwlake/study-algorithm/blob/master/Python/baekjoon/n1463/n1463.py](https://github.com/qwlake/study-algorithm/blob/master/Python/baekjoon/n1463/n1463.py)