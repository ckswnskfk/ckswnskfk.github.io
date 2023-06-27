---
title: "[F-Lab 모각코 챌린지] 39일차"
excerpt: "프라미스 에러 핸들링"
categories:
  - Javascript
tags:
  - f-lab
  - 에프랩
  - TIL
  - Javascript
  - Promise
  - Error Handling
last_modified_at: 2023-06-27
toc: true
---

얼마전 프라미스에 대해 알아볼 때 `then`과 `catch`에 대해서도 같이 알아보았다. 오늘은 프라미스 에러 핸들링에 대한 내용을 좀 더 자세히 알아보자.

# 암시적 try...catch

프라미스 내부 실행 함수와 핸들러 함수(then, catch) 주위엔 암시적인 `try...catch` 문이 있다. 예외가 발생하게 되면 여기서 예외를 잡고 이를 reject처럼 취급한다.

아래의 코드를 보자.

```javascript
new Promise((resolve, reject) => {
  throw new Error("에러 발생!");
}).catch(alert); // Error: 에러 발생!

new Promise((resolve, reject) => {
  reject(new Error("에러 발생!"));
}).catch(alert); // Error: 에러 발생!
```

두 코드는 똑같이 작동한다. 별도로 `reject()`를 호출하지 않아도 `catch()`로 넘어가는 것이다. 이런 일은 실행 함수뿐만 아니라 핸들러 함수에서도 발생한다. 아래 코드를 보자.

```javascript
new Promise((resolve, reject) => {
  resolve("OK");
})
  .then((result) => {
    throw new Error("에러 발생!");
  })
  .catch(alert); // Error: 에러 발생!
```

`then` 핸들러 안에서 throw를 사용해 에러를 던지면, 이 자체가 거부된 프라미스를 의미한다. 따라서 제어 흐름이 가장 가까운 에러 핸들러(`catch`)로 넘어간다.

```javascript
new Promise((resolve, reject) => {
  resolve("OK");
})
  .then((result) => {
    blabla(); // 존재하지 않는 함수
  })
  .catch(alert); // ReferenceError: blabla is not defined
```

`throw`문이 만든 에러뿐만 아니라 모든 종류의 에러가 암시적 `try..catch`에서 처리된다. 암시적 `try..catch`가 오류를 만나면 에러 핸들러로 자동으로 넘어가게 한다.

# 다시 던지기

`then` 핸들러를 원하는 만큼 사용하다 마지막에 `catch` 하나만 붙이면 `then` 핸들러에서 발생한 모든 에러를 처리할 수 있다.

일반 `try..catch`에선 에러를 분석하고, 처리할 수 없는 에러라 판단되면 에러를 다시 던질 때가 있다. 프라미스에도 유사한 일을 할 수 있다. `catch` 안에서 throw를 사용하면 제어 흐름이 가장 가까운 곳에 있는 에러 핸들러로 넘어간다. 여기서 에러가 성공적으로 처리되면 가장 가까운 곳에 있는 `then` 핸들러로 제어 흐름이 넘어가 실행이 이어진다.

```javascript
new Promise((resolve, reject) => {
  throw new Error("에러 발생!");
})
  .catch(function (error) {
    alert("에러가 잘 처리되었습니다. 정상적으로 실행이 이어집니다.");
  })
  .then(() => alert("다음 핸들러가 실행됩니다."));
```

여기서 `catch`는 에러를 성공적으로 처리한다.

# 오래된 콜백 API를 사용해 Promise 만들기

이상적인 모던 자바스크립트 환경은 모든 비동기 함수가 promise를 반환하는 것일 것이다. 하지만 `setTimeout()` 같은 함수는 여전히 success 혹은 failure 콜백을 전달하는 방식이다.

문제는 콜백 함수가 실패하거나 오류가 있어도 그 어떤 핸들링도 할 수 없는 것이다. 그럼 어떻게 오래된 콜백 api를 pomise로 만들 수 있을까?

```javascript
const wait = (ms) => new Promise((resolve) => setTimeout(resolve, ms));

wait(10000)
  .then(() => saySomething("10 seconds"))
  .catch(failureCallback);
```

가장 좋은 방법은 가능한 가장 낮은 수준에서 문제가 되는 오래된 콜백 api를 감싼 다음 다시는 직접 호출하지 않는 것이다. 기본적으로 promise constructor는 promise를 직접 해결하거나 `reject` 할 수 있는 실행자 함수를 사용한다. `setTimeout()`은 함수에서 fail이 일어나거나 error가 발생하지 않기 때문에 이 경우 `reject`를 사용하지 않는다.

\*참고: <https://ko.javascript.info/promise-error-handling>,  
<https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Using_promises>
