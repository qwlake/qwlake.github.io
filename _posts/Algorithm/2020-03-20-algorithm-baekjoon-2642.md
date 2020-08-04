---
layout: post
title: "백준 2642번 - 전개도"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon KOI C DP
---

처음에는 어떻게 접근해야할 지 몰랐지만 알고보니 간단했다.

우선 목표에 도달하기 위해서는 두 가지를 주목해야 한다.

1. 모든 면은 하나의 면만 마주보아야 한다.

2. 1번이 만족하면 숫자 1이 마주보는 면을 찾아야 한다.

따라서 각각의 면이 어떠한 면을 마주보는지 알면 2번은 자연스럽게 해결된다.

각 면이 어떤 면을 마주보는지 어떻게 알까?

그건 바로 전개도에서 같은 방향으로 두 번 접혔을때이다.

사진을 보자.

![image](https://user-images.githubusercontent.com/41278416/89278016-85f7f800-d680-11ea-84d6-c1d86dadae14.png)

전개도 (1)에서 4번 면이 왼쪽으로 두 번 접히면 2번 면을 만난다. 이 때 2, 4번 면은 서로 마주보는 관계가 된다.

4번 면에서 한 방향으로 두 번 접을 수 있는 면이 더 없으므로, 4번 면에서 마주볼 수 있는 면은 2번 밖에 없다. 2번에서 또한 마찬가지이다.

그렇다면 전개도 (2)는 어떨까?6번 면에서 한 방향으로 접을 수 있는 면을 따라가보자.

6 - 3 - 4

6 - 3 - 2 - 1 - 5

4, 5번 면이 6번 면에서 위쪽으로 두 번 접혔을때 나오는 면이다. 즉, 6번 면은 두 개의 면과 마주하고 있으므로 이 전개도는 접을 수 없는 전개도이므로 0을 출력해야 한다.

반대로 2번 면은 한 방향으로 두 번 접을 수 있는 면이 없다. 즉, 2번 면은 마주보는 면이 없다.

이렇게 각각의 면이 어떤 면을 바라보는지 배열에 저장하고, 이 배열 중에서 어떤 면이 마주보게 되는 면이 없다면 이는 "접을 수 없는전개도"로 분류하고, 모든 배열 원소가 각각의 값을 가진다면(각각 마주보는 면이 있다면) 1번 면이 마주보는 면을 출력하면 된다.

다음은 전체 코드이다.

```c
#include <stdio.h>

int a[8][8] = {0,};
int d[4][2] = {
	{-1, 0}, 		// up
	{0, 1}, 		// right
	{1, 0}, 		// down
	{0, -1}			// left
};
int opp[7] = {0,};

void findOpposite(int path[7], int direction, int cnt, int x, int y, int n) {
	int target;
	for (int i = 0; i < 4; i++) {
		target = a[x+d[i][0]][y+d[i][1]];
		if (x+d[i][0] > 0 && y+d[i][1] > 0 && 
			x+d[i][0] < 8 && y+d[i][1] < 8 &&
			target != 0 && 
			path[target] == 0) {
			path[target] = 1;
			if (direction == -1) {
				findOpposite(path, i, 1, x+d[i][0], y+d[i][1], n);
			} else if (direction==i && cnt==1 &&
					   (opp[n]==0 || opp[n]==target) &&
					   (opp[target]==0 || opp[target]==n)) {
				opp[n] = target;
				opp[target] = n;
				findOpposite(path, direction, cnt+1, x+d[i][0], y+d[i][1], n);
			} else {
				findOpposite(path, direction, cnt, x+d[i][0], y+d[i][1], n);
			}
		}
	}
}

int main() {
	for (int i = 1; i < 7; i++) {
		for (int j = 1; j < 7; j++) {
			scanf("%d", &a[i][j]);
		}
	}
	for (int i = 1; i < 7; i++) {
		for (int j = 1; j < 7; j++) {
			if (a[i][j] != 0) {
				int path[7] = {0,};
				path[a[i][j]] = 1;
				findOpposite(path, -1, 0, i, j, a[i][j]);
			}
		}
	}
	int flag = 0;
	for (int i = 1; i < 7; i++) {
		if (opp[i] == 0) flag = 1;
	}
	if (flag) printf("%d",0);
	else printf("%d", opp[1]);
	return 0;
}
```

[https://github.com/qwlake/study-algorithm/blob/master/C%2CC%2B%2B/baekjoon/KOI1999/n2642/n2642.c](https://github.com/qwlake/study-algorithm/blob/master/C%2CC%2B%2B/baekjoon/KOI1999/n2642/n2642.c)