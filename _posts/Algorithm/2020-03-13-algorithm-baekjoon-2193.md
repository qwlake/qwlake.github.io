---
layout: post
title: "백준 2193번 - 이친구"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon C DP
---

[https://www.acmicpc.net/problem/2193](https://www.acmicpc.net/problem/2193)

다이나믹 프로그래밍 입문 문제다.

쉬운지는 모르겠지만 간단한 문제다.

1이 연속으로 존재하면 안 되니, 새로운 자리수를 만드려면 이전 자리수가 반드시 0으로 끝나야 한다.

따라서 i-1번째에 0으로 끝난 경우와 i-2번째에 0으로 끝난 경우를 합하면 i번째의 값이 나온다.

즉, 피보나치 수열이다.

시간복잡도:O(n)

전체 코드는 다음과 같다.

```c
#include <stdio.h>

int main() {
    int n;
	scanf("%d", &n);
	
	long long a[91];
	a[1] = 1; a[2] = 1;
	for (int i = 3; i <= n; i++) {
		a[i] = a[i-1] + a[i-2];
	}
	printf("%lld", a[n]);
}
```

[https://github.com/qwlake/study-algorithm/blob/master/C%2CC%2B%2B/baekjoon/n2193/n2193.c](https://github.com/qwlake/study-algorithm/blob/master/C%2CC%2B%2B/baekjoon/n2193/n2193.c)