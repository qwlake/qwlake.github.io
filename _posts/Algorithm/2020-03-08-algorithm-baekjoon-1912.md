---
layout: post
title: "백준 1912번 - 연속합"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon C DP
---

[https://www.acmicpc.net/problem/1912](https://www.acmicpc.net/problem/1912)

정답률 27%로 꽤 낮은 편에 속한다.

동적 프로그래밍으로 풀면 쉽다.

...,(i-2),(i-1)번째 까지의 sum에 i번째의 값을 더한다.

이 값이 양수이면 i번째 값을 채택하고, 아니라면 sum을 0으로 초기화 후 (i+1)부터 다시 쌓나 나간다.

그리고 sum이 가장 높았을때의 값이 연속합 최대값이다.

```c
#include <stdio.h>
 
int n;
int a[100001];
 
int max(int a, int b) {return a<b?b:a;}
 
int main() {
    scanf("%d", &n);
    for (int i = 0; i < n; i++) {
        scanf("%d", &a[i]);
    }
 
    int max = a[0] + 0;
    int sum = a[0] + 0;
    int temp = 0;
    for (int i = 1; i < n; i++) {
        temp = sum+a[i];
        if (temp > 0) {
            sum += a[i];
        } 
        if (max<sum) {
            max = sum;
        }
        if (temp <= 0)    {
            sum = 0;
        }
    }
    if (max == 0) {
        max = a[0] + 0;
        for (int i = 0; i < n; i++) {
            max = max<a[i]? a[i]:max;
        }
    }
    printf("%d",max);
 
    return 0;
}
```

29번줄에 있는 if문은 주어진 입력이 모두 음수일때 max값이 지정되지 않기 때문이다.

이때는 주어진 입력을 돌면서 그나마 가장 최대인 값을 찾는다.