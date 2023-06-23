---
title: "[F-Lab 모각코 챌린지] 35일차"
excerpt: "Promise Chaining"
categories:
  - Javascript
tags:
  - f-lab
  - 에프랩
  - TIL
  - Javascript
  - Promise
  - Promise Chaining
last_modified_at: 2023-06-23
toc: true
---

어제 프라미스에 대해 알아봤으니 오늘은 프라미스를 활용하는 방법 중 하나인 프라미스 체이닝에 대해 알아보자.

# 프라미스 체이닝

프라미스의 특장점중 하나는 여러개의 프라미스를 연결해 사용할 수 있다는 점이다. 콜백 지옥에서 알아본 것 처럼 스크립트를 불러오는 등의 비동기 처리를 순차적으로 해야할 때, 프라미스 체이닝을 통해 해결할 수 있다.

프라미스 체이닝은 `then`이 프라미스를 반환하고, `result`가 `then` 핸들러의 `resolve`를 통해 전달된다는 점에서 착안한 아이디어다.

```javascript
new Promise(function (resolve, reject) {
  setTimeout(() => resolve(1), 1000);
})
  .then(function (result) {
    alert(result);

    return new Promise((resolve, reject) => {
      setTimeout(() => resolve(result * 2), 1000);
    });
  })
  .then(function (result) {
    alert(result);

    return new Promise((resolve, reject) => {
      setTimeout(() => resolve(result * 2), 1000);
    });
  })
  .then(function (result) {
    alert(result);
  });
```

코드를 살펴보자. 첫 번째 `.then`은 1을 출력하고 new Promise(…)를 반환한다.
1초 후 이 프라미스가 `resolve`되고 그 결과(`resolve`의 인수인 result \* 2)는 두 번째 `.then`으로 전달된다. 두 번째 핸들러는 2를 출력하고 동일한 과정이 반복된다.

프라미스 하나에 `.then`을 여러개 붙히는 건 프라미스 체이닝이 아니다.

# fetch와 프라미스 체이닝

자바스크립트에서 http 요청이야말로 가장 대표적인 비동기 작업일 것이다.

```javascript
let promise = fetch(url, [options]);
```

`fetch()`는 받은 파라미터 정보를 바탕으로 네트워크 요청을 보내고 `response` 객체와 함께 프라미스가 반환된다. 아래의 예제를 보자.

```javascript
fetch("/article/promise-chaining/user.json")
  .then((response) => response.json())
  .then((user) => fetch(`https://api.github.com/users/${user.name}`))
  .then((response) => response.json())
  .then(
    (githubUser) =>
      new Promise(function (resolve, reject) {
        let img = document.createElement("img");
        img.src = githubUser.avatar_url;
        img.className = "promise-avatar-example";
        document.body.append(img);

        setTimeout(() => {
          img.remove();
          resolve(githubUser);
        }, 3000);
      })
  )
  // 3초 후 동작함
  .then((githubUser) =>
    alert(`${githubUser.name}의 이미지를 성공적으로 출력하였습니다.`)
  );
```

이 코드의 실행 순서를 살펴보자.

1. 첫 번째 `fetch`: 해당 url로 http 요청을 보낸 후 응답을 `resolve`한다.
2. 첫 번째 `then`: 응답을 json 형태로 바꾼 후 `resolve`한다.
3. 두 번째 `then`: json 형태로 바뀐 user 정보로 http 요청을 보낸 후 `resolve`한다.
4. 세 번째 `then`: 받아온 응답을 json으로 바꾼 후 `resolve`한다.
5. 네 번째 `then`: img 태그를 만들어 받아온 깃허브 유저 정보의 아바타 url을 src로 설정하고 문서 객체에 append 한다. 3초 뒤 이미지 태그를 삭제하고 받아온 깃허브 유저 정보를 `resolve`한다.
6. 다섯번 째 `then`: 받아온 깃허브 유저 정보를 바탕을 메시지를 출력한다.

비동기 동작은 항상 프라미스를 반환하도록 하는 것이 좋다.
