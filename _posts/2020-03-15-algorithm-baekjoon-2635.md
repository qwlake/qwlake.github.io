---
layout: post
title: "백준 2635번 - 수 이어가기"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon C Recursive
---

[https://www.acmicpc.net/problem/2635](https://www.acmicpc.net/problem/2635)

입력 수를 재귀로 탐색하며 풀었다.

여기서 포인트는 **두 번째 수**가 **첫 번째 수의 3/4~2/4만큼의 값**을 가진다는 것이다.

예를 들어 입력값이 100이면 두 번째로 와야 하는 수는 100의 **3/4~2/4인** 75~50 사이에 있다.

물론 이 다음에 올 숫자도 그렇다.

다음에 오는 숫자들이 이 범위에 있어야 연속적으로 길게 숫자를 이어갈 수 있다.

시간복잡도 : O(1/4n)*0.625=O(n)

전체 코드는 다음과 같다.

```c
#include <stdio.h>

int n, m=0;
int ans[100], arr[100];

void loop(int p, int q, int k) {
    if (p < 0) {
        if (m < k) {
            m = k;
            for (int i = 0; i < k; i++) {
                ans[i] = arr[i];
            }
        }
        return;
    }
    arr[k] = q;
    loop(q, p-q, k+1);
}

int main() {
	scanf("%d", &n);
    if (n == 1) {
        printf("4\n1 1 0 1");
        return 0;
    }
    int i, stNum, edNum;
    stNum = (n/2)+(n%2);
    edNum = n-(stNum/2)+(stNum%2);
    for (i = stNum; i < edNum; i++) {
        loop(n, i, 0);
    }
    printf("%d\n%d ", m, n);
    for (i = 0; i < m-1; i++) {
        printf("%d ", ans[i]);
    }
	return 0;
}
```

[https://github.com/qwlake/study-algorithm/blob/master/C%2CC%2B%2B/baekjoon/KOI2000/n2635/n2635.c](https://github.com/qwlake/study-algorithm/blob/master/C%2CC%2B%2B/baekjoon/KOI2000/n2635/n2635.c)