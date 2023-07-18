---
title: "[F-Lab 모각코 챌린지] 60일차"
excerpt: "five lines of code - 11"
categories:
  - Refactoring
tags:
  - f-lab
  - 에프랩
  - TIL
  - Typescript
  - Five lines of code
last_modified_at: 2023-07-18
toc: true
---

# 유사한 코드 융합하기

## 클래스 간의 코드 통합

### 리팩터링 패턴: 전략 패턴의 도입

#### 설명

다른 클래스를 인스턴스화 해서 변형을 도입하는 개념을 전략 패턴이라고 한다.

많은 패턴이 전략 패턴의 다른 형태이다. 전략이 필드를 가지고 있는 경우, 이를 상태 패턴이라고 한다. 어쨌든 기본적인 아이디어는 동일한다. 클래스를 추가해서 변경이 가능하게 하는 것이다. 코드를 자체 클래스로 옮기는 것이다. 그럴 경우 새로운 변형을 추가하지 않더라도 확장성을 가질 수 있다.

이것은 타입을 가리키는 코드를 클래스로 변환하는 것과 다르다. 전략 클래스가 완료된 후 메서드를 추가하는 경우는 거의 없다. 기능을 변경해야 하는 경우 새 클래스를 만드는 것이 선호된다.

전략 패턴의 변형은 늦은 바인딩의 궁극적인 형태이다. 런타임에 전략 패턴을 사용하면 코드에 사용자 정의 클래스를 적재하고 이를 제어 흐름에 원활하게 통합할 수 있다.

두 가지 상황에서 전략 패턴을 도입한다.

1. 코드에 변형을 도입하고 싶어서 리팩터링을 수행하는 경우
   - 이 경우 결국 인터페이스가 있어야 한다. 그러나 빠른 리팩터링을 위해 인터페이스를 나중에 만드는 것이 좋다.
2. 클래스 간의 동작을 통합하려는 경우

#### 절차

1. 분리하려는 코드에 대해 메서드 추출을 수행한다. 다른 것과 통합하려면 메서드가 동일한지 확인한다.

2. 새 클래스를 만든다.

3. 생성자에서 새로운 클래스를 인스턴스화 한다.

4. 메서드를 새로운 클래스로 옮긴다.

5. 필드에 종속성이 있을 경우 다음을 수행한다.

   - 필드를 새로운 클래스로 옮기고 옮긴 필드에 대한 접근자를 만든다.
   - 새로운 접근자를 사용해 원래 클래스에서 발생하는 오류를 바로잡는다.

6. 새 클래스의 나머지 오류에 대해 해당 값을 대체할 매개변수를 추가한다.

7. 매서드의 인라인화를 사용해 1단계의 추출을 반대로 한다.

#### 예제

배열을 일괄 처리할 수 있는 두 개의 클래스가 있다. 더 큰 배열을 작은 배열로 나누어 계속 전달한다. 최솟값을 찾는 일괄 처리 프로세서와 합을 계산하는 일괄 처리 프로세서가 있다.

```typescript
class ArrayMinimum {
  constructor(private accumulator: number) {}
  process(arr: number[]) {
    for (let i = 0; i < arr.length; i++) {
      if (this.accumulator > arr[i]) {
        this.accumulator = arr[i];
      }
    }
    return this.accumulator;
  }
}

class ArraySum {
  constructor(private accumulator: number) {}
  process(arr: number[]) {
    for (let i = 0; i < arr.length; i++) {
      this.accumulator += arr[i];
    }
    return this.accumulator;
  }
}
```

비슷하지만 동일하진 않다. 나중에 클래스를 통합할 준비가 되도록 두 가지 모두에서 동시에 전략을 추출하는 방법을 보자.

1. 분리하려는 코드에 대해 메서드 추출을 수행. 두 클래스를 통합해야 하기 때문에 동일한지 확인한다.

```typescript
class ArrayMinimum {
  constructor(private accumulator: number) {}
  process(arr: number[]) {
    for (let i = 0; i < arr.length; i++) {
      this.processElement(arr[i]);
    }
    return this.accumulator;
  }

  processElement(e: number) {
    if (this.accumulator > e) {
      this.accumulator = e;
    }
  }
}

class ArraySum {
  constructor(private accumulator: number) {}
  process(arr: number[]) {
    for (let i = 0; i < arr.length; i++) {
      this.processElement(arr[i]);
    }
    return this.accumulator;
  }

  processElement(e: number) {
    this.accumulator += e;
  }
}
```

2. 새 클래스를 만든다

```typescript
class MinimumProcessor {}
class SumProcessor {}
```

3. 생성자에서 새 클래스를 초기화한다.

```typescript
class ArrayMinimum {
  private processor: MinimumProcessor;

  constructor(private accumulator: number) {
    this.processor = new MinimumProcessor();
  }
  // ...
}

class ArraySum {
  private processor: SumProcessor;

  constructor(private accumulator: number) {
    this.processor = new SumProcessor();
  }
  // ...
}
```

4. 메서드를 각각 `MinimumProcessor`와 `SumProcessor`로 옮긴다.

```typescript
class ArrayMinimum {
  private processor: MinimumProcessor;
  constructor(private accumulator: number) {
    this.processor = new MinimumProcessor();
  }
  process(arr: number[]) {
    for (let i = 0; i < arr.length; i++) {
      this.processElement(arr[i]);
    }
    return this.accumulator;
  }

  processElement(e: number) {
    this.processor.processElement(e);
  }
}

class ArraySum {
  private processor: SumProcessor;
  constructor(private accumulator: number) {
    this.processor = new SumProcessor();
  }
  process(arr: number[]) {
    for (let i = 0; i < arr.length; i++) {
      this.processElement(arr[i]);
    }
    return this.accumulator;
  }

  processElement(e: number) {
    this.processor.processElement(e);
  }
}

class MinimumProcessor {
  processElement(e: number) {
    if (this.accumulator > e) {
      this.accumulator = e;
    }
  }
}
class SumProcessor {
  processElement(e: number) {
    this.accumulator += e;
  }
}
```

5. 둘 모두 `accumulator` 필드에 의존하므로 `accumulator`필드를 각각 `MinimumProcessor`와 `SumProcessor`클래스로 옮기고 이에 대한 접근자를 만든다.

```typescript
class ArrayMinimum {
  private processor: MinimumProcessor;
  constructor(accumulator: number) {
    this.processor = new MinimumProcessor(accumulator);
  }
  process(arr: number[]) {
    for (let i = 0; i < arr.length; i++) {
      this.processElement(arr[i]);
    }
    return this.accumulator;
  }

  processElement(e: number) {
    this.processor.processElement(e);
  }
}

class ArraySum {
  private processor: SumProcessor;
  constructor(accumulator: number) {
    this.processor = new SumProcessor(accumulator);
  }
  process(arr: number[]) {
    for (let i = 0; i < arr.length; i++) {
      this.processElement(arr[i]);
    }
    return this.accumulator;
  }

  processElement(e: number) {
    this.processor.processElement(e);
  }
}

class MinimumProcessor {
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

- 새로운 접근자를 사용해 원래 클래스에서 발생하는 오류를 바로잡는다.

```typescript
class ArrayMinimum {
  private processor: MinimumProcessor;
  constructor(accumulator: number) {
    this.processor = new MinimumProcessor(accumulator);
  }
  process(arr: number[]) {
    for (let i = 0; i < arr.length; i++) {
      this.processElement(arr[i]);
    }
    return this.processor.getAccumulator();
  }

  processElement(e: number) {
    this.processor.processElement(e);
  }
}

class ArraySum {
  private processor: SumProcessor;
  constructor(accumulator: number) {
    this.processor = new SumProcessor(accumulator);
  }
  process(arr: number[]) {
    for (let i = 0; i < arr.length; i++) {
      this.processElement(arr[i]);
    }
    return this.processor.getAccumulator();
  }

  processElement(e: number) {
    this.processor.processElement(e);
  }
}
```

6. 새 클래스의 나머지 오류에 대해 해당 값을 대체할 매개변수를 추가한다.(지금의 경우에는 필요 없음)

7. 메서드 인라인화를 사용해 1단계의 추출을 반대로 한다.

```typescript
class ArrayMinimum {
  private processor: MinimumProcessor;
  constructor(accumulator: number) {
    this.processor = new MinimumProcessor(accumulator);
  }
  process(arr: number[]) {
    for (let i = 0; i < arr.length; i++) {
      this.processor.processElement(arr[i]);
    }
    return this.processor.getAccumulator();
  }
}

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
```

이제 두 개의 원래 클래스인 `ArrayMinimum`과 `ArraySum`은 생성자에서 초기화를 제외하고는 동일하다. 리팩터링 이후 코드의 전문이다.

```typescript
class ArrayMinimum {
  private processor: MinimumProcessor;
  constructor(accumulator: number) {
    this.processor = new MinimumProcessor(accumulator);
  }
  process(arr: number[]) {
    for (let i = 0; i < arr.length; i++) {
      this.processor.processElement(arr[i]);
    }
    return this.processor.getAccumulator();
  }
}

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

class MinimumProcessor {
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
