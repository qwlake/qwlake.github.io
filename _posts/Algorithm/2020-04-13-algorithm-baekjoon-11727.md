---
layout: post
title: "백준 11727번 - 2xn 타일링 2"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon C++ DP
---

[https://www.acmicpc.net/problem/11727](https://www.acmicpc.net/problem/11727)

DP(동적프로그래밍) 문제이긴 한데, 간단한 규칙을 보인다.

타일이 한 개 있을 때부터 순서대로

1, 3, 5, 11, 21 ...

위와 같은 규칙을 보인다.

규칙을 수식으로 표현하면 다음과 같다.

```cpp
﻿a[i] = a[i-1]+a[i-1]*2
```

전체 코드는 다음과 같다.

C++ 코드

```cpp
#include <iostream>

using namespace std;

int N;
long long a[1001];

int max(int a, int b) {return a<b?b:a;}

int main() {
	cin >> N;
	// for (int i = 0; i < N; i++)
	// 	cin >> a[i];

	a[1] = 1;
	a[2] = 3;
	a[3] = 5;

	for (int i = 4; i <= N; i++) {
		a[i] = (a[i-1] + a[i-2]*2)%10007;
	}

	cout << a[N];

	return 0;
}
```

[https://github.com/qwlake/study-algorithm/blob/master/C%2CC%2B%2B/baekjoon/n11727/n11727.cpp](https://github.com/qwlake/study-algorithm/blob/master/C%2CC%2B%2B/baekjoon/n11727/n11727.cpp)