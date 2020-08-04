---
layout: post
title: "백준 9465번 - 스티커"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon Java DP
---

[https://www.acmicpc.net/problem/9465](https://www.acmicpc.net/problem/9465)

전형적인 DP(다이나믹 프로그래밍, 동적계획법)문제다.

```java
matrix[0][j] = Math.max(matrix[1][j-1], matrix[1][j-2]) + sticker[0][j-1];
matrix[1][j] = Math.max(matrix[0][j-1], matrix[0][j-2]) + sticker[1][j-1];
```

한 칸 전의 대각선에 있는 값과 두 칸 전의 대각선에 있는 값 중 큰 값과 스티커에서 그 자리의 값을 더해 matrix를 구성하는 방식으로 풀면 된다.

두 칸 전의 대각선을 선택하는 경우는 현재 스티커의 한 칸 전 위아래 스티커 모두 값이 낮아 두 칸 전 대각선의 값을 선택하는게 나은 경우이다.

한 칸 전의 대각선을 선택하는 경우는 그 반대이다. 위 코드를 보면 이해가 더 쉬울 것이다.

자바 코드

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;

public class Main {
	static int N;
	static int[][] sticker;
	static int[][] matrix;
	public static void main(String[] args) throws NumberFormatException, IOException {
		BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
		int times = Integer.parseInt(br.readLine());
		int answer[] = new int[times];
		for (int i = 0; i < times; i++) {
			in(br);
			matrix = new int[2][N+1];
			matrix[0][1] = sticker[0][0];
			matrix[1][1] = sticker[1][0];
			for (int j = 2; j < N+1; j++) {
				matrix[0][j] = Math.max(matrix[1][j-1], matrix[1][j-2]) + sticker[0][j-1];
				matrix[1][j] = Math.max(matrix[0][j-1], matrix[0][j-2]) + sticker[1][j-1];
			}
			answer[i] = Math.max(matrix[0][N], matrix[1][N]);
		}
		for (int temp : answer)
			System.out.println(temp);
		br.close();
	}
	
	public static void in(BufferedReader br) throws NumberFormatException, IOException {
		N = Integer.parseInt(br.readLine());
		sticker = new int[2][N];
		for (int i = 0; i < 2; i++) {
			StringTokenizer st = new StringTokenizer(br.readLine());
			for (int j = 0; j < N; j++)
				sticker[i][j] = Integer.parseInt(st.nextToken());
		}
	}
}
```

[https://github.com/qwlake/study-algorithm/blob/master/Java/baekjoon/src/n9465/n9465.java](https://github.com/qwlake/study-algorithm/blob/master/Java/baekjoon/src/n9465/n9465.java)