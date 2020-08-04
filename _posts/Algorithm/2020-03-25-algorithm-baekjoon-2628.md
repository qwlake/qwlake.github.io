---
layout: post
title: "백준 2628번 - 종이자르기"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon KOI C
---

[https://www.acmicpc.net/problem/2628](https://www.acmicpc.net/problem/2628)

아주 간단하다.

세로로 잘라야 하는 배열과 가로로 잘라야 하는 배열을 두고 입력을 받아 각각 채운다.

그 다음 이 두 배열(a,b 배열)을 정렬시킨 후에 넓이를 구해 비교하면 된다.

넓이를 구해 비교하는 부분을 자세히 설명해 보겠다.

넓이를 구하기 위해서는 잘려진 종이의 1.가로 길이와 2.세로 길이가 필요하다.

각각의 가로 세로 길이를 구하기 위해서 아까 만든 배열을 사용하는데, 이는 정렬되어 있으므로

a[i+1]-a[i]가 가로 길이가 되고, 마찬가지로 b[j+1]-b[j]가 세로 길이가 되어 이중 for 문을 구성해 돌린다.

이렇게 구한 가로 세로 길이를 곱해서 가장 큰 부분을 뽑는다.

```c
#include <stdio.h>

int row, col, n, a[100], b[100], tmp;
int as = 0;
int bs = 0;

void swap(int *x, int *y) {
    tmp = *x;
    *x = *y;
    *y = tmp;
}

void sort(int arr[], int size) {
    for (int i = 1; i < size; i++) {
        for (int j = i; j > 0; j--) {
            if (arr[j-1] > arr[j]) swap(&arr[j-1], &arr[j]);
            else break;
        }
    }
}

int main() {
	scanf("%d %d\n%d", &col, &row, &n);
    a[as++] = 0; b[bs++] = 0;
    for (int i = 0; i < n; i++) {
        scanf("%d", &tmp);
        if (tmp == 0) scanf("%d", &a[as++]); 
        else scanf("%d", &b[bs++]); 
    }
    a[as++] = row; b[bs++] = col;
    sort(a, as); sort(b, bs);
    int max = 0;
    for (int i = 0; i < as-1; i++) {
        for (int j = 0; j < bs-1; j++) {
            tmp = (a[i+1]-a[i])*(b[j+1]-b[j]);
            max = max<tmp?tmp:max;
        }
    }
    printf("%d", max);
	return 0;
}
```

[https://github.com/qwlake/study-algorithm/blob/master/C%2CC%2B%2B/baekjoon/KOI2001/n2628/n2628.c](https://github.com/qwlake/study-algorithm/blob/master/C%2CC%2B%2B/baekjoon/KOI2001/n2628/n2628.c)