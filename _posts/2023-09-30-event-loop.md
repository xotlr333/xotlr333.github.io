---
layout: post
title: Event Loop
date: 2023-09-30
categories: [JavaScript, 이론]
tags: [동작원리]
---

## 프로세스 및 스레드

- 프로세스는 실행되고 있는 컴퓨터 프로그램이라고 생각하시면 됩니다. 게임, 웹 브라우저 등 컴퓨터에서 실행하고 있는 프로그램을 프로세스라고 합니다.
- 프로세스 안에서 실제로 일처리를 하고 있는 것이 스레드 입니다. 하나의 스레드는 한가지 일만 할 수 있습니다. 프로세스가 식당이고 스레드가 직원이라고 할 경우, 직원이 많을 수록 음식을 빨리 만들 수 있는 것 처럼 스레드가 많으면 동시에 일처리가 가능하여 속도가 빠릅니다.

## 싱글 스레드 VS 멀티 스레드

- 싱글 스레드
  - 하나의 프로세스가 한번에 하나의 일만 처리하는 것
- 멀티 스레드
  - 하나의 프로세스에서 여러 개의 스레드르 가지고 있는 것을 말합니다
  - 각 스레드가 작업을 각각 수행합니다
  - 예시 - 멀티 스레드가 적용된 웹 브라우저에서 하나의 스레드는 이미지를 다운받고 있는 동안에 다른 스레드로 웹 서핑을 할 수 있습니다.
  - 멀티 스레드일 경우 프로세스 안의 스레드끼리 자원을 공유하기때문에 자원 관리에 주의해야한다는 단점이 있습니다. 동시에 같은 자원을 사용할 경우 문제가 발생할 수 있습니다.

![스레드](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/fe4a0a86-2306-49bc-9717-18cbfff4dc70)

## 자바스크립트는 싱글 스레드이다

자바스크립트는 웹 페이지의 보조적인 기능을 수행하기 위해 브라우저에서 동작하는 경량 프로그래밍 언어를 도입하고자 만든 언어입니다. 멀티 스레드인 경우, 동시성 문제를 많이 신경써야하기에 경량 프로그래밍 언어와는 맞지 않습니다. 그래서 자바스크립트는 싱글 스레드를 선택하게 되었습니다.

## 자바스크립트 엔진(javascript engine)

- 자바스크립트 코드를 실행하는 프로그램
- 대체적으로 웹 브라우저에 포함되어 있다
- 런타임이란 - 자바스크립트가 엔진에 의해 해석되고 실행되는 것을 말한다
- 브라우저 별로 엔진이 다르다
- 크롬과 node.js에서는 Google V8 이라는 엔진을 사용한다

## 자바스크립트 엔진의 동작 원리

자바스크립트는 콜 스택과 메모리 힙이라는 메모리 구조를 통해 데이터 및 코드 실행을 관리한다.

1. 메모리 힙(Memory Heap) : 메모리 할당이 일어나는 곳
   1. 참조타입(객체, 배열, 함수 등ㅇ) 데이터가 저장된다.
2. 콜 스택(Call Stack): 코드 실행에 따라 호출 스택이 쌓이는 곳

   1. 자바스크립트는 싱글 스레드이기 때문에 콜 스택이 하나이기에 한 번에 한 작업만 처리할 수 있다
   2. 함수의 우선 순위는 전역함수 → 지역함수 순으로 쌓인다.
   3. 함수 실행이 끝나면 해당 스택은 콜 스택에서 제거한다

   <img width="738" alt="콜스택 동작 예시" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/f82b7571-3bb7-4e3e-8d83-654b56080a6d">

   콜 스택 동작 예시

![자바스트립트 엔지 시작화](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/d742c6e9-6257-4767-a38c-ebff65e8d7a0)

자바스크립트 엔지 시각화

## 웹에서 자바스크립트 동작

자바스크립트 엔진만으로 웹이 동작하지 않는다. 자바 스크립트는 싱글 스레드이기 때문에 동기적인 처리만 가능합니다. 하지만 웹에서 비동기적인 동작도 가능합니다. 그것은 자바스크립트 이외의 요소들과 함께 런타임을 이루고 있기 때문입니다. 여기서 비동기적인 처리를 도와주는 요소는 브라우저가 제공합니다.

1. Web API : 브라우저에서 지원하는 api
   1. DOM이벤트, ajax, settimeout 등의 비동기 작업을 수행할 수 있도록 지원하는 api이다
2. task queue : 비동기 함수의 콜백 함수가 일시적으로 보관되는 영역
   1. 태스크의 큐는 두가지 종류가 있다. 태스크 큐와 마이크로태스크 큐가 있다. 각각 보관하는 콜백함수의 종류가 다릅니다.
   2. 태스크 큐에 저장되는 콜백함수는 setTimeout, setInterval 등이 있다
   3. 마이크로태스크 큐에는 Promises, queueMicrotask() 등이 있다
   4. 실행되는 우선 순위는 마이크로태스트 큐가 높다
3. 이벤트 루프 : 콜 스택과 태스크 큐의 상태를 확인하여 콜 스택이 빈 상태가 되면 태스트 큐에 있는 콜백함수를 순차적으로 콜 스택에 넣어주는 역할
   1. 이런 반복적인 행동을 틱(tick)이라 부릅니다.

# 참고 영상

[https://www.youtube.com/watch?v=8aGhZQkoFbQ](https://www.youtube.com/watch?v=8aGhZQkoFbQ)

참고)

[https://miracleground.tistory.com/entry/자바스크립트는-왜-싱글-스레드를-선택했을까-프로세스-스레드-비동기-동기-자바스크립트-엔진-이벤트루프](https://miracleground.tistory.com/entry/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EB%8A%94-%EC%99%9C-%EC%8B%B1%EA%B8%80-%EC%8A%A4%EB%A0%88%EB%93%9C%EB%A5%BC-%EC%84%A0%ED%83%9D%ED%96%88%EC%9D%84%EA%B9%8C-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4-%EC%8A%A4%EB%A0%88%EB%93%9C-%EB%B9%84%EB%8F%99%EA%B8%B0-%EB%8F%99%EA%B8%B0-%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%97%94%EC%A7%84-%EC%9D%B4%EB%B2%A4%ED%8A%B8%EB%A3%A8%ED%94%84)

[https://lifeandit.tistory.com/145](https://lifeandit.tistory.com/145)

[https://velog.io/@kirin/자바스크립트-엔진JavaScript-engine](https://velog.io/@kirin/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%97%94%EC%A7%84JavaScript-engine)

[https://velog.io/@yejineee/이벤트-루프와-태스크-큐-마이크로-태스크-매크로-태스크-g6f0joxx](https://velog.io/@yejineee/%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EB%A3%A8%ED%94%84%EC%99%80-%ED%83%9C%EC%8A%A4%ED%81%AC-%ED%81%90-%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C-%ED%83%9C%EC%8A%A4%ED%81%AC-%EB%A7%A4%ED%81%AC%EB%A1%9C-%ED%83%9C%EC%8A%A4%ED%81%AC-g6f0joxx)

[https://developer.mozilla.org/ko/docs/Web/API/HTML_DOM_API/Microtask_guide](https://developer.mozilla.org/ko/docs/Web/API/HTML_DOM_API/Microtask_guide)
