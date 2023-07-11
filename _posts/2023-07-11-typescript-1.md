---
title: "[F-Lab 모각코 챌린지] 53일차"
excerpt: "typescript-1"
categories:
  - Typescript
tags:
  - f-lab
  - 에프랩
  - TIL
  - Typescript
last_modified_at: 2023-07-11
toc: true
---

파이브라인스 오브 코드와 병행하는 타입스크립트 공부. 나름대로 정리하며 공부하자.

# 객체 형태

여기에 자바스크립트 객체가 하나 있다.

```javascript
const person = {
  name: "찬주",
  age: 32,
};
```

타입스크립트 파일로 만들었다면 IDE가 VSCode라는 전제하에, 객체 명이나 프로퍼티명에 마우스를 올리면 타입을 알려준다. 이것은 우리가 할당한 값에서 추론한 것으로, 당연히 명시적으로 타입을 지정할 수도 있다.

```typescript
const person: {
  name: string;
  age: number;
} = {
  name: "찬주",
  age: 32,
};
```

객체명 옆에 콜론으로 표시된 부분은 새로운 객체를 만든 것이 아니며, 해당 객체의 프로퍼티의 타입을 지정한다.

만약 중첩된 객체가 있다면 어떻게 쓸까?

```typescript
const product: {
  id: string;
  price: number;
  tags: string[];
  details: {
    title: string;
    description: string;
  };
} = {
  id: "abc1",
  price: 12.99,
  tags: ["great-offer", "hot-and-new"],
  details: {
    title: "Red Carpet",
    description: "A great carpet - almost brand-new!",
  },
};
```

# 배열

```typescript
const person = {
  name: "찬주",
  age: 32,
  hobbies: ["음악 감상", "낮잠"],
};
```

마찬가지로 hobbies에 마우스를 올려보면 `(property) hobbies: string[]` 이렇게 나올 것이다. 이것 또한 타입스크립트가 추론한 타입이다.

```typescript
let favoriteActivities: string[];

favoriteActivities = ["음악 감상", 3];
```

이렇게 타입을 명시한 경우, 다른 타입의 데이터가 들어오게 되면 오류가 나게 된다. 만약 여러 타입을 담고자 한다면 다음과 같이 작성해야한다.

```typescript
let favoriteActivities: any[];
```

`any`타입은 나중에 더 알아볼 거지만 자주 사용하면 좋지 않다. 타입스크립트의 장점을 무시하게 된다.

```typescript
const person = {
  name: "찬주",
  age: 32,
  hobbies: ["음악 감상", "낮잠"],
};

for (const hobby of person.hobbies) {
  console.log(hobby.toUpperCase());
}
```

타입 추론의 강력함을 알아보자. 위의 예제 코드는 전혀 문제 없이 동작한다. for문 내에서 `hobby`의 메서드를 배열 메서드로 바꾸면 어떻게 될까?

```typescript
const person = {
  name: "찬주",
  age: 32,
  hobbies: ["음악 감상", "낮잠"],
};

for (const hobby of person.hobbies) {
  console.log(hobby.map()); // error
}
```

오류가 나게 된다. 할당된 값이 string이고 이에 따라 타입스크립트는 `person.hobbies`의 타입은 `string[]`라고 추론하기 때문이다. 이는 루프문 안에서의 각 요소에도 적용이 된다.
