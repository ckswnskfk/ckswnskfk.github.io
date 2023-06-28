---
title: "[F-Lab 모각코 챌린지] 40일차"
excerpt: "async/await"
categories:
  - Javascript
tags:
  - f-lab
  - 에프랩
  - TIL
  - Javascript
  - Promise
  - async/await
last_modified_at: 2023-06-28
toc: true
---

프라미스에 이어 `async`, `await`에 대해 알아보자. `async`와 `await`는 자바스크립트의 비동기 처리 패턴 중 가장 최근에 나온 문법이다. 프라미스를 좀 더 편하게 사용할 수 있게 해준다.

# async function

`async`는 function 앞에 위치하는 키워드이다.

```javascript
async function func() {
  return 10;
}
```

`async function` 선언은 `AsyncFunction`객체를 반환하는 비동기 함수를 정의한다(자바스크립트에서 모든 비동기 함수는 사실상 `AsyncFunction` 객체이다). 암시적으로 `Promise`를 사용하여 결과를 반환한다. 프라미스가 아닌 값을 반환하더라도 `resolve` 상태의 프라미스로 값을 감싸 이행된 프라미스가 반환된다.

한 마디로 하면, `async` 키워드가 붙은 함수의 반환 값은 항상 `resolve`된 함수다!

# await

`async` 함수에는 `await` 키워드가 쓰일 수 있다. `await`는 `async` 함수의 실행을 일시 중지하고 전달 된 `Promise`의 해결을 기다린 다음, `async` 함수의 실행을 다시 시작하고 완료후 값을 반환합니다. 즉, 자바스크립트는 `await` 키워드를 만나면 프라미스가 처리될 때까지 기다린다.

예제 코드를 보면 바로 이해할 수 있다.

```javascript
async function f() {
  let promise = new Promise((resolve, reject) => {
    setTimeout(() => resolve("완료!"), 1000);
  });

  let result = await promise; // 프라미스가 이행될 때까지 기다림

  alert(result); // "완료!"
}

f();
```

만약 `async` `await`가 없었다면 프라미스를 기다리지 않고 바로 얼럿창 까지 실행됐을 것이다. 지금의 코드는 `await`를 만나면 실행이 잠시 '중단’되었다가 프라미스가 처리되면 실행이 재개된다.

`await`는 `promise.then`보다 좀 더 세련되게 프라미스의 result 값을 얻을 수 있도록 해주는 문법입니다. `promise.then`보다 가독성 좋고 쓰기도 쉽습니다. 마치 프로그램이 동기적으로 동작하는 것 처럼 읽을 수 있다.

한 편, 프라미스가 처리되길 기다리는 동안엔 엔진이 다른 일(다른 스크립트를 실행, 이벤트 처리 등)을 할 수 있기 때문에, CPU 리소스가 낭비되지 않는다.

참고로 `async function`이 아닌 일반 함수에는 `await`를 쓸 수 없다.

# 일반적 사용

일반적으로 `await`의 대상이 되는 비동기 처리 코드는 `Axios` 등 프로미스를 반환하는 HTTP 통신 호출 함수이다.

특히 여러개의 비동기 처리를 할 때 가장 진가가 드러난다.

```javascript
async function showAvatar() {
  // JSON 읽기
  let response = await fetch("/article/promise-chaining/user.json");
  let user = await response.json();

  // github 사용자 정보 읽기
  let githubResponse = await fetch(`https://api.github.com/users/${user.name}`);
  let githubUser = await githubResponse.json();

  // 아바타 보여주기
  let img = document.createElement("img");
  img.src = githubUser.avatar_url;
  img.className = "promise-avatar-example";
  document.body.append(img);

  // 3초 대기
  await new Promise((resolve, reject) => setTimeout(resolve, 3000));

  img.remove();

  return githubUser;
}

showAvatar();
```

`promise.then`을 사용한 것 보다 훨씬 보기 깔끔하다.

# 에러 핸들링

프라미스가 정상적으로 이행되면 `await promise`는 프라미스 객체의 `result`에 저장된 값을 반환한다. 반면 프라미스가 거부되면 마치 throw문을 작성한 것처럼 에러가 던져진다.

```javascript
async function f() {
  await Promise.reject(new Error("에러 발생!"));
}

// 두 코드는 같은 동작을 함

async function f() {
  throw new Error("에러 발생!");
}
```

`await`가 던진 에러는 `throw`가 던진 에러를 잡을 때처럼 `try..catch`를 사용해 잡을 수 있다.

```javascript
async function f() {
  try {
    let response = await fetch("http://유효하지-않은-주소");
  } catch (err) {
    alert(err); // TypeError: failed to fetch
  }
}

f();
```

\*참고:
<https://ko.javascript.info/async-await>,  
<https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/async_function>,  
<https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/AsyncFunction>,
