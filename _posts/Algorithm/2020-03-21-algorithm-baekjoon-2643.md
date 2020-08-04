---
layout: post
title: "백준 2643번 - 색종이 올려 놓기"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon KOI C Python DP
---

[https://www.acmicpc.net/problem/2643](https://www.acmicpc.net/problem/2643)

일단 먼저 파이썬으로 짠 코드를 보자. (정답코드이다)

아주 간단하다.

```python
import sys
n = int(input())
a = [sorted(list(map(int, sys.stdin.readline().split(" ")))) for _ in range(n)]
a = sorted(a, key=lambda x:(x[0], x[1]))
d = [0]*n
for i, e in enumerate(a):
    d[i] = 1
    for j in range(i):
        if e[1] >= a[j][1]:
            d[i] = max(d[j]+1, d[i])
print(max(d))
```

원리는 색종이들을 먼저 세로순으로 정렬한다. 뒤이어 가로순으로 정렬한다.

이렇게 했을시 색종이들을 세로순만 보면 모두 다 채택 가능하다.

이제 변수는 가로길이인데, 이는 가장 긴 증가하는 부분 수열 문제와 같다.

가로 길이가 줄어들지 않고 증가하면서 가장 길게 뽑을 수 있는 부분 수열을 구하면 된다.

C언어 코드를 보고 마치겠다.

```c
#include <stdio.h>

int n;
int a[100][2];
int d[100] = {0,};

int insertionSort(int idx) {
	int i;
	int tmp[2] = {a[idx][0], a[idx][1]};
	for (i = idx-1; i >= 0; i--) {
		if (a[i][0] > tmp[0]) {
			a[i+1][0] = a[i][0]; a[i+1][1] = a[i][1];
		} else if (a[i][0] == tmp[0] && a[i][1] > tmp[1]) {
			a[i+1][0] = a[i][0]; a[i+1][1] = a[i][1];
		} else {
			break;
		}
	}
	i++;
	a[i][0] = tmp[0]; a[i][1] = tmp[1];
	return 0;
}

int main() {
	scanf("%d", &n);
	int t1, t2;
	for (int i = 0; i < n; i++) {
		scanf("%d %d", &t1, &t2);
		if (t1 > t2) {
			a[i][0] = t2;
			a[i][1] = t1;
		} else {
			a[i][0] = t1;
			a[i][1] = t2;
		}
		insertionSort(i);
	}
	for (int i = 0; i < n; i++) {
		d[i] = 1;
		for (int j = 0; j < i; j++) {
			if (a[i][1] >= a[j][1]) {
				d[i] = d[i]>d[j]+1? d[i]:d[j]+1;
			}
		}
	}
	for (int i = 1; i < n; i++) {
		d[0] = d[i]>d[0]? d[i]:d[0];
	}
	printf("%d", d[0]);
	return 0;
}
```

[https://github.com/qwlake/study-algorithm/blob/master/C%2CC%2B%2B/baekjoon/KOI1999/n2643/n2643.c](https://github.com/qwlake/study-algorithm/blob/master/C%2CC%2B%2B/baekjoon/KOI1999/n2643/n2643.c)