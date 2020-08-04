---
layout: post
title: "2019 카카오 겨울 인턴 4번 - 호텔 방 배정"
subtitle: ""
author: qwlake
categories: Algorithm
tags: Algorithm Programmers Kakao Intern Javascript
---

[https://programmers.co.kr/learn/courses/30/lessons/64063](https://programmers.co.kr/learn/courses/30/lessons/64063)

# 연결 리스트(Linked List)

입력 배열의 크기가 최대 20만 이므로, 꽤 크다. 방이 이미 배정 됐을때 그 다음 방을 찾기 위해서 순차 탐색만 하면 시간이 너무 오래 걸린다.

비어 있는 방을 빠르게 찾기 위해서는 연결 리스트(Linked List)의 개념을 쓰면 된다.

자바스크립트에서는 Map(), 파이썬에서는 dict()로 구현 가능하다.

예를 들면 다음과 같은 관계가 있을때.

1->2

3->4

{1:2, 3:4} 와 같이 표현 가능하다.

# 그렇다면 문제에 적용시켜 보자

![image](https://user-images.githubusercontent.com/41278416/89281663-5eeff500-d685-11ea-91dd-867f7c0d6299.png)

방을 배정할때 경우의 수는 다음과 같다.

1. 방이 비어 있을때

2. 방이 이미 배정됐을때

1번의 경우는 방을 배정하고(answer 배열에 추가), map에 관계를 삽입한다.

삽입시 key는 (배정되는 방 번호), value는 (배정되는 방 번호)+1이다.

즉, key는 배정된 방 번호이고, value는 key의 방이 배정됐을때 참조해야할 다음 방 번호를 가르킨다.

관계는 다음과 같다.

{ 1 => 2,

3 => 4,

4 => 5}

2번의 경우에는 원하는 방이 가르키고 있는 방을 연속적으로 탐색해 나간다.

{ 1 => 2,

3 => 4,

4 => 5 }

에서 1번 방을 배정 하려는데 이미 배정돼있다.

이런 경우 **1번 방이 가르키고 있는 2번 방**을 본다.

2번 방은 map의 key들 중에 없다. 즉, 배정 가능한 방이다.

그리고 방들이 가르키고 있는 것들을 따라 탐색하는 과정 중에 있는 방들도 업데이트한다. 배정 결과는 다음과 같다.

{ **1 => 3**, // 업데이트

3 => 4,

4 => 5,

**2 => 3** } // 추가

이어서 3번 방도 배정하지만 이것 역시 이미 배정돼있다.

3번은 4번을 가르키고 4번은 5번을 가르킨다. 5번은 map에서 key로 존재하지 않으므로 배정 가능하다. 거쳐온 3번 4번 방을 최종 노드인 6번 방으로 업데이트한다.

{ 1 => 3,

**3 => 6**, // 업데이트

**4 => 6**, // 업데이트

2 => 3,

**5 => 6** } // 추가

이런 식으로 마지막 1번도 1번, 3번 방을 거쳐 6번으로 할당됐다.

{ **1 => 7**, // 업데이트

**3 => 7**, // 업데이트

4 => 6,

2 => 3,

5 **=**> 6,

**6 => 7** } // 추가

설명이 어려우면 [카카오 테크 블로그](https://tech.kakao.com/2020/04/01/2019-internship-test/)에서 다시 한 번 보자.

다음은 자바스크립트 코드이다.

```jsx
function solution(k, room_number) {
    let answer = [];
    let link = new Map();
    let tmp;
    let tmp_list;
    room_number.forEach(e => {
        if (!link.has(e)) {
            link.set(e, e+1);
            answer.push(e);
        } else {
            tmp = e;
            tmp_list = [];
            while (link.has(tmp)) {
                tmp_list.push(tmp);
                tmp = link.get(tmp);
            }
            link.set(tmp, tmp + 1);
            tmp_list.forEach(t => {
                link.set(t, tmp + 1);
            });
            answer.push(tmp);
        }
        console.log(link);
    });
    return answer;
}

// console.log(solution( // result [1,3,4,2,5,6]
//     10, 
//     [1,3,4,1,3,1]
// ));
console.log(solution( // result [1,2,3,4,5,6,7,8,9]
    100, 
    [1,1,1,1,5,5,5,5,3]
));
```

[https://github.com/qwlake/study-algorithm/blob/master/Javascript/2019 카카오 겨울 인턴/n04.js](https://github.com/qwlake/study-algorithm/blob/master/Javascript/2019%20%EC%B9%B4%EC%B9%B4%EC%98%A4%20%EA%B2%A8%EC%9A%B8%20%EC%9D%B8%ED%84%B4/n04.js)