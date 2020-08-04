---
layout: post
title: "백준 11726번 - 2×n 타일링"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Baekjoon Python Fibonacci
---

일단 이 문제는 피보나치 수열의 패턴을 갖는 문제다.

따라서 다음과 같이 짧은 코드로 풀 수 있다.

파이썬 코드

```python
cache = [1 for _ in range(int(input()) + 1)]
for i in range(2, len(cache)):
    cache[i] = cache[i-2] + cache[i-1]
print(int(cache[-1] % 10007))
```

풀고 나서 인터넷에서 다른 코드를 참고해 풀어봤는데 파이썬 기준으로 4ms 차이가 난다. 그냥 짧은 코드가 좋은 듯 하다.

아래는 인터넷에서 찾은 여러 가지 코드

```python
import time

def fib1(n):
    cache = [1 for _ in range(n + 1)]
    for i in range(2, len(cache)):
        cache[i] = cache[i-2] + cache[i-1]
    return int(cache[-1] % 10007)

def fib2(n, __cache={0: 1, 1: 1}):
    if n in __cache:
        return __cache[n]

    __cache[n] = fib2(n-1) + fib2(n-2)
    return __cache[n]

def fib3(n):
    cache = [-1 for _ in range(n+1)]

    def iterate(n):
        # 기저사례 1.
        if n < 2:
            return n

        # 기저사례 2.
        if cache[n] != -1:
            return cache[n]

        # 기저사례 충족 못할 시 값을 실제로 구함
        cache[n] = iterate(n-1) + iterate(n-2)
        return cache[n]

    return iterate(n)

def fib4(n):
    SIZE = 2
    ZERO = [[1, 0], [0, 1]] # 행렬의 항등원
    BASE = [[1, 1], [1, 0]] # 곱셈을 시작해 나갈 기본 행렬

    # 두 행렬의 곱을 구한다
    def square_matrix_mul(a, b, size=SIZE):
        new = [[0 for _ in range(size)] for _ in range(size)]

        for i in range(size):
            for j in range(size):
                for k in range(size):
                    new[i][j] += a[i][k] * b[k][j]

        return new

    # 기본 행렬을 n번 곱한 행렬을 만든다
    def get_nth(n):
        matrix = ZERO.copy()
        k = 0
        tmp = BASE.copy()

        while 2 ** k <= n:
            if n & (1 << k) != 0:
                matrix = square_matrix_mul(matrix, tmp)
            k += 1
            tmp = square_matrix_mul(tmp, tmp)

        return matrix

    return get_nth(n)[1][0]

def fib5(n):
    sqrt_5 = 5 ** (1/2)
    ans = 1 / sqrt_5 * ( ((1 + sqrt_5) / 2) ** n  - ((1 - sqrt_5) / 2) ** n )
    return int(ans)

def fib6(n):
    a, b = 1, 1
    for i in range(2, n+1):
        if i % 2 == 0:
            a = a + b
        else:
            b = a + b
    return max(a,b)

n = int(input())
# start = time.time()
# print(fib1(n))
# print(time.time() - start)

# start = time.time()
# print(fib2(n) % 10007)
# print(time.time() - start)

# start = time.time()
# print(fib3(n) % 10007)
# print(time.time() - start)

# start = time.time()
# print(fib4(n+1) % 10007)
# print(time.time() - start)

# start = time.time()
# print(fib5(n+1) % 10007)
# print(time.time() - start)

start = time.time()
print(fib6(n) % 10007)
# print(time.time() - start)
```

[https://github.com/qwlake/study-algorithm/blob/dev/Python/baekjoon/n11726/n11726.py](https://github.com/qwlake/study-algorithm/blob/dev/Python/baekjoon/n11726/n11726.py)