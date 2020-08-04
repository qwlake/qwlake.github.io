---
layout: post
title: "백준 4963번 - 섬의 개수"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon Java DFS Recursive
---

[https://www.acmicpc.net/problem/4963](https://www.acmicpc.net/problem/4963)

이 문제를 풀었을 때가 알고리즘 공부를 막 시작했을때라 코드가 많이 더럽다.

지금 보니 불필요한 부분도 많이 보인다.

그 때 헤매었던 부분 중 하나는 메모리 초과다.

섬을 탐색할 때 재귀를 사용했는데, 이미 탐색한 섬을 체크하는 부분이 재귀를 들어가기 전이 아니고 들어가고 나서 체크했었다.

따라서 이미 다른쪽에서 탐색한 섬이라고 체크했어야 하는데, 그러지 못해서 같은 섬을 중복해서 탐색하는 일이 발생했다.

이런 부분만 신경쓴다면 쉬운 문제다.

자바 소스코드

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.LinkedList;
import java.util.Queue;

public class Main {
	static int[][] direction = { { -1, 0 }, { -1, 1 }, { 0, 1 }, { 1, 1 }, { 1, 0 }, { 1, -1 }, { 0, -1 }, { -1, -1 } };
	static int row;
	static int col;
	static int map[][];

	public static void main(String[] args) throws IOException {
		BufferedReader input = new BufferedReader(new InputStreamReader(System.in));
		while (true) {
			String temp[] = input.readLine().split(" ");
			row = Integer.parseInt(temp[1]);
			col = Integer.parseInt(temp[0]);
			if (row == 0 && col == 0)
				break;
			map = new int[row][col];

			for (int i = 0; i < row; i++) {
				temp = input.readLine().split(" ");
				for (int j = 0; j < col; j++)
					map[i][j] = Integer.parseInt(temp[j]);
			}

			int count = startMerge();
			System.out.println(count);
		}
		input.close();
	}
    
    public static int startMerge() {
        Queue<int[]> que = new LinkedList<>();
		int count = 0;
		for (int i = 0; i < row; i++) {
			for (int j = 0; j < col; j++) {
				if (map[i][j] == 1) {
					merge(que, new int[] { i, j });
					count++;
				}
			}
		}
		return count;
	}

	public static void merge(Queue<int[]> que, int[] startPos) {
		que.add(startPos);
		map[startPos[0]][startPos[1]] = 0;
		int now[];
		int tempRow;
		int tempCol;
		while (!que.isEmpty()) {
			now = que.remove();
			for (int[] dir : direction) {
				tempRow = now[0] + dir[0];
				tempCol = now[1] + dir[1];
				if (0 <= tempRow && tempRow < map.length && 0 <= tempCol && tempCol < map[0].length
						&& map[tempRow][tempCol] == 1) {
					que.add(new int[] { tempRow, tempCol });
					map[tempRow][tempCol] = 0;
				}
			}
		}
		que.clear();
	}
}
```

[https://github.com/qwlake/study-algorithm/blob/master/Java/baekjoon/src/n4963/n4963.java](https://github.com/qwlake/study-algorithm/blob/master/Java/baekjoon/src/n4963/n4963.java)