---
title: "[F-Lab 모각코 챌린지] 55일차"
excerpt: "five lines of code - 8"
categories:
  - Refactoring
tags:
  - f-lab
  - 에프랩
  - TIL
  - Typescript
  - Five lines of code
last_modified_at: 2023-07-13
toc: true
---

오늘도 규칙과 의도를 알아보자.

# 타입 코드 처리하기

## 필요 없는 코드 제거하기

많은 IDE는 사용되지 않는 함수를 표시해준다. 이런 표시가 있으면 즉시 함수를 삭제하자. 그것이 시간을 아끼는 방법이다.

아쉽지만 인터페이스의 메서드가 사용되는지 여부를 알려주는 IDE는 없다.

하지만 컴파일러를 이용해 간단히 사용 여부를 알아볼 수 있다.

1. 컴파일. 오류가 없어야 함
2. 인터페이스에서 메서드 삭제
3. 컴파일
   - 오류가 발생하면 실행을 취소하고 계속 진행
   - 그렇지 않으면 각 클래스를 살펴보고 오류 없이 해당 메서드를 삭제할 수 있는지 확인

이것을 리팩터링 패턴화 해보자.

### 리팩터링 패턴: 삭제 후 컴파일

#### 설명

이 리팩터링 패턴은 인터페이스의 범위를 알고 있을 때, 인터페이스에서 사용하지 않는 메서드를 제거하는 용도로 사용할 수 있다. 이름처럼 간단한 절차를 따른다. 하지만 아직 사용하지 않는 메서드를 삭제할 수 있기 때문에 새로운 기능을 구현하는 동안에는 이 리팩터링을 수행하면 안된다.

#### 절차

위에 적혀있는 것과 같다.

#### 예제

이 예제에는 사용되지 않는 메서드가 세개 있지만 IDE에서는 제대로 표시되지 않는다.

```typescript
interface A {
  m1(): void;
  m2(): void;
}

class B implements A {
  m1() {
    console.log("m1");
  }
  m2() {
    this.m3();
  }
  m3() {
    console.log("m3");
  }
}

let a = new B();
a.m1();
```

```typescript
interface A {
  m1(): void;
}

class B implements A {
  m1() {
    console.log("m1");
  }
}

let a = new B();
a.m1();
```
