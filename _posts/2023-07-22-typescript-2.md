---
title: "[F-Lab 모각코 챌린지] 64일차"
excerpt: "타입스크립트"
categories:
  - Typescript
tags:
  - f-lab
  - 에프랩
  - TIL
  - Programming Terms
last_modified_at: 2023-07-22
toc: true
---

타입스크립트 공부를 너무 띄엄띄엄 했다. 파이브 라인스 오브 코드만 쫓아가지도 쉽지 않긴 한데, 타입스크립트 공부도 루틴화 하자.

# 튜플

타입스크립트에는 바닐라 자바스크립트에는 없던 새로운 개념과 타입이 있다. 그 중 하나가 튜플이다.

튜플은 배열과 비슷하지만 길이와 타입이 고정된다. 이것이 필요한 예제를 살펴보자. `person` 에는 role이 필요하고, 이것은 role number와 description만으로 표현돼야 한다고 전제해보자. 즉, 배열의 첫 번째 요소는 숫자, 두 번째 요소는 문자열이어야 한다.

먼저 아래와 같이 작성하고 타입스크립트의 추론을 알아보자.

```typescript
const person = {
  name: "찬주",
  age: 32,
  hobbies: ["eat", "shit"],
  role: [1, "guest"],
};
```

`role`에 커서를 올리고 기다려보면 `(property) role: (string | number)[]` 라고 나온다. `(string | number)[]`는 union 타입인데, 이건 나중에 다시 알아보자.

어쨌든 이 의미는 string 타입이나 number 타입을 요소로 갖고 있는 배열이어야 한다는 뜻이다.

이 상태에서 우리는 `person.role.push()` 혹은 인덱스에 직접 접근해 어떤 값이든 추가하거나 빼거나 할 수 있다. 이렇게 되면 우리가 설계한 역할이란 것이 제 기능을 할 수 없게 된다.

이제 명시적으로 `role`을 어떻게 써야 하는지 정의해보자.

```typescript
const person: {
  name: string;
  age: number;
  hobbies: string[];
  role: [number, string];
} = {
  name: "찬주",
  age: 32,
  hobbies: ["eat", "shit"],
  role: [1, "guest"],
};
```

`person` 객체의 프로퍼티 각각에 타입을 지정했다. 이제

`role`의 타입을 보자. 대괄호 안에 순서대로 타입을 넣었다. 첫 번째로 숫자가 와야하고, 두 번째로 문자열이 와야한다. 끝. 타입과 길이까지 고정한 것이다.

그 뒤로 `person.role[1] = '10'` 과 같은 코드를 입력하면 IDE에서 바로 에러를 알려준다. 컴파일 시 에러가 난다는 것이다.

아까 언급한 배열 메서드인 `push()` 어떨까? 아쉽게도 이것을 막아주지는 못한다.

튜플을 사용하면 설계나 기획상에서 정해진 타입을 인간적인 실수로 어기는 일을 줄일 수 있다.
