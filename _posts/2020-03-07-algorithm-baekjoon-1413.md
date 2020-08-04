---
layout: post
title: "백준 1413번 - 박스 안의 열쇠"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon C++ DP
---

[https://www.acmicpc.net/problem/1413](https://www.acmicpc.net/problem/1413)

코드를 보면 간단해 보이는데 애를 먹었던 문제다.

이 문제에서 원하는 출력은 열쇠를 얻는 경우의 수가 아닌 확률이다.

따라서 (주어진 폭탄만을 사용해 모든 상자를 연 경우의 수)/(모든 상자를 연 경우의 수) 의 형태로 출력해야 한다.

(주어진 폭탄만을 사용해 모든 상자를 연 경우의 수)는 주어진 폭탄을 사용해서 열쇠를 얻는 경우를 세면 되고,

(모든 상자를 연 경우의 수)는 폭탄의 개수에 상관 없이 모든 상자를 열 수 있다고 가정하고 모든 열쇠를 얻는 경우의 수를 세면 된다.

(모든 상자를 연 경우의 수)는 상자의 개수 만큼 M 팩토리얼을 해주면 쉽게 구할 수 있다.

문제는 (주어진 폭탄만을 사용해 모든 상자를 연 경우의 수)인데, 이 경우도 2가지로 나뉜다.

1. 보유한 열쇠를 사용해서 상자를 연 경우

2. 폭탄을 사용해서 상자를 연 경우

1번은 보유한 열쇠 중 어느 것을 사용할지 모르기 때문에 열쇠의 수만큼 가지수가 늘어난다.

또한 가용 폭탄의 개수는 그대로다.

이를 코드로 표현하면 다음과 같다.

```cpp
typedef long long ll;
ll loop(int m, int n) {
    if (n == 0) return 1;
    if (m == 0) return 0;
    return (n-1)*loop(m, n-1) + loop(m-1, n-1);
}
```

3,4번 줄은 재귀함수의 종료조건이다.

n==0 : 모든 상자를 깠다. > 상자를 연 경우의 수 1증가

m==0 : 상자가 남아 있지만 폭탄이 없다. > 더이상 진행할 수 없다

5번 줄에서 열쇠를 사용한 경우와 폭탄을 사용한 경우를 재귀함수로 호출한다.

전체 코드는 하단 링크에 있다.

[https://github.com/qwlake/study-algorithm/blob/master/C%2CC%2B%2B/baekjoon/n1413/n1149.cpp](https://github.com/qwlake/study-algorithm/blob/master/C%2CC%2B%2B/baekjoon/n1413/n1149.cpp)