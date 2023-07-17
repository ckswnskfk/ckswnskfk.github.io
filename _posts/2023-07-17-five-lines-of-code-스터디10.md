---
title: "[F-Lab 모각코 챌린지] 59일차"
excerpt: "five lines of code - 10"
categories:
  - Refactoring
tags:
  - f-lab
  - 에프랩
  - TIL
  - Typescript
  - Five lines of code
last_modified_at: 2023-07-17
toc: true
---

# 유사한 코드 융합하기

## 단순한 조건 통합하기

### 리팩터링 패턴: if문 결합

#### 설명

말 그대로 연속적인 if문을 결합해서 중복을 제거한다. `||`으로 처리할 수 있다.

#### 절차

1. 본문이 동일한지 확인
2. 첫 번째 if문의 닫는 괄호와 else if 문의 여는 괄호 사이의 코드를 삭제하고 `||`를 삽입한다. if 뒤에 여는 괄호를 삽입하고 중괄호 앞에는 닫는 괄호를 삽입한다. 동작을 변경하지 않도록 항상 표현식을 괄호로 묶는다.

```typescript
if (exp1) {
  // 본문
} else if (exp2) {
  // 동일한 본문
}
```

```typescript
if (exp1 || exp2) {
  // 본문
}
```

#### 예제

invoice로 무엇을 할 지 결정하는 예제로 해보자.

```typescript
if (today.getDate() === 1 && account.getBalance() > invoice.getAmount()) {
  account.pay(bill);
} else if (invoice.isLastDayOfPayment() && invoice.isApproved()) {
  account.pay(bill);
}
```

절차대로 해보면 다음과 같이 된다.

```typescript
if (
  (today.getDate() === 1 && account.getBalance() > invoice.getAmount()) ||
  (invoice.isLastDayOfPayment() && invoice.isApproved())
) {
  account.pay(bill);
}
```

### 규칙: 순수 조건 사용

#### 정의

조건은 항상 순수 조건이어야 한다.

#### 설명

'순수 조건'은 조건에 부수적인 동작이 없음을 의미한다. 부수적인 동작이란 조건이 변수에 값을 할당하거나 예외를 발생기키거나 출력, 파일 쓰기 등과 같이 I/O 상호작용 하는 것을 말한다.

순수한 조건을 갖는 것이 중요한 이유는 다음과 같다.

1. 부수적인 동작이 존재하면 앞서 언급한 규칙들을 사용할 수 없다.
2. 부수적인 동작은 조건문에서 흔하게 사용되지 않기 때문에 예상할 수 없다. 즉, 부수적인 동작을 확인하고 추적하는 데 시간과 노력이 든다.

다음 예제를 살펴보자.

```typescript
class Reader {
  private data: string[];
  private current: number;

  readLine() {
    this.current++;
    return this.data[this.current] || null;
  }
}

// ...

let br = new Reader();
let line: string | null;
while ((line = br.readLine()) !== null) {
  console.log(line);
}
```

`while`의 조건은 부수적인 동작을 하므로 이 순수하지 않다. 이것을 분리해보자.

```typescript
class Reader {
  private data: string[];
  private current: number;

  nextLine() {
    // 부수적인 동작을 가진 새 메서드
    this.current++;
  }

  readLine() {
    return this.data[this.current] || null;
  }
}

// ...

let br = new Reader();
for (; br.readLine() !== null; br.nextLine()) {
  let line = br.readLine();
  console.log(line);
}
```

줄을 가져오고 포인터를 이동하는 책임을 분리했다. `readLine()`은 부수적인 동작 없이 원하는 만큼 호출할 수 있다.

우리가 구현을 통제할 수 없으므로 부수적인 동작에서 `return`을 분리할 수 없을 경우 캐시를 사용할 수 있다.

```typescript
class Reader {
  private data: string[];
  private current: number;

  nextLine() {
    this.current++;
  }

  readLine() {
    return this.data[this.current] || null;
  }
}

// ...

class Cacher<T> {
  private data: T;

  constructor(private mutator: () => T) {
    this.data = this.mutator();
  }

  get() {
    return this.data;
  }

  next() {
    this.data = this.mutator();
  }
}

let tmBr = new Reader();
let br = new Cacher(() => tmBr.readLine());

for (; br.get() !== null; br.next()) {
  let line = br.get();
  console.log(line);
}
```

#### 스멜

이 규칙은 '명령에서 질의 분리' 라는 일반적인 스멜에서 비롯한다. '명령'은 부수적인 동작이 있는 모든 것을 의미하고 '질의'는 순수한 것을 의미한다. 이 스멜을 따르는 쉬운 방법은 void 메서드에서만 부수적인 동작을 허용하는 것이다. 즉, 부수적인 동작을 하거나 무언가를 반환하는 것 중 하나만 하면 된다.

'명령에서 질의 분리' 스멜과 순수 조건 사용 규칙의 차이는 정의보다는 호출에 초점을 맞추는 것이다.

또한 이 규칙은 '메서드는 한 가지 작업을 해야 한다'는 일반적인 원칙에 근거한다. 부수적인 동작은 한 가지 작업이고 무언가 반환하는 것은 별개이다.

#### 의도

목적은 데이터를 가져오는 것과 변경하는 것을 분리하는 것이다. 이는 코드를 더 깔끔하고 예측 가능하게 만든다. 메서드가 더 간단하기 때문에 더 좋은 이름을 지을 수 있다.
