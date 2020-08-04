---
layout: post
title: "2019 카카오 겨울 인턴 1번 - 쇠막대기"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Programmers Kakao Intern Javascript
---

[https://programmers.co.kr/learn/courses/30/lessons/42585](https://programmers.co.kr/learn/courses/30/lessons/42585)

자바스크립트(javascript) 연습겸 풀었던 문제다.

문제 자체는 간단하다. 스택을 사용해서 괄호가 닫히는 경우만 잘 처리 해주면 된다.

괄호가 닫혔을때 취해야 하는 경우의 수는 두 가지인데,

1. 레이저 위치인 경우'()'

2. 쇠막대기가 끝나는 경우')'

로 볼 수 있다.

레이저 위치인 경우에는 스택에 쌓아 두었던 쇠막대기의 개수 만큼을 정답 변수에 추가한다.

쇠막대기가 끝나는 경우는 정답 변수를 1 증가시킨다.

다음은 자바스크립트 코드이다.

```jsx
function solution(arrangement) {
    let stack = 0;
    let close = false;
    let answer = 0;
    for (let i = 0; i < arrangement.length; i++) {
        if (arrangement[i] === "(") {
            stack += 1;
            close = true;
        } else if (close) {
            stack -= 1;
            answer += stack;
            close = false;
        } else {
            stack -= 1;
            answer += 1;
        }
        // console.log(i, arrangement.slice(0,i+1), stack, answer);
    }
    return answer;
}

console.log(solution("()(((()())(())()))(())"));
```

[https://github.com/qwlake/study-algorithm/blob/master/Javascript/programmers/n42585/n42585.js](https://github.com/qwlake/study-algorithm/blob/master/Javascript/programmers/n42585/n42585.js)