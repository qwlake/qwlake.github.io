---
layout: post
title: "백준 4179번 - 불!"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon Python BFS Deque
---

# **4179 - 불!**

## **1. 개요**

[https://www.acmicpc.net/problem/4179](https://www.acmicpc.net/problem/4179)

## **2. 코드**

```python
import sys
from collections import deque

directions = ((1,0),(-1,0),(0,1),(0,-1))
R, C = map(int, sys.stdin.readline().split())
maze = [['O']*(C+2)]
for _ in range(R):
    maze.append(list(map(str, f'O{sys.stdin.readline().strip()}O')))
maze.append(['O']*(C+2))
fire_q_tmp = deque([])
jihun_q = deque([])
for r in range(1, len(maze)-1):
    for c in range(1, len(maze[0])-1):
        if maze[r][c] == 'F':
            fire_q_tmp.append((r, c, 0))
        elif maze[r][c] == 'J':
            jihun_q.append([(r, c, 0)])
fire_q = deque([fire_q_tmp])

break_flag = False
while not break_flag:
    if fire_q:
        fire_popped = fire_q.popleft()
        fire_nxt = []
        for r, c, d in fire_popped:
            for direction in directions:
                if maze[r+direction[0]][c+direction[1]] in ('.','J'):
                    maze[r+direction[0]][c+direction[1]] = 'F'
                    fire_nxt.append((r+direction[0], c+direction[1], d+1))
        if fire_nxt:
            fire_q.append(fire_nxt)

    if jihun_q:
        jihun_popped = jihun_q.popleft()
        jihun_nxt = []
        for r, c, d in jihun_popped:
            if r in (0, len(maze)-1) or c in (0, len(maze[0])-1):
                break_flag = True
                print(d)
                break
            for direction in directions:
                if maze[r+direction[0]][c+direction[1]] in ('.','O'):
                    maze[r+direction[0]][c+direction[1]] = 'J'
                    jihun_nxt.append((r+direction[0], c+direction[1], d+1))
        if jihun_nxt:
            jihun_q.append(jihun_nxt)
    else:
        print('IMPOSSIBLE')
        break
```

## **3. 설명**

1. 불이 다음에 번질 곳을 리스트로 묶어 `fire_q`에 저장
2. 지훈이 다음에 갈 곳을 리스트로 묶어 `jihun_q`에 저장
3. 큐에서 각각 하나씩 꺼내어 각각 한 칸씩 진행
4. 지훈이 가장자리에 도착하면 성공
5. 가장자리에 도달하기 전에 큐가 비게 되면 실패

## **4. 여정**

1. 메모리 초과 (어디가 문제?)
2. 배열 따로 두고 각각 BFS 진행 -> 그래도 틀림
3. 불, 지훈 동시 진행으로 갈아엎음 - > 성공

## **5. 결과**
![image](https://user-images.githubusercontent.com/41278416/88818590-d6db9c80-d1f9-11ea-866a-aaf69da735f7.png)