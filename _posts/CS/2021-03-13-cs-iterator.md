---
layout: post
title: "[CS] iterator"
author: qwlake
categories: CS
tags: -
---

# iterator

iterator는 해당 collection 객체를 참조한다. (shallow copy)

→ 값을 조작하려면 객체 내부값(필드)을 조작해야 한다.

iterator 사용시 collection 객체에 대해 add나 remove를 막아놓은 이유

→ iterator가 참조하던 위치가 틀어지기 때문

→ 막는 방법: iterator.next() 시에 collection 객체의 변동이 발생했는지 modCount를 감지

→ modCount 변경시 Exception 발생

다른 대안으로는 CopyOnWriteArrayList 가 있음

→ iterator 생성시 원본 collection의 스냅샷을 찍어 반영

→ 매 iterator 생성시 객체의 생성이 발생하기 때문에 느림

→ 대신 thread-safe를 보장 보장

![image](https://user-images.githubusercontent.com/41278416/130633831-3dc04c3a-0524-4148-b6ed-20911eb038b1.png)

[How to change value of ArrayList element in java](https://stackoverflow.com/questions/12772443/how-to-change-value-of-arraylist-element-in-java)

[Iterator의 내부동작과 ConcurrentModificationException을 피하는 방법](https://brandpark.github.io/java/2021/01/24/iterator.html)

[Thread Safety and how to achieve it in Java - GeeksforGeeks](https://www.geeksforgeeks.org/thread-safety-and-how-to-achieve-it-in-java/)

[참고) [List] Concurrent List 만들기](https://okihouse.tistory.com/entry/List-Concurrent-List-%EB%A7%8C%EB%93%A4%EA%B8%B0)

## Stream

[Java 스트림 Stream (1) 총정리](https://futurecreator.github.io/2018/08/26/java-8-streams/)

[자바 스트림(Stream) API 정리, 스트림을 이용한 가독성 좋은 코드 만들기(feat. 자바 람다, 함수형 프로그래밍, 자바8)](https://jeong-pro.tistory.com/165)

[for-loop 를 Stream.forEach() 로 바꾸지 말아야 할 3가지 이유](https://homoefficio.github.io/2016/06/26/for-loop-%EB%A5%BC-Stream-forEach-%EB%A1%9C-%EB%B0%94%EA%BE%B8%EC%A7%80-%EB%A7%90%EC%95%84%EC%95%BC-%ED%95%A0-3%EA%B0%80%EC%A7%80-%EC%9D%B4%EC%9C%A0/)

# 구현코드
https://github.com/qwlake/oscar/tree/main/iterator