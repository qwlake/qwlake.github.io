---
layout: post
title: "백준 2579번 - 계단 오르기"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon Java DP
---

다이나믹 프로그래밍 문제다. 나는 자바로 풀었다.
3번 연속 계단을 밟지 않고 최대값을 찾는 방향으로 진행하면 된다.

시간복잡도 : O(n)

전체 코드는 다음과 같다.

```java
import java.util.Scanner;

public class Main {
	public static void main(String[] args) {
		Scanner input = new Scanner(System.in);
		int N = Integer.parseInt(input.nextLine());
		int[] steps = new int[N];
		int[][] matrix = new int[N+1][3];
		for (int i = 0; i < N; i++)
			steps[i] = Integer.parseInt(input.nextLine());
		for (int i = 1; i < N+1; i++) {
			matrix[i][0] = (matrix[i-1][1] > matrix[i-1][2])? matrix[i-1][1]:matrix[i-1][2];
			matrix[i][1] = matrix[i-1][0] + steps[i-1];
			matrix[i][2] = matrix[i-1][1] + steps[i-1];
		}
		System.out.println((matrix[N][1] > matrix[N][2])? matrix[N][1]:matrix[N][2]);
		input.close();
	}
}
```

[https://github.com/qwlake/study-algorithm/blob/master/Java/baekjoon/src/n2579/n2579_v2.java](https://github.com/qwlake/study-algorithm/blob/master/Java/baekjoon/src/n2579/n2579_v2.java)