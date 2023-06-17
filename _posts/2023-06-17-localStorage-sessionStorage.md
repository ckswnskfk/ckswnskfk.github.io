---
title: "[F-Lab 모각코 챌린지] 29일차"
excerpt: "localStorage와 sessionStorage"
categories:
  - Javascript
tags:
  - f-lab
  - 에프랩
  - TIL
  - Javascript
  - Browser
  - localStorage
  - sessionStorage
last_modified_at: 2023-06-17
toc: true
---

오늘은 어제의 쿠키에 이어 브라우저에 데이터를 저장할 수 있는 localStorage와 sessionStorage에 대해 알아보려한다.

# localStorage와 sessionStorage

이 둘은 쿠키처럼 브라우저 내에 key-value 쌍을 저장할 수 있게 해준다. 이 둘을 사용하면 페이지를 새로 고침하거나(sessionStorage) 심지어 브라우저를 다시 실행해도(localStorage) 데이터가 사라지지 않고 남아있다. 키와 값은 항상 각 문자에 2바이트를 할당하는 UTF-16 DOMString의 형태로 저장된다. 객체와 마찬가지로 정수 키는 자동으로 문자열로 변환한다.

## 쿠키가 있는데 또 왜?

쿠키와는 구분되는, 이 두 방식을 사용하는 이유는 다음과 같다.

- 쿠키와 다르게 웹 스토리지 객체는 네트워크 요청 시 서버로 전송되지 않는다. 이런 특징 때문에 쿠키보다 더 많은 자료를 보관할 수 있다. 대부분의 브라우저가 최소 2MB 혹은 그 이상의 웹 스토리지 객체를 저장할 수 있도록 해준다. 또한 개발자는 브라우저 내 웹 스토리지 구성 방식을 설정할 수 있다.
- 쿠키와 또 다른 점은 서버가 HTTP 헤더를 통해 스토리지 객체를 조작할 수 없다는 것이다. 웹 스토리지 객체 조작은 모두 자바스크립트 내에서 수행된다.
- 웹 스토리지 객체는 오리진(origin, 출처)에 묶여있다. 따라서 프로토콜과 서브 도메인이 다르면 데이터에 접근할 수 없다.

\*origin, 즉 출처란 URL 구조에서 protocol, host(도메인), port를 합친 것을 말한다.

## 스토리지 객체

이 두 스토리지 객체는 동일한 메서드와 프로퍼티를 제공한다.

- `setItem(key, value)` – 키-값 쌍을 보관한다.
- `getItem(key)` – 키에 해당하는 값을 받아온다.
- `removeItem(key)` – 키와 해당 값을 삭제한다.
- `clear()` – 모든 것을 삭제한다.
- `key(index)` – 인덱스(index)에 해당하는 키를 받아온다.
- `length` – 저장된 항목의 개수를 얻는다.

`Map` 과 비슷하지만 인덱스로 키에 접근할 수 있다는 점이 다르다.

이제 각각을 자세히 알아보자.

## localStorage

localStorage의 주요 기능은 다음과 같다.

- 오리진이 같은 경우, 데이터는 모든 탭과 창에서 공유된다.
- 브라우저나 OS가 재시작하더라도 데이터가 파기되지 않는다.

syntax는 단순하다.

```javascript
localStorage.setItem("test", 1);
```

이후 `localStorage.getItem('test')` 를 통해 값을 얻을 수 있다. 브라우저를 종료 했다가 다시 켜던지 다른 창에서 이 페이지를 다시 열었어도 결과를 얻을 수 있다. localStorage는 동일한 오리진을 가진 모든 창에서 공유된다. 따라서 한 창에 데이터를 설정하면 다른 창에서 변동 사항을 볼 수 있다. (사생활 보호 모드 중 생성한 localStorage 데이터는 마지막 사생활 보호 탭이 닫힐 때 지워진다.)

### 사용하기

#### 일반 객체처럼

호환성 때문에 권장되지 않지만, 일반 객체와 유사하게 사용할 수 있다.

```javascript
// 키 설정하기
localStorage.test = 2;

// 키 얻기
alert(localStorage.test); // 2

// 키 삭제하기
delete localStorage.test;
```

#### 순회

localStorage는 이터러블 객체는 아니지만, `key(index)`메서드를 이용해 배열처럼 순회할 수 있다.

```javascript
for (let i = 0; i < localStorage.length; i++) {
  let key = localStorage.key(i);
  alert(`${key}: ${localStorage.getItem(key)}`);
}
```

`for...in` 문도 사용할 수 있긴 하지만, 프로토타입에서 상속 받은 프로퍼티까지 순회하기 때문에 `hasOwnProperty`를 이용해 순회해야한다.

#### 문자열

localStorage의 키와 값은 반드시 문자열이어야 한다. 그래서 숫자나 객체 등 다른 자료형을 사용하게 되면 문자열로 자동 변환된다.

```javascript
localStorage.user = { name: "John" };
alert(localStorage.user); // [object Object]
```

## sessionStorage

sessionStorage 객체는 localStorage에 비해 자주 사용되진 않는다. 제공하는 프로퍼티와 메서드는 같지만, 훨씬 제한적이기 때문이다.

- 같은 페이지라도 다른 탭에 있으면 다른 곳에 저장되기에 sessionStorage는 현재 떠 있는 탭 내에서만 유지된다.
- 하지만 하나의 탭에 여러 개의 iframe이 있는 경우엔 동일한 오리진에서 왔다고 취급되기 때문에 sessionStorage가 공유된다.
- 페이지를 새로 고침할 때 sessionStorage에 저장된 데이터는 사라지지 않는다. 하지만 탭을 닫고 새로 열 때는 사라진다.
- 페이지를 새로운 탭이나 창에서 열면, 세션 쿠키의 동작과는 다르게 최상위 브라우징 맥락의 값을 가진 새로운 세션을 생성한다.

## storage 이벤트

localStorage나 sessionStorage의 데이터가 갱신될 때, storage 이벤트가 실행된다. storage 이벤트는 다음과 같은 프로퍼티를 지원한다.

- key – 변경된 데이터의 키(.clear()를 호출했다면 null)
- oldValue – 이전 값(키가 새롭게 추가되었다면 null)
- newValue – 새로운 값(키가 삭제되었다면 null)
- url – 갱신이 일어난 문서의 url
- storageArea – 갱신이 일어난 localStorage나 sessionStorage 객체

중요한 점은, storage 이벤트가 이벤트를 발생시킨 스토리지를 제외하고 스토리지에서 접근 가능한 window 객체 전부에서 일어난다는 것이다. 예를 들어, 두 개의 창에 같은 사이트를 띄워놨다고 가정해보자. 창은 다르지만 localStorage는 서로 공유된다. 두 창에서 모두 storage 이벤트를 수신하고 있기 때문에 한 창에서 데이터를 갱신하면 다른 창에 해당 사항이 반영된다.

storage 이벤트의 또 다른 중요한 특징은 event.url이 있어 데이터가 갱신된 문서의 URL을 알 수 있다는 점이다.

또한 event.storageArea에는 스토리지 객체가 포함되어 있는데, storage 이벤트는 sessionStorage나 localStorage가 변경될 때 모두 발생하기 때문에 event.storageArea는 스토리지 종류에 상관없이 실제 수정이 일어난 것을 참조한다는 것 역시 중요한 특징이다. 변경이 일어났을 때 우리는 event.storageArea에 무언가를 설정해 '응답’이 가능하도록 할 수 있다. 이런 특징을 이용하면 오리진이 같은 창끼리 메시지를 교환하게 할 수 있다.

## localStorage 또는 sessionStorage를 활용한 인증, 인가

### 인증과 인가?

인증(Authentication)은 사용자의 신원을 확인하고 확인된 사용자에게 접근 권한을 부여하는 과정을 말한다. 인가(Authorization)는 인증된 사용자가 특정 리소스에 접근할 수 있는 권한을 가지고 있는지 여부를 결정하는 과정이다.

### 구현 방식

웹 스토리지를 활용한 인증과 인가의 구현 방식은 다양한데, 일반적으로는 다음과 같은 접근 방법을 사용한다.

1. 로그인 후 토큰 저장: 사용자가 로그인하면 서버에서 발급한 인증 토큰을 받아 웹 스토리지인 localStorage 또는 sessionStorage에 저장한다. 이 토큰은 사용자의 인증 상태를 나타내는 역할을 하며, 이후에 서버로 보내는 요청에 토큰을 함께 전송하여 인증을 유지한다.

2. 토큰 유효성 검사: 클라이언트는 요청을 보낼 때마다 토큰을 함께 전송하여 서버에 인증을 요청한다. 서버는 받은 토큰의 유효성을 검사하고, 유효한 토큰일 경우 해당 요청에 대한 인가를 수행한다.

3. 토큰 갱신: 토큰은 유효기간을 가지고 있으며, 만료되면 다시 로그인해야 한다. 그러나 로그인 과정은 번거롭기 때문에, 토큰을 갱신하는 방법을 사용한다. 클라이언트는 만료된 토큰을 서버에 보내어 새로운 토큰을 발급받고, 이를 다시 웹 스토리지에 저장하여 인증을 유지한다.

4. 로그아웃 및 토큰 삭제: 사용자가 로그아웃하거나 토큰을 더 이상 사용하지 않을 경우, 웹 스토리지에서 토큰을 삭제하여 인증을 종료한다.

이와 같은 방식으로 웹 스토리지를 활용하여 인증과 인가를 구현할 수 있다. 내일은 흔히 비교되는 쿠키나 세션, 토큰등을 이용한 인증과 인가에 대해 알아보자.

\*참고: <https://developer.mozilla.org/ko/docs/Web/API/Web_Storage_API/Using_the_Web_Storage_API>,  
<https://ko.javascript.info/localstorage>,  
<https://chat.openai.com/>
