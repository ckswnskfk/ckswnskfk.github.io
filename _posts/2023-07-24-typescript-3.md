---
title: "[F-Lab 모각코 챌린지] 66일차"
excerpt: "타입스크립트"
categories:
  - Typescript
tags:
  - f-lab
  - 에프랩
  - TIL
  - ENUM
last_modified_at: 2023-07-24
toc: true
---

# enum

자바스크립트에는 없던 타입이 또 있다. 바로 enum이다. enum은 열거형으로 이름이 있는 상수들의 집합을 정의할 수 있다.

enum을 선언하는 방식은 이렇다.

```typescript
enum {
  NEW , OLD
}
```

선언된 레이블들은 0부터 시작하는 정수로 변환된다.

애플리케이션에서 사용하는 식별자를 관리할 때 enum을 사용하기 좋다.

```typescript
enum Role {
  ADMIN,
  READ_ONLY,
  AUTHOR,
}

const person = {
  name: "찬주",
  age: 32,
  hobbies: ["eat", "shit"],
  role: Role.ADMIN,
};

if (person.role === Role.ADMIN) {
  console.log("관리자입니다.");
}
```

선언된 enum에 마우스 커서를 올려보면 순서대로 0, 1, 2 가 할당된 것 처럼 표시 된다.

원한다면 초기값을 새롭게 지정할 수도 있다.

```typescript
enum Role {
  ADMIN = 5,
  READ_ONLY,
  AUTHOR,
}
```

이 경우 5, 6, 7이 된다. 각각 고유한 값을 할당할 수도 있다.

```typescript
enum Role {
  ADMIN = 10,
  READ_ONLY = 0,
  AUTHOR = 15,
}
```

문자열도 되고 섞어도 된다.

```typescript
enum Role {
  ADMIN = "ADMIN",
  READ_ONLY = 0,
  AUTHOR = "아무거나",
}
```

개발자가 읽기에도 좋고 백그라운드에서 매핑된 값이 있는 식별자가 필요할 때 훌륭한 선택지이다.

# any

타입스크립트의 타입중 가장 유연하다. 아무 타입이나 가능하다는 의미로, 아무것도 강제하지 않는다. 물론 작동에도 아무런 문제가 없다.

하지만 정말 특별한 경우가 아니면 쓰지 않아야한다. any를 남발하면 타입스크립트를 쓸 이유가 없다.

따라서 어떤 값이나 종류의 데이터가 어디에 저장될지 전혀 알 수 없는 경우에 대비할 때나, 런타임 검사를 수행하는 경우 런타임 도중 이처럼 특정 값에 수행하고자 하는 작업의 범위를 좁히기 위해서만 any를 사용하면 된다.

그 외에는 쓰면 안된다.

# 유니언 타입

타입을 좀 더 유연하게 쓰고자 할 때 `|`를 기준으로 타입을 더 나열해 허용할 타입들을 지정할 수 있다. 이것은 유니언 타입이라고 한다.

```typescript
function combine(input1: number | string, input2: number | string) {
  let result;
  if (typeof input1 === "number" && typeof input2 === "number") {
    result = input1 + input2;
  } else {
    result = input1.toString() + input2.toString();
  }

  return result;
}

const conbinedAges = combine(30, 26);

console.log(conbinedAges);

const combinedNames = combine("찬주", "영은");
console.log(combinedNames);
```

하지만 위의 예제처럼 타입에 대해 열려있다면 런타임으로 타입을 체크해야 하는 경우가 생길 수도 있다. 권장되는 방법은 아니다.
