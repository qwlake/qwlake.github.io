---
layout: post
title: "백준 1149번 - RGB거리"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon C DP
---

[https://www.acmicpc.net/problem/1149](https://www.acmicpc.net/problem/1149)

크게 보면 [이전까지의 집들을 칠하는 최소 비용 + 현재 집을 칠하는 비용] 중에서 최소값을 선택해 나가면 된다.

이웃하는 집(현재 위치의 집에서 이전 집과 다음 집) 끼리는 같은 색을 사용할 수 없으니,

[이전까지의 집들을 칠하는 최소 비용 + 현재 집을 칠하는 비용]의 경우는 3가지가 나올 수 있다.

1. 이전 집에서 레드와 그린 선택, 현재 집에서 블루 선택.

2. 이전 집에서 그린와 블루 선택, 현재 집에서 레드 선택.

3. 이전 집에서 레드와 블루 선택, 현재 집에서 그린 선택.

이 3가지 경우 중에서 가장 비용이 적게 드는 값을 저장하고 다음 집으로 이동해서 똑같은 절차를 밟는다.

```c
    int temp1, temp2;
    for (int i = 1; i < n; i++) {
        temp1 = a[i][0] + a[i-1][1];
        temp2 = a[i][0] + a[i-1][2];
        a[i][0] = (temp1<temp2)? temp1:temp2;
        temp1 = a[i][1] + a[i-1][0];
        temp2 = a[i][1] + a[i-1][2];
        a[i][1] = (temp1<temp2)? temp1:temp2;
        temp1 = a[i][2] + a[i-1][0];
        temp2 = a[i][2] + a[i-1][1];
        a[i][2] = (temp1<temp2)? temp1:temp2;
    }
```

전체 코드는 하단 링크에 있다.

[https://github.com/qwlake/study-algorithm/blob/master/C%2CC%2B%2B/baekjoon/n1149/n1149.c](https://github.com/qwlake/study-algorithm/blob/master/C%2CC%2B%2B/baekjoon/n1149/n1149.c)