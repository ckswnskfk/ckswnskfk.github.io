---
title: "[F-Lab 모각코 챌린지] 22일차"
excerpt: "Meta Programming 2 - Reflect"
categories:
  - Javascript
tags:
  - f-lab
  - 에프랩
  - TIL
  - Javascript
  - Meta Programming
  - Reflect
last_modified_at: 2023-06-10
toc: true
---

오늘은 `Reflect`를 알아보자. Reflect는 사전적으로 '반영' 이란 뜻을 갖고 있다. 왜 이런 이름을 가지게 되었는지 천천히 알아보자.

# Reflect

`Reflect`는 프록시와 비슷하게 중간에서 작업을 가로채 그 작업에 대한 메소드를 제공하는 내장 객체이다. 어제 살펴본 프록시처럼 내부 메서드가 그 트리거이다.

`Reflect`는 함수 객체가 아닌 일반 객체이므로 new 연산자를 쓰지 않고, Math 객체처럼 정적이다. 메소드의 종류는 어제 살펴본 프록시의 트랩 메소드와 같다.

먼저 사용법을 간단히 알아보자.

```javascript
let user = {};

Reflect.set(user, "name", "John");

console.log(user.name); // John
```

`Reflect`를 사용하면 함수처럼 연산자(new, delete 등)를 호출할 수 있다(Reflect.construct, Reflect.deleteProperty 등). 필요할 때 `Object` 대신 더 자연스러운 네임 스페이스인 `Reflect`를 사용할 수 있다. 아래의 예시를 보자.

```javascript
// 1. 에러 핸들링
const obj = {};
try {
  Object.defineProperty(obj, "prop", { value: 1 });
  console.log("success");
} catch (e) {
  console.log(e);
}

const obj2 = {};
if (Reflect.defineProperty(obj2, "prop", { value: 1 })) {
  console.log("success");
} else {
  console.log("problem creating prop");
}
```

## with Pxory

`Reflect`는 주로 `Proxy`와 함께 사용된다. `Reflect`의 메소드중 동일한 이름과 인자를 가진 `Proxy` 트랩과 대응 되는 것들이 있다. 따라서 `Reflect`를 사용하면 `Proxy`에 의해 가로챈 작업을 원래 객체로 전달할 수 있다. 아래의 예시를 보자.

```javascript
let user = {
  name: "John",
};

user = new Proxy(user, {
  get(target, prop, receiver) {
    alert(`GET ${prop}`);
    return Reflect.get(target, prop, receiver);
  },
  set(target, prop, val, receiver) {
    alert(`SET ${prop}=${val}`);
    return Reflect.set(target, prop, val, receiver);
  },
});

let name = user.name; // shows "GET name"
user.name = "Pete"; // shows "SET name=Pete"
```

프록시의 get 트랩과 set 트랩에서 `Reflect`를 사용해 반환하고 있다. 사실 `Reflect`를 사용하지 않아도 동일한 작업을 할 수 있다. (ex. get의 경우 `target[prop]`를 반환)

그렇다면 이렇게 쓰는 이유가 따로 있는걸까?

### Reflect.get 이 더 좋은 이유

아래의 예제를 보자.

```javascript
let user = {
  _name: "Guest",
  get name() {
    return this._name;
  },
};

let userProxy = new Proxy(user, {
  get(target, prop, receiver) {
    return target[prop];
  },
});

alert(userProxy.name); // Guest
```

프로퍼티로 `_name` 과 접근자 getter인 `name()`이 있는 `user` 객체가 있고, `user` 객체를 감싼 프록시 객체 `userProxy`가 있다. 그리고 `userProxy`의 get 트랩은 `target[prop]`을 반환하고 있다.

이대로 아무 문제가 없어 보이지만, 좀 더 복잡한 상황이 추가되면 어떨까? 위의 예제 바로 아랫줄에서 이런 일이 발생하고 있다.

```javascript
let admin = {
  __proto__: userProxy,
  _name: "Admin",
};

console.log(admin.name); // Guest
```

`admin.name`을 읽을때, `admin`에는 `name`프로퍼티가 없으므로 프로토타입 체인상 가장 가까이 있는 `name`을 찾으려 한다. 그리고 `admin`의 가장 가까운 프로토타입은 `userProxy` 이다. 때문에 `userProxy`에서 `name`을 읽으려고 하면 get 트랩이 동작하고 `target[prop]` 이 반환된다. `prop`이 getter일 때, this는 `target`이 된다. 따라서 결과는 원본 객체 `target`인 `user`의 `this._name`인 "Guest"가 된다.

결국 `admin`을 this로 전달해줘야 하는데, get 트랩의 세 번째 인자인 `receiver`가 그 역할을 한다. 하지만 우리의 get 트랩을 보면 받은 `receiver`를 처리하지 않고 있다. 만약 일반 함수를 호출하는 상황이면 `call`이나 `apply`를 사용할 수 있겠지만, getter는 함수가 아니라 접근자 프로퍼티이기 때문에 호출되지 않고 다른 프로퍼티처럼 접근된다.

그래서 `Reflect.get`를 사용한다. 드디어. 프록시 객체 get 트랩 단 한줄만 바꾸면 된다.

```javascript
let userProxy = new Proxy(user, {
  get(target, prop, receiver) {
    // receiver = admin
    return Reflect.get(target, prop, receiver); // (*)
  },
});
```

이제 `admin` this를 유지하고 전달할 수 있다. 모든 인자를 사용하니 더 짧게 쓸 수도 있다.

```javascript
let userProxy = new Proxy(user, {
  get(target, prop, receiver) {
    return Reflect.get(...arguments);
  },
});
```

이렇듯 `Reflect`는 프록시의 트랩과 동일한 이름과 인자를 사용하게 설계 돼 있다. 따라서 `Reflect`는 작업을 안전하게 전달하고 해당 작업과 관련된 내용을 빠뜨리지 않도록 하는 간편한 방법을 제공한다.

\*참고: <https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Reflect>,
<https://ui.toast.com/posts/ko_20210413>,
<https://velog.io/@longroadhome/%EB%AA%A8%EB%8D%98JS-Proxy%EC%99%80-Reflect>,
<https://ko.javascript.info/proxy>
