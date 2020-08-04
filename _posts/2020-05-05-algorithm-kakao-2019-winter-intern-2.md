---
layout: post
title: "2019 카카오 겨울 인턴 2번 - 튜플"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Programmers Kakao Intern Javascript
---

[https://programmers.co.kr/learn/courses/30/lessons/64065](https://programmers.co.kr/learn/courses/30/lessons/64065)

문제 이해를 잘 못 하는 사람이 많은데, 나도 코딩테스트때 좀 헤맸었다.

```jsx
"{{1,2,3},{2,1},{1,2,4,3},{2}}"
```

두 번째 입력 예제를 보면 각 원소의 개수가 모두 다른 것을 볼 수 있다.

이를 원소의 개수대로 오름차순 정렬을 하면

```jsx
"{{2},{2,1},{1,2,3},{1,2,4,3}}"
```

위와 같이 나온다.

앞에서부터 새로 추가되는 원소를 구해 정답 배열을 구할 수 있다.

위 예제에서는

1) 2 =>2 추가

2) 2,1 => 1 추가

3) 1,2,3 => 3 추가

4) 1,2,4,3 => 4 추가

측, [2,1,3,4]가 답이다.

다음은 자바스크립트 코드이다.

```jsx
function solution(s) {
    let answer = new Set();
    s = s.slice(1, s.length-1);
    let arr = [];
    let tmpArr = [];
    let tmp;
    let intSt;
    for (let i = 0; i < s.length; i++) {
        if (s[i] === '{') {
            intSt = i+1;
        } else if ((s[i] === ',' || s[i] === '}') && intSt !== -1) {
            tmp = parseInt(s.slice(intSt, i));
            tmpArr.push(tmp);
            intSt = i+1;
        }
        if (s[i] === '}') {
            arr.push(tmpArr);
            tmpArr = [];
            intSt = -1;
        }
    }

    arr.sort(function(a, b) {
        return a.length - b.length;
    });

    arr.forEach(e => {
        tmp = e.filter(x => !answer.has(x));
        answer.add(tmp[0]);
    });
    
    return Array.from(answer);
}

console.log(solution(
    "{{2},{2,1},{2,1,3},{2,1,3,4}}", 
));

console.log(solution(
    "{{20,111},{111}}",
));
```

[https://github.com/qwlake/study-algorithm/blob/master/Javascript/2019 카카오 겨울 인턴/n02.js](https://github.com/qwlake/study-algorithm/blob/master/Javascript/2019%20%EC%B9%B4%EC%B9%B4%EC%98%A4%20%EA%B2%A8%EC%9A%B8%20%EC%9D%B8%ED%84%B4/n02.js)