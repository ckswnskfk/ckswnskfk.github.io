---
title: "[F-Lab 모각코 챌린지] 21일차"
excerpt: "Meta Programming 1 - Proxy"
categories:
  - Javascript
tags:
  - f-lab
  - 에프랩
  - TIL
  - Javascript
  - Meta Programming
  - Proxy
last_modified_at: 2023-06-09
toc: true
---

어제 포스팅에서 마지막에 심볼의 심화된 용례인 proxy, reflect를 알아보겠다고 했지만, 알고보니 proxy와 reflect는 독립적인 개념이고 필요하면 심볼이 그 안에서 사용될 수 있는 것이었다. 그래서 오늘부터는 제목을 Meta Programming 으로 해서 자바스크립트에서 메타 프로그래밍적인 동작을 하는 proxy와 reflect를 알아보려고 한다. 오늘은 그 첫 날인 proxy다.

# proxy?

proxy는 특정 객체를 감싸 프로퍼티 읽기, 쓰기와 같은 객체에 가해지는 작업을 중간에서 가로채는 객체다. 가로채서 뭐 하냐고? 그 작업을 재정의 할 수 있다. 다르게 보자면, 프록시 객체(Proxy object)는 대상 객체(Target object) 대신 사용된다. 대상 객체를 직접 사용하는 대신, 프록시 객체가 사용되며 각 작업을 대상 객체로 전달하고 결과를 다시 코드로 돌려준다.

먼저 프록시를 어떻게 생성하는지 보자.

```javascript
let proxy = new Proxy(target, handler);
```

`new` 연산자를 붙혀 프록시 생성자를 호출하며 2개의 필수적인 인자를 넣어줘야 한다.

- `target`: 감싸게 될 객체로, 함수를 포함한 모든 객체가 가능.
- `handler`: 가로채는 작업과 가로채는 작업을 재정의하는 방법을 정의하는 객체. 동작을 가로채는 메서드인 '트랩(trap)'이 담긴 객체로, 여기서 프락시를 설정(예시: `get` 트랩은 `target`의 프로퍼티를 읽을 때, `set` 트랩은 `target`의 프로퍼티를 쓸 때 활성화됨).

아래의 예제를 통해 좀 더 직관적으로 이해해보자.

```javascript
let target = {};
let proxy = new Proxy(target, {}); // 빈 핸들러를 넣음

proxy.test = 5; // 프락시에 값을 씁니다. -- (1)
alert(target.test); // 5, target에 새로운 프로퍼티가 생김

alert(proxy.test); // 5, 프락시를 사용해 값을 읽을 수도 있음. -- (2)

for (let key in proxy) alert(key); // test, 반복도 잘 동작. -- (3)
```

위 예시에서는 빈 핸들러를 인자로 줬기 때문에 트랩이 없다. 트랩이 없는 경우 `proxy`에 하는 모든 작업은 `target`에 전달된다.

그렇다면 트랩을 어떻게 설정하고 그것이 어떤 동작을 하게 될까?

## 트랩

객체에 어떤 작업을 할 땐 자바스크립트 명세서에 정의된 '내부 메서드(internal method)'가 깊숙한 곳에서 관여한다. 프로퍼티를 읽을 땐 `[[Get]]`이라는 내부 메서드가, 프로퍼티에 쓸 땐 `[[Set]]`이라는 내부 메서드가 관여한다. 이런 내부 메서드들은 명세서에만 정의된 메서드이기 때문에 개발자가 코드를 사용해 호출할 순 없다.

그리고 프록시는 이 내부 메서드의 호출을 가로챈다. 그래서 모든 내부 메서드에 대응하는 트랩이 있다. 그 핸들러 메소드는 다음과 같다.

|       내부 메서드       |       핸들러 메서드        |                                           작동 시점                                           |
| :---------------------: | :------------------------: | :-------------------------------------------------------------------------------------------: |
|        `[[Get]]`        |           `get`            |                                      프로퍼티를 읽을 때                                       |
|        `[[Set]]`        |           `set`            |                                       프로퍼티에 쓸 때                                        |
|    `[[HasProperty]]`    |           `has`            |                                    `in` 연산자가 동작할 때                                    |
|      `[[Delete]]`       |      `deleteProperty`      |                                  `delete` 연산자가 동작할 때                                  |
|       `[[Call]]`        |          `apply`           |                                       함수를 호출할 때                                        |
|     `[[Construct]]`     |        `construct`         |                                    new 연산자가 동작할 때                                     |
|  `[[GetPrototypeOf]]`   |      `getPrototypeOf`      |                                     Object.getPrototypeOf                                     |
|  `[[SetPrototypeOf]]`   |      `setPrototypeOf`      |                                     Object.setPrototypeOf                                     |
|   `[[IsExtensible]]`    |       `isExtensible`       |                                      Object.isExtensible                                      |
| `[[PreventExtensions]]` |    `preventExtensions`     |                                   Object.preventExtensions                                    |
| `[[DefineOwnProperty]]` |      `defineProperty`      |                        Object.defineProperty, Object.defineProperties                         |
|  `[[GetOwnProperty]]`   | `getOwnPropertyDescriptor` |             Object.getOwnPropertyDescriptor, for..in, Object.keys/values/entries              |
|  `[[OwnPropertyKeys]]`  |         `ownKeys`          | Object.getOwnPropertyNames, Object.getOwnPropertySymbols, for..in, Object/keys/values/entries |

내부 메소드나 트랩을 쓸 때 몇 가지 규칙이 있다. 대부분 반환 값과 관련되어 있는데, 아주 이상한 짓을 하지 않는한 이 규칙을 어길 일은 거의 없을 거라고 한다... 그래도 궁금하면 [명세서](https://tc39.es/ecma262/#sec-proxy-object-internal-methods-and-internal-slots)를 보자.

### get 트랩

이제 본격적으로 트랩을 사용해 동작을 가로채는 걸 해보자. 가장 쉽고 흔하게 볼 수 있는 get 트랩을 사용해 알아보도록 하자.

프로퍼티 읽기를 가로채려면 `handler`에 `get(target, property, receiver)` 메소드가 있어야 한다. `get` 메소드는 프로퍼티를 읽으려고 할 때 작동한다.

- `target`: 동작을 전달할 객체로 new Proxy의 첫 번째 인자.
- `property`: 프로퍼티 이름
- `receiver`: 타깃 프로퍼티가 getter라면 `receiver`는 getter가 호출될 때 this 이다. 대개는 proxy 객체 자신이 this가 된다. 프락시 객체를 상속받은 객체가 있다면 해당 객체가 this가 되기도 한다. 지금 당장은 이 인수가 필요 없으므로 더 자세한 내용은 나중에 다루자.

get 트랩을 사용한 예제를 보자. 이 예제는 존재하지 않는 요소를 읽으려고 할 때 기본값인 0을 반환해주는 배열을 만든다(원래 배열은 존재하지 않는 요소를 읽으려고 하면 `undefined`를 반환한다).

```javascript
let numbers = [1, 2, 3];

numbers = new Proxy(numbers, {
  get(target, prop) {
    if (prop in target) {
      return target[prop];
    } else {
      return 0; // 기본값
    }
  },
});

console.log(numbers[1]); // 2
console.log(numbers[123]); // 0
```

여기서 한 가지 짚고 넘어가야 할 점이 있다. 예제를 보면 목표가 되는 타겟 객체가 프록시 객체로 덮어써지는 것을 알 수 있다. 이렇게 하는 이유는 프록시로 감싸진 이후 원본 타겟 객체를 참조하지 못하게 하기 위해서다. 만약 이후 원본 타겟 객체를 참조하게 되면 엉망이 될 확률이 아주 높아진다.

### set 트랩

이번엔 값을 쓰려고 할 때 이를 가로채는 set 트랩에 대해 알아보자.

`set(target, property, value, receiver)` 메소드의 인수는 아래와 같은 역할을 한다.

- `target`: 동작을 전달할 객체로 new Proxy의 첫 번째 인자.
- `property`: 프로퍼티 이름
- `value`: 프로퍼티 값
- `receiver`: get 트랩과 유사하게 동작하는 객체로, setter 프로퍼티에만 관여.

바로 예제를 볼텐데, 숫자형 값을 설정하려 할 때만 `true`를, 그렇지 않으면 `false`를 반환하도록 만들어 볼 것이다.

```javascript
let numbers = [];

numbers = new Proxy(numbers, {
  set(target, prop, val) {
    if (typeof val == "number") {
      target[prop] = val;
      return true;
    } else {
      return false;
    }
  },
});

numbers.push(1); // 추가 성공
numbers.push(2); // 추가 성공

numbers.push("test"); // Uncaught TypeError: 'set' on proxy: trap returned falsish for property '2'
```

문자열을 넣으려고 하면 타입 에러와 함께 이유와 위치도 친절하게 알려준다.

그리고 주목해야 할 점은 배열 메소드를 여전히 사용할 수 있다는 것이다. 프록시를 사용해도 기존에 있던 기능은 똑같이 동작한다.

`push`나 `unshift` 같이 배열에 값을 추가해주는 메서드들은 내부에서 `[[Set]]`을 사용하고 있기 때문에 메서드를 오버라이드 하지 않아도 프락시가 동작을 가로채고 값을 검증해준다.

오늘은 프록시가 무엇인지와 가장 기초적인 트랩 두 가지에 대해 알아보았다. 내일은 리플렉트에 대해 알아보도록 하자!

\*참고: <https://ko.javascript.info/proxy>,
<https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Proxy>,
<https://yceffort.kr/2021/03/javascript-proxy>,
<https://ui.toast.com/posts/ko_20210413>
