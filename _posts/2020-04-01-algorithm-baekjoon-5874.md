---
layout: post
title: "백준 5874번 - 소를 찾아라"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon Java
---

[https://www.acmicpc.net/problem/5874](https://www.acmicpc.net/problem/5874)

풀 때에는 몰랐는데 지금 보니 문제가 좀 이상하다.

힌트도 예제 입력과 다른 것 같고, 출력의 설명도 뭔가 빠져있다.

문제 자체는 간단하다. (( 과 ))을 세어서 나올 수 있는 순서쌍을 구하면 된다.

자세히는 "("의 개수를 ")"를 찾은 개수만큼 더한다. ( 다음에 오는 ")"를 찾으면 그 전에 있던 모든 "(" 개수를 더하기 때문에 모든 경우의 수를 파악할 수 있다. )

자바 코드

```java
import java.util.Scanner;

public class n5874_v2 {
	public static void main(String[] args) {
		Scanner input = new Scanner(System.in);
		while (input.hasNext()) {
			String[] line = input.nextLine().split("");
			int count = 0;
			int sum = 0;
			for (int i = 1; i < line.length; i++) {
				if (line[i].compareTo(line[i-1]) == 0) {
					if (line[i].compareTo("(") == 0)
						count++;
					else
						sum += count;
				}
			}
			System.out.println(sum);
		}
		input.close();
	}
}
```

[https://github.com/qwlake/study-algorithm/blob/master/Java/baekjoon/src/n5874/n5874_v2.java](https://github.com/qwlake/study-algorithm/blob/master/Java/baekjoon/src/n5874/n5874_v2.java)