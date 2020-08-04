---
layout: post
title: "백준 2636번 - 치즈"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon KOI C
---

[https://www.acmicpc.net/problem/2636](https://www.acmicpc.net/problem/2636)

처음에는 치즈 안쪽에서 탐색해서 바깥쪽의 치즈만 녹이는 방식으로 접근하려 했다.

그런데 그렇게 하면 치즈 안의 기포가 있는 부분은 녹지 않아야 하는데 내 방식대로 하면 녹게 된다.

따라서 치즈가 없는 부분을 탐색하다가 공기와 맞닿아 있는 부분의 치즈만 녹인다.

이렇게 하면 치즈 안의 기포는 탐색하지 않으므로 문제에서 원하는 답을 구할 수 있다.

```c
#include <stdio.h>

int row, col, tmp, a[100][100];

int loop(int x, int y) {
    if (0<=y-1)
        if (a[x][y-1]==0) {a[x][y-1]--; loop(x, y-1);}
        else if (a[x][y-1]==1) a[x][y-1]=2;
    if (x+1<row)
        if (a[x+1][y]==0) {a[x+1][y]--; loop(x+1, y);}
        else if (a[x+1][y]==1) a[x+1][y]=2;
    if (y+1<col)
        if (a[x][y+1]==0) {a[x][y+1]--; loop(x, y+1);}
        else if (a[x][y+1]==1) a[x][y+1]=2;
    if (0<=x-1)
        if (a[x-1][y]==0) {a[x-1][y]--; loop(x-1, y);}
        else if (a[x-1][y]==1) a[x-1][y]=2;
}
int check() {
    tmp = 0;
    for (int i = 0; i < row; i++) {
        for (int j = 0; j < col; j++) {
            if (a[i][j] == 2) {
                a[i][j] = 0;
                tmp++;
            } else if (a[i][j] == -1) {
                a[i][j] = 0;
            }
        }
    }
    return tmp;
}

int main() {
	scanf("%d %d", &row, &col);
    for (int i = 0; i < row; i++) {
        for (int j = 0; j < col; j++) {
            scanf("%d", &a[i][j]);
        }
    }
    int n = 0;
    int cheese = 0;
    int ret = 0;
    while (1) {
        loop(0, 0);
        cheese = check();
        n++;
        if (cheese != 0) ret = cheese;
        else break;
    }
    printf("%d\n%d", n-1, ret);
	return 0;
}
```

[https://github.com/qwlake/study-algorithm/blob/master/C%2CC%2B%2B/baekjoon/KOI2000/n2636/n2636.c](https://github.com/qwlake/study-algorithm/blob/master/C%2CC%2B%2B/baekjoon/KOI2000/n2636/n2636.c)