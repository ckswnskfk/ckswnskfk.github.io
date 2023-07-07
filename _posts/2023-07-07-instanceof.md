---
title: "[F-Lab 모각코 챌린지] 49일차"
excerpt: "instanceof"
categories:
  - Javascript
tags:
  - f-lab
  - 에프랩
  - TIL
  - Javascript
  - Class
  - instanceof
last_modified_at: 2023-07-07
toc: true
---

잠시 쉬어가는 차원에서 javascript의 연산자 하나를 살펴보고자 한다.

# `instanceof`로 클래스 확인하기

`instanceof` 연산자는 대상 객체가 특정 클래스에 속하는지 아닌지 상속 관계도 포함하여 확인할 수 있게 해준다.

## instanceof

```javascript
obj instanceof Class;
```

아래의 예제를 보자.

```javascript
class Rabbit {}
let rabbit = new Rabbit();

// rabbit이 클래스 Rabbit의 객체인가요?
alert(rabbit instanceof Rabbit); // true
```

Class 문법과 내부 동작은 거의 같은 생성자 함수도 마찬가지다.

```javascript
// 클래스가 아닌 생성자 함수
function Rabbit() {}

alert(new Rabbit() instanceof Rabbit); // true

// Array 같은 내장 클래스도 가능
let arr = [1, 2, 3];
alert(arr instanceof Array); // true
alert(arr instanceof Object); // true
```

자바스크립트의 상속은 프로토타입 기반으로 이루어진다. `instanceof` 연산자는 프로토타입 체인을 거슬러 올라가며 인스턴스 여부를 확인한다. 한 편, 정적 메서드인 `Symbol.hasInstance`를 사용하면 직접 확인 로직을 설정할 수도 있다.

`instanceof`는 다음과 같은 순서로 동작한다.

1. 클래스에 `Symbol.hasInstance`가 구현되어 있으면 instanceof가 실행될 때, `Class[Symbol.hasInstance](obj)`가 호출된다. 호출 결과는 `true`나 `false`이어야 한다.

```javascript
// canEat 프로퍼티가 있으면 animal이라고 판단할 수 있도록
// instanceOf의 로직을 직접 설정
class Animal {
  static [Symbol.hasInstance](obj) {
    if (obj.canEat) return true;
  }
}

let obj = { canEat: true };

alert(obj instanceof Animal); // true, Animal[Symbol.hasInstance](obj)가 호출됨
```

2. 하지만 대부분의 클래스엔 `Symbol.hasInstance`가 구현되어있지 않다. 이럴 때 `obj instanceOf Class`는 `Class.prototyp`e이 `obj` 프로토타입 체인 상의 프로토타입 중 하나와 일치하는지 확인한다.

```javascript
class Animal {}
class Rabbit extends Animal {}

let rabbit = new Rabbit();
alert(rabbit instanceof Animal); // true

// rabbit.__proto__ === Rabbit.prototype
// rabbit.__proto__.__proto__ === Animal.prototype (일치!)
```

![](https://ko.javascript.info/article/instanceof/instanceof.svg)

## `instanceof`와 `constructor.name`

객체의 타입을 확인하는 데 사용되는 또 다른 방법으로 `constructor.name`이 있다. 그렇다면 둘은 어떤 게 다를까?

`instanceof`는 위에서 살펴봤으니 `constructor.name`을 알아보자.

`constructor.name`은 객체를 생성한 생성자 함수의 이름을 나타낸다. 이 속성은 생성자 함수의 `name` 속성을 참조해 객체가 어떤 생성자 함수에 의해 생성되었는지 확인한다.

```javascript
function MyClass() {}
let obj = new MyClass();
alert(obj.constructor.name); // "MyClass"
```

결국 `instanceof`와 `constructor.name` 사이의 주요 차이점은 `instanceof`가 상속 구조에 올바르게 동작하는 반면, `constructor.name`은 상속과 무관하게 생성자 함수의 이름에 직접 접근한다는 것이다. 또한, 두 방법 모두 ES6 클래스 및 함수 생성자와 함께 사용할 수 있다. 어떤 경우에는 `instanceof`가 더 견고하고 안전한 방법이 될 수 있다.
