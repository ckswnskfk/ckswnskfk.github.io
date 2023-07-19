---
title: "[F-Lab 모각코 챌린지] 61일차"
excerpt: "five lines of code - 12"
categories:
  - Refactoring
tags:
  - f-lab
  - 에프랩
  - TIL
  - Typescript
  - Five lines of code
last_modified_at: 2023-07-19
toc: true
---

# 유사한 코드 융합하기

## 클래스 간의 코드 통합

### 규칙: 구현체가 하나뿐인 인터페이스를 만들지 말 것

#### 정의

구현체가 하나뿐인 인터페이스를 사용하지 마라.

#### 설명

말 그대로 단일 구현체를 가진 인터페이스가 있으면 안된다는 규칙이다. '항상 인터페이스로 코딩하라' 라는 말도 있지만 그것이 정말 항상 유익하지만은 않다.

먼저 구현 클래스가 하나밖에 없는 인터페이스는 가독성에 도움이 되지 않는다. 더 나쁜 점은 인터페이스는 변형을 전제로 하는데, 아무것도 없다면 오히려 가독성을 방해할 수 있다는 점이다. 그리고 반대로 구현 클래스를 수정하려면 인터페이스를 수정해야 해서 오히려 오버헤드가 발생할 수 있다.

인터페이스를 독립된 파일에 작성하는 경우 구현 클래스가 하나밖에 없다면 두 개의 파일을 사용하게 되는 반면, 인터페이스 없이 구현 클래스만 있는 경우는 하나의 파일만 사용한다.

한 편 아무런 구현체가 없는 인터페이스를 사용하는 경우도 있다.

#### 스멜

'컴퓨터 과학의 모든 문제는 간접 레이어를 도입함으로써 해결할 수 있다'는 말이 있듯이, 인터페이스는 세부적인 내용을 추상화 아래에 숨김으로써 많은 문제를 해결한다.

추상화는 인지의 복잡성의 감소를 위해 실제의 복잡성의 증가를 허용하는 것이다. 이 말은 추상화에 신중해야 한다는 것을 암시한다.

#### 의도

불필요한 코드의 상용구를 제한하는 것이다. 인터페이스는 상용구의 일반적인 원인이다. 무분별한 인터페이스의 사용은 애플리케이션의 크기를 부풀리는 경향이 있다.

### 리팩터링 패턴: 구현에서 인터페이스 추출

#### 설명

인터페이스 만드는 것을 그것이 필요할 때(변형을 도입하고 싶을 때)까지 연기할 수 있다.

#### 절차

1. 추출할 클래스와 동일한 이름으로 새로운 인터페이스를 만든다.
2. 인터페이스를 추출할 클래스의 이름을 변경하고 새로운 인터페이스를 구현하게 한다.
3. 컴파일하고 오류를 검토한다.
   - `new` 때문에 오류가 발생하면 인스턴스화하는 부분을 새로운 클래스의 이름으로 변경한다.
   - 그렇지 않으면 오류를 일으키는 메서드를 인터페이스에 추가한다.

#### 예제

어제 사용한 `SumProcessor` 예제를 다시 사용한다.

```typescript
// 초기 코드
class ArraySum {
  private processor: SumProcessor;

  constructor(accumulator: number) {
    this.processor = new SumProcessor(accumulator);
  }

  process(arr: number[]) {
    for (let i = 0; i < arr.length; i++) {
      this.processor.processElement(arr[i]);
    }
    return this.processor.getAccumulator();
  }
}

class SumProcessor {
  constructor(private accumulator: number) {}

  getAccumulator() {
    return this.accumulator;
  }

  processElement(e: number) {
    this.accumulator += e;
  }
}
```

절차를 따라가 보자.

1. 추출할 클래스와 동일한 이름으로 새 인터페이스를 만든다.

```typescript
interface SumProcessor {}
```

2. 인터페이스를 추출하려는 클래스의 이름을 바꾸고 새 인터페이스를 구현하게 한다.

```typescript
class TmpName implements SumProcessor {
  constructor(private accumulator: number) {}

  getAccumulator() {
    return this.accumulator;
  }

  processElement(e: number) {
    this.accumulator += e;
  }
}
```

3. 컴파일 및 오류 검토
   - `new` 로 인한 오류인 경우 인스턴스화 하는 부분을 새 클래스 이름으로 변경한다.

```typescript
class ArraySum {
  private processor: SumProcessor;

  constructor(accumulator: number) {
    this.processor = new TmpName(accumulator); // 인터페이스 대신 클래스를 인스턴스 화
  }
  // ...
}
```

    - 그렇지 않으면 오류를 일으키는 메서드를 인터페이스에 추가.

```typescript
interface SumProcessor {
  processElement(e: number): void;
  getAccumulator(): number;
}
```

이제 잘 동작한다. 인터페이스 이름을 `ElementProcessor` 같은 더 적합한 것으로 변경하고 클래스 이름을 다시 `SumProcessor`로 변경한다.

```typescript
class ArraySum {
  private processor: SumProcessor;

  constructor(accumulator: number) {
    this.processor = new SumProcessor(accumulator);
  }

  process(arr: number[]) {
    for (let i = 0; i < arr.length; i++) {
      this.processor.processElement(arr[i]);
    }
    return this.processor.getAccumulator();
  }
}

class SumProcessor implements ElementProcessor {
  constructor(private accumulator: number) {}

  getAccumulator() {
    return this.accumulator;
  }

  processElement(e: number) {
    this.accumulator += e;
  }
}

interface ElementProcessor {
  processElement(e: number): void;
  getAccumulator(): number;
}
```

이전의 `MinumunProcessor`가 인터페이스를 구현하도록 하고, `ArraySum`의 `accumulator` 매개 변수를 `processor`로 바꾸고, 이름을 `BatchProcessor`로 바꾼다.

```typescript
class BatchProcessor {
  constructor(processor: ElementProcessor) {}
  process(arr: number[]) {
    for (let i = 0; i < arr.length; i++) {
      this.processor.processElement(arr[i]);
    }
    return this.processor.getAccumulator();
  }
}

interface ElementProcessor {
  processElement(e: number): void;
  getAccumulator(): number;
}

class MinimumProcessor implements ElementProcessor {
  constructor(private accumulator: number) {}
  getAccumulator() {
    return this.accumulator;
  }
  processElement(e: number) {
    if (this.accumulator > e) {
      this.accumulator = e;
    }
  }
}

class SumProcessor implements ElementProcessor {
  constructor(private accumulator: number) {}
  getAccumulator() {
    return this.accumulator;
  }
  processElement(e: number) {
    this.accumulator += e;
  }
}
```
