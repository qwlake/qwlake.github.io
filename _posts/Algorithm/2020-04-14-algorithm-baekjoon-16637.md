---
layout: post
title: "백준 16637번 - 괄호 추가하기"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon Python
---

[https://www.acmicpc.net/problem/16637](https://www.acmicpc.net/problem/16637)

모든 자리에 괄호를 대입해 완전탐색하여 최대값을 찾아야 한다.

나는 그냥 간단하게 재귀 탐색으로 풀었다.

내가 푼 코드 중에서 다음과 같은 함수가 있는데

```python
def ev(a,b,c):
    return eval(str(a)+b+str(c))
```

이는 정수 a, c를 b에 대한 연산자로 계산하여 반환하라는 뜻이다.

즉, ev(2,+,3)=5

다음은 파이썬 전체 코드이다.

```python
def ev(a,b,c):
    return eval(str(a)+b+str(c))

def loop(a, num, idx1, idx2):
    tmp = 0
    if idx2-idx1 == 4:
        tmp = ev(a[idx2-2], a[idx2-1], a[idx2])
        tmp = ev(num, a[idx2-3], tmp)
    else:
        tmp = ev(num, a[idx2-1], a[idx2])
    if idx2 == N-1:
        return tmp
    x = loop(a, tmp, idx2, idx2+2)
    y = loop(a, tmp, idx2, idx2+4) if idx2+4 < N else x
    return max(x,y)

N = int(input())
a = [int(e) if i%2==0 else e for i, e in enumerate(list(input()))]

if N == 1:
    print(a[0])
elif N == 3:
    print(ev(a[0], a[1], a[2]))
else:
    print(max(loop(a, a[0], 0, 4), loop(a, a[0], 0, 2)))
```

[https://github.com/qwlake/study-algorithm/blob/master/Python/baekjoon/n16637/n16637.py](https://github.com/qwlake/study-algorithm/blob/master/Python/baekjoon/n16637/n16637.py)