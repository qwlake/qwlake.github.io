---
layout: post
title: "백준 9663번 - N-Queen"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon Java BackTracking
---

[https://www.acmicpc.net/problem/9663](https://www.acmicpc.net/problem/9663)

일단 퀸은 대각선, 상하좌우 모두 움직일 수 있으므로, 퀸을 배치할때 배치할 퀸 주변 8방향에는 다른 퀸이 없어야 한다.

이 전제를 베이스로 백트래킹을 하다 보면 퀸을 배치할 수 있는 모든 경우의 수가 나온다.

```java
    public static void loop(int[] q, int pos) {
        boolean flag = true;
        if (pos == N) {
            count++;
            return;
        }
        for (int i = 0; i < N; i++) {
            q[pos] = i;
            flag = true;
            for (int j = 0; j < pos; j++)
                if (q[j] == q[pos] || Math.abs(q[j] - q[pos]) == pos - j) {
                    flag = false;
                    break;
                }
            if (flag)
                loop(q, pos+1);
        }
    }
```

위의 백트래킹 함수를 보자.

7번줄에 있는 for문은 새로 배치할 퀸의 위치를 찾는다.

10번줄에 있는 for문은 지금까지 배치된 퀸들에 대해서 반복한다.

11번줄의 if문에서 새로 배치할 퀸의 위치가 기존 퀸들의 공격 범위 안에 있는지 검사하고,

만약 기존 퀸들의 공격 범위에 있지 않다면 다음 퀸을 배치하러 함수를 호출한다.

전체 코드는 하단 링크에 있다.

[https://github.com/qwlake/study-algorithm/blob/master/Java/baekjoon/src/n9663/n9663_v2.java](https://github.com/qwlake/study-algorithm/blob/master/Java/baekjoon/src/n9663/n9663_v2.java)