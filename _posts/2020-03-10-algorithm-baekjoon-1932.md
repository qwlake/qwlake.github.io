---
layout: post
title: "백준 1932번 - 정수 삼각형"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon C DP
---

[https://www.acmicpc.net/problem/1932](https://www.acmicpc.net/problem/1932)

동적 프로그래밍 문제다.

입력 데이터를 전부 2차 배열에 넣고 탐색한다.

왼쪽 위에서 내려올때와 오른쪽 위에서 내려올때를 구분해서 DP 배열을 완성시키면 된다.

(i,j) 위치에서의 최대값을 구한다면,

(i-1,j)과 (i-1,j-1) 중 더 큰 값에 (i,j)의 값을 더해서 DP(i,j)에 저장한다.

![image](https://user-images.githubusercontent.com/41278416/89265950-3c9fac80-d670-11ea-97b1-84cb52bd238f.png)

진행 순서는 7>3>8>8>1>0>... 순으로 보면 된다. 잘 이해할거라 믿는다.

이런식으로 진행하면 마지막 행에서 최대값이 삼각형 전체의 경로 최대값이 된다.

코드는 다음과 같다.

```c
#include <stdio.h>
 
int max(int a, int b) {return a<b?b:a;}
 
int main() {
    int n;
    scanf("%d", &n);
    
    int a[500][500];
    for (int i = 0; i < n; i++)
        for (int j = 0; j <= i; j++)
            scanf("%d", &a[i][j]);
 
    for (int i = 1; i < n; i++) {
        a[i][0] = a[i][0] + a[i-1][0];
        for (int j = 1; j <= i+1; j++) {
            a[i][j] = max(a[i][j]+a[i-1][j], a[i][j]+a[i-1][j-1]);
        }
    }
 
    int max = 0;
    for (int i = 0; i < n; i++) {
        max = max < a[n-1][i]? a[n-1][i]:max;
    }
    printf("%d", max);
}
```

[https://github.com/qwlake/study-algorithm/blob/master/C%2CC%2B%2B/baekjoon/n1932/n1932.c](https://github.com/qwlake/study-algorithm/blob/master/C%2CC%2B%2B/baekjoon/n1932/n1932.c)