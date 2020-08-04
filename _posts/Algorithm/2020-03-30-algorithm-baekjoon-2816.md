---
layout: post
title: "백준 2816번 - 디지털 티비"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon Java Greedy
---

[https://www.acmicpc.net/problem/2816](https://www.acmicpc.net/problem/2816)

이 문제는 스페셜저지다.

KBS1, KBS2 를 제외한 나머지 채널들의 순서가 상관 없고, 최소한 움직임 개수를 구하라고 한 것도 아니니(아마도?)

여러 가지의 답이 나올 수 있다.

나는 KBS1과 KBS2의 순번(인덱스)을 저장해서 해당 순번만큼 화살표를 아래로 움직이고(리모컨의 1번 기능)

아래로 움직인만큼 채널을 다시 위로 올렸다(리모컨 4번 기능).

따라서 1번 기능과 4번 기능만으로도 문제를 풀 수 있다.

자바 소스코드

```java
import java.util.Scanner;

public class Main {

	public static void main(String[] args) {
		Scanner input = new Scanner(System.in);
		int N = Integer.parseInt(input.nextLine());
		String[] channels = new String[N];
		StringBuilder orders = new StringBuilder();
		int k1 = 0;
		int k2 = 0;
		int temp;
		for (int i = 0; i < N; i++) {
			channels[i] = input.nextLine().trim();
			if (channels[i].compareTo("KBS1") == 0)
				k1 = i;
			else if (channels[i].compareTo("KBS2") == 0)
				k2 = i;
		}
		if (k1 > k2) {
			temp = k1;
			k1 = k2;
			k2 = temp;
			temp = 0;
		} else {
			temp = 1;
		}
		for (int i = 0; i < k1; i++)
			orders.append("1");
		for (int i = 0; i < k1; i++)
			orders.append("4");
		for (int i = 0; i < k2; i++)
			orders.append("1");
		k2 = (temp == 0)? k2+1:k2;
		for (int i = 0; i < k2-1; i++)
			orders.append("4");
		System.out.println(orders);
		input.close();
	}
}
```

[https://github.com/qwlake/study-algorithm/blob/master/Java/baekjoon/src/n2816/n2816_v2.java](https://github.com/qwlake/study-algorithm/blob/master/Java/baekjoon/src/n2816/n2816_v2.java)