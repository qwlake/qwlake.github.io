---
layout: post
title: "백준 2659번 - 십자카드 문제"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon KOI Python
---

[https://www.acmicpc.net/problem/2659](https://www.acmicpc.net/problem/2659)

먼저 입력받은 숫자를 시계수로 만들고, 이보다 작은 시계수의 개수를 구하는 방법으로 풀었다.

지금 보니 이게 최선의 알고리즘인가 싶다.

무엇보다

```python
t = str(i)+str(j)+str(k)+str(l)
```

이 문장이 마음에 안 드는데, 이는 f스트링을 사용해서 메모리를 더 적게 사용할 수 있다.

```python
t = f"{i}{j}{k}{l}"
```

성능도 좋고 가독성도 좋으니 애용하도록 하자.

다음은 전체 코드이다.

```python
n = input().split(" ")

def find(num):
    a = [num]
    for i in range(1,4):
        num = num[1:]+num[0:1]
        a.append(num)
    return min(a)

n = find(n)
a = []
for i in range(int(n[0]), 0, -1):
    for j in range(int(n[1]), 0, -1):
        for k in range(int(n[2]), 0, -1):
            for l in range(int(n[3]), 0, -1):
                t = str(i)+str(j)+str(k)+str(l)
                if t == find(t):
                    a.append(t)
            n[3] = "9"
        n[2] = "9"
    n[1] = "9"
print(len(a))
```

[https://github.com/qwlake/study-algorithm/blob/master/Python/baekjoon/KOI1997/n2659/n2659.py](https://github.com/qwlake/study-algorithm/blob/master/Python/baekjoon/KOI1997/n2659/n2659.py)