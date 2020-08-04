---
layout: post
title: "백준 2156번 - 포도주 시식"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon C DP
---

3잔 연속으로 마실 수 없는 것만 기억하면 쉽게 동적 프로그래밍으로 풀 수 있다.

경우의 수는 다음과 같다.

1. i번째를 안 마신 경우

2. i-1번째를 안 마신 경우

3. i-2번째를 안 마신 경우

|      | i-3  | i-2  | i-3  |   i  |
| ---- | ---- | ---- | ---- | ---- |
|1번 경우|  O   |  O   |  O   |  X   |
| ---- | ---- | ---- | ---- | ---- |
|2번 경우|  O   |  O   |  X   |  O   |
| ---- | ---- | ---- | ---- | ---- |
|3번 경우|  O   |  X   |  O   |  O   |

시간복잡도 : O(n)

전체 코드는 다음과 같다.

```c
#include <stdio.h>

int n;
int a[10001];
int sum[10001];

int max(int a, int b) {return a<b?b:a;}

int dp() {
	sum[0] = a[0] + 0;
	sum[1] = sum[0] + a[1];
	for (int i = 2; i < n; i++) {
		sum[i] = max(max(sum[i-2]+a[i], sum[i-3]+a[i-1]+a[i]), sum[i-1]);
	}
	printf("%d", sum[n-1]);
}

int main() {
	scanf("%d", &n);
	for (int i = 0; i < n; i++) {
		scanf("%d", &a[i]);
	}
	dp();
	return 0;
}
```

블로그 포스팅에는 이해하기 쉽게 13번 줄의 비교 순서를 바꿨다. 깃허브 코드와는 약간 다르니 유의하자.

[https://github.com/qwlake/study-algorithm/blob/master/C%2CC%2B%2B/baekjoon/n2156/n2156.c](https://github.com/qwlake/study-algorithm/blob/master/C%2CC%2B%2B/baekjoon/n2156/n2156.c)