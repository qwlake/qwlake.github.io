---
layout: post
title: "2019 카카오 겨울 인턴 3번 - 불량 사용자"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Programmers Kakao Intern Javascript
---

[https://programmers.co.kr/learn/courses/30/lessons/64064](https://programmers.co.kr/learn/courses/30/lessons/64064)

이 문제는 각 제재 아이디에 해당하는 응모 아이디를 선별해서 가능한 모든 조합을 구하면 된다. 포인트는 각 제재 아이디는 응모 아이디 하나에 무조건 매칭되어야 한다.

예를 들어

제재 아이디: `*a*`, `***`

응모 아이디: `aaa`, `bbb`

인 경우에 가능한 모든 경우의 수는 `[aaa,bbb]`, `[aaa]` 두 가지가 된다.

`[aaa]` 하나만 있는 경우는 제재 아이디 `***`가 응모 아이디 `aaa`에 매칭됐고, 제재 아이디 `*a*`가 매칭할 응모 아이디가 없는 상황이다.

따라서 `[aaa]` 하나만 있는 경우는 정답의 경우의 수에 포함되지 못한다.

이를 다시 말하면 매칭된 아이디의 개수가 제재 아이디 개수만큼 나와주면 정답의 경우의 수에 포함되는 것이고, 그렇지 않다면 포함하지 않는다.

따라서 일단 모든 경우의 수를 탐색(재귀 탐색)을 한 다음에, 매칭된 아이디의 개수가 제재 아이디의 개수인지 파악하면 된다.

일단 가장 나은 코드는 다음과 같다.

본 코드에서는 제재 아이디가 매칭 됐는지 여부와 정답의 조합을 Set()으로 저장해 관리한다.

```jsx
function loop(bans, idx, filterd) {
    let ret = new Set();
    if (filterd.length === bans.length) {
        let tmp = filterd.slice();
        tmp = tmp.sort().join('');
        ret.add(tmp);
    } else {
        let res;
        for (let i = 0; i < bans[idx].length; i++) {
            if (!filterd.includes(bans[idx][i])) {
                filterd.push(bans[idx][i]);
                res = loop(bans, idx+1, filterd);
                res.forEach(function(e) {
                    ret.add(e);
                });
                filterd.pop();
            }
        }
    }
    return ret;
}
function solution(user_id, banned_id) {
    let answer = 0;
    const re = (a, b) =>
        new RegExp(`^${b}$`).test(a);
    for (let i = 0; i < banned_id.length; i++) {
        banned_id[i] = banned_id[i].replace(/\*/g, '.');
    }
    let bans_list = [];
    banned_id.forEach(b => {
        bans_list.push(user_id.filter(u => re(u, b)));
    });
    answer = loop(bans_list, 0, []);
    return answer.size;
}

console.log(solution( // result 2
    ["frodo", "fradi", "crodo", "abc123", "frodoc"], 
    ["fr*d*", "abc1**"]
));
console.log(solution( // result 2
    ["frodo", "fradi", "crodo", "abc123", "frodoc"],
    ["*rodo", "*rodo", "******"]
));
console.log(solution( // result 3
    ["frodo", "fradi", "crodo", "abc123", "frodoc"],
    ["fr*d*", "*rodo", "******", "******"]
));
```

[https://github.com/qwlake/study-algorithm/blob/master/Javascript/2019 카카오 겨울 인턴/n03-1.js](https://github.com/qwlake/study-algorithm/blob/master/Javascript/2019%20%EC%B9%B4%EC%B9%B4%EC%98%A4%20%EA%B2%A8%EC%9A%B8%20%EC%9D%B8%ED%84%B4/n03-1.js)