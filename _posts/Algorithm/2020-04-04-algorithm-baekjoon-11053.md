---
layout: post
title: "백준 11053번 - 가장 긴 증가하는 부분 수열"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon C++ DP
---

[https://www.acmicpc.net/problem/11053](https://www.acmicpc.net/problem/11053)

DP(동적 프로그래밍, 다이나믹 프로그래밍) 문제의 기본이다. 응용되는 DP 문제가 많다. 꼭 풀어보길 권하는 문제이다.

1. 앞 부분부터 서서히 부분집합을 늘려간다.

2. 부분집합 안에서 i번째 수보다 i+1번째 수가 클 때,

3. (이전 크기의 부분집합에서 구했던 수열의 길이) 와 (현재의 부분집합에서 구하는 수열의 길이) 중 큰 값을 저장한다.

C언어로도 코드가 간단하니 코드를 보길 바란다.

C++ 코드

```cpp
#include <iostream>
#include <algorithm>

using namespace std;

int N, a[1001], d[1001], ans;

int main() {
	cin >> N;
	for (int i = 1; i < N+1; i++)
		cin >> a[i];

	for (int i = 1; i <= N; i++) {
		d[i] = 1;
		for (int j = 1; j <= i; j++) {
			if (a[i] > a[j]) {
				d[i] = max(d[i], d[j]+1);
			}
		}
	}

	for (int i = 1; i <= N; i++) {
		ans = d[i]<ans? ans:d[i];
	}

	cout << ans << endl;

	return 0;
}
```

[https://github.com/qwlake/study-algorithm/blob/dev/C%2CC%2B%2B/baekjoon/n11053/n11053.cpp](https://github.com/qwlake/study-algorithm/blob/dev/C%2CC%2B%2B/baekjoon/n11053/n11053.cpp)