---
layout: post
title: "백준 1244번 - 스위치 켜고 끄기"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon KOI C
---

[https://www.acmicpc.net/problem/1244](https://www.acmicpc.net/problem/1244)

정보올림피아드 2000년도 초등부 문제로 정답률은 낮지만 쉬운 문제다.

문제에서 1. 남학생이 스위치를 조작할 때와 2. 여학생이 스위치를 조작할 때를 구분할 수 있다.

남학생 같은 경우는 몰아서 스위치를 탐색하는 시간을 줄일 수 있지만,

여학생 같은 경우는 조건부 스위치 조작이기 때문에 순서가 상관 있다.

따라서 입력된 학생 순서대로, 매 학생마다 스위치를 한 번씩 탐색하는 수 밖에 없다. (내가 풀기론 그렇다.)

남학생의 스위치 탐색에서는 스위치를 0번부터 N번까지 전부 다 도는게 아니라,

0번, 3번, 6번 ... N번 순으로 탐색하는 것이 좋겠다.

```c
        if (st[i][0] == 1) {    // man
            for (int j = 1; j <= n/st[i][1]; j++) {
                arr[j*st[i][1]] = !arr[j*st[i][1]];
            }
        }
```

여학생의 스위치 탐색은 그냥 (i-1),(i+1)번, (i-2)(i+2)번 순으로 탐색한다.

```c
        else {                // woman
            arr[st[i][1]] = !arr[st[i][1]];
            for (int j = 1; 0<st[i][1]-j && st[i][1]+j<=n; j++) {
                if (arr[st[i][1]-j] == arr[st[i][1]+j]) {
                    arr[st[i][1]-j] = !arr[st[i][1]-j];
                    arr[st[i][1]+j] = !arr[st[i][1]+j];
                } else break;
            } 
        }
```

마지막으로 출력은 20개 단위로 끊어서 출력해야 하기 때문에 약간의 주의를 기울일 필요가 있다.

```c
    for (int i = 0; i < (n-1)/21; i++) {
        for (int j = i*20+1; j <= i*20+20; j++) printf("%d ", arr[j]);
        printf("\n");
    }
    int tmp = n/21;
    for (int i = tmp*20+1; i <= (tmp*21)+(n%21); i++) {
        printf("%d ", arr[i]);
    }
```

전체 코드는 다음과 같다.

```c
#include <stdio.h>

int n, m, arr[101], st[101][2];
int main() {
	scanf("%d", &n);
    for (int i = 1; i <= n; i++) scanf("%d", &arr[i]);
    scanf("%d", &m);
    for (int i = 1; i <= m; i++) scanf("%d %d", &st[i][0], &st[i][1]);
    for (int i = 1; i <= m; i++) {
        if (st[i][0] == 1) {    // man
            for (int j = 1; j <= n/st[i][1]; j++) {
                arr[j*st[i][1]] = !arr[j*st[i][1]];
            }
        } else {                // woman
            arr[st[i][1]] = !arr[st[i][1]];
            for (int j = 1; 0<st[i][1]-j && st[i][1]+j<=n; j++) {
                if (arr[st[i][1]-j] == arr[st[i][1]+j]) {
                    arr[st[i][1]-j] = !arr[st[i][1]-j];
                    arr[st[i][1]+j] = !arr[st[i][1]+j];
                } else break;
            } 
        }
    }
    for (int i = 0; i < (n-1)/21; i++) {
        for (int j = i*20+1; j <= i*20+20; j++) printf("%d ", arr[j]);
        printf("\n");
    }
    int tmp = n/21;
    for (int i = tmp*20+1; i <= (tmp*21)+(n%21); i++) {
        printf("%d ", arr[i]);
    }
	return 0;
}
```

[https://github.com/qwlake/study-algorithm/blob/master/C%2CC%2B%2B/baekjoon/KOI2000/n1244/n1244.c](https://github.com/qwlake/study-algorithm/blob/master/C%2CC%2B%2B/baekjoon/KOI2000/n1244/n1244.c)