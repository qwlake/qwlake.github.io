---
layout: post
title: "백준 10844번 - 쉬운 계단 수"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon C++ DP
---

[https://www.acmicpc.net/problem/10844](https://www.acmicpc.net/problem/10844)

DP(다이나믹 프로그래밍, 동적 프로그래밍) 문제이다.

계단의 길이가 1자리일때에는 다음과 같이 0부터 9까지의 숫자 중 하나만 올 수 있다.

단, 계단 길이가 1자리에서 끝나면 0이 오는 경우는 빼므로 총 9개의 경우가 생긴다.

![image](https://user-images.githubusercontent.com/41278416/89280001-01f33f80-d683-11ea-8ed9-b107560d2083.png)

계단 길이가 2일때는 다음과 같다.

진행 원리는 어떤 숫자(i번째)가 있을때 (i-1)번째의 경우의 수를 더한다.

더하게 되는 수는 (i-1)번째에서 하나 작은 수의 경우와 하나 큰 수의 경우를 더하게 된다.

그리고 0과 9일때에는 (i-1)번째에서 1의 경우와 8의 경우만 가져올 수 있다.

![image](https://user-images.githubusercontent.com/41278416/89279991-fe5fb880-d682-11ea-8434-809967410995.png)

하나 작은 수의 경우와 하나 큰 수의 경우를 더한다

![image](https://user-images.githubusercontent.com/41278416/89280005-03bd0300-d683-11ea-8f16-ef32eaebdba9.png)

마찬가지로 계단 길이가 3일때는 다음과 같다.

![image](https://user-images.githubusercontent.com/41278416/89280009-04559980-d683-11ea-9fc0-5b367f14a30e.png)

코드를 보면 이해가 더 쉬울 것이다.

C++ 코드

```cpp
#include <iostream>

using namespace std;

int N;
long long a[10][100];

int max(int a, int b) {return a<b?b:a;}

int main() {
	cin >> N;

	for (int i = 0; i < 10; i++)
		a[i][0] = 1;

	long ret = 0;
	for (int i = 1; i < N; i++) {
		for (int j = 1; j < 9; j++) {
			a[j][i] = (a[j-1][i-1] + a[j+1][i-1]) % 1000000000;
		}
		a[0][i] = a[1][i-1] % 1000000000;
		a[9][i] = a[8][i-1] % 1000000000;
	}

	for (int i = 1; i < 10; i++)
		ret += a[i][N-1];
	cout << ret % 1000000000 << endl;

	return 0;
}
```

[https://github.com/qwlake/study-algorithm/blob/dev/C%2CC%2B%2B/baekjoon/n10844/n10844.cpp](https://github.com/qwlake/study-algorithm/blob/dev/C%2CC%2B%2B/baekjoon/n10844/n10844.cpp)