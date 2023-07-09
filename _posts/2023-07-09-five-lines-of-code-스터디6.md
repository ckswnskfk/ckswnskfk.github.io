---
title: "[F-Lab 모각코 챌린지] 51일차"
excerpt: "five lines of code - 6"
categories:
  - Refactoring
tags:
  - f-lab
  - 에프랩
  - TIL
  - Typescript
  - Five lines of code
last_modified_at: 2023-07-09
toc: true
---

계속 이어서 책을 따라가보자.

# 타입 코드 처리하기

## 간단한 if 문 리팩터링

### 리팩터링 패턴: 클래스로의 코드 이관

#### 설명

이 패턴은 기능을 클래스로 옮기기 때문에 클래스로 타입 코드 대체 패턴의 연장선에 있다. if 구문이 제거되고 기능이 데이터와 가까운 곳에 위치하게 된다. 이는 불변속성을 지역화하는 데 도움이 된다.

코드를 먼저 추출하지 않고 메서드를 옮길 수도 있지만 문제가 없는지 확인하기 더 어렵다.

#### 절차

1. 원래 함수를 복사해 모든 클래스에 붙여 넣는다. 컨텍스트를 this로 바꾸고 사용하지 않는 매개 변수를 제거한다. 메서드엔 여전히 잘못된 이름이 있어서 오류가 난다.

2. 메서드 선언 부분의 메서드 이름과 매개변수 리스트를 인터페이스에 복사하고 원래 메서드와 조금 다른 이름을 지정한다.

3. 모든 클래스에서 새로운 메서드를 점검한다.

4. 원래 함수의 본문을 새 메서드에 대한 호출로 바꾼다.

#### 예제

클래스로 타입 코드 대체 패턴과 밀접한 연관이 있기에 신호등 예제를 계속해서 사용한다.

```typescript
interface TrafficLight {
  isRed(): boolean;
  isYellow(): boolean;
  isGreen(): boolean;
}

class Red implements TrafficLight {
  isRed() {
    return true;
  }
  isYellow() {
    return false;
  }
  isGreen() {
    return false;
  }
}
class Yellow implements TrafficLight {
  isRed() {
    return false;
  }
  isYellow() {
    return true;
  }
  isGreen() {
    return false;
  }
}
class Green implements TrafficLight {
  isRed() {
    return false;
  }
  isYellow() {
    return false;
  }
  isGreen() {
    return true;
  }
}

function updateCarForLight(current: TrafficLight) {
  if (current.isRed()) {
    car.stop();
  } else {
    car.drive();
  }
}
```

이제 절차를 따라 코드를 수정해보자.

1. 인터페이스에 새 메서드를 만든다. 옮길 함수와 조금 다른 이름을 짓는다.

```typescript
interface TrafficLight {
  // ...
  updateCar(): void;
}
```

2. 옮길 함수를 복사해 모든 클래스에 붙여 넣는다. function 키워드를 제거하고 컨텍스트를 this로 대체한다. 사용하지 않는 매개변수를 제거한다. 이때는 함수명을 맞추지 않아 오류가 발생한다.

```typescript
class Red implements TrafficLight {
  // ...

  updateCarForLight() {
    if (this.isRed()) {
      car.stop();
    } else {
      car.drive();
    }
  }
}

class Yellow implements TrafficLight {
  // ...

  updateCarForLight() {
    if (this.isRed()) {
      car.stop();
    } else {
      car.drive();
    }
  }
}

class Green implements TrafficLight {
  // ...

  updateCarForLight() {
    if (this.isRed()) {
      car.stop();
    } else {
      car.drive();
    }
  }
}
```

3. 모든 클래스에서 새 메서드를 점검한다.
   - 클래스에 맞게 조건식 수정
   - 미리 계산할 수 있는 모든 계산을 수행한다.

```typescript
class Red implements TrafficLight {
  // ...

  updateCarForLight() {
    if (true) {
      car.stop();
    } else {
      car.drive();
    }
  }
}

class Yellow implements TrafficLight {
  // ...

  updateCarForLight() {
    if (false) {
      car.stop();
    } else {
      car.drive();
    }
  }
}

class Green implements TrafficLight {
  // ...

  updateCarForLight() {
    if (false) {
      car.stop();
    } else {
      car.drive();
    }
  }
}
```

```typescript
class Red implements TrafficLight {
  // ...

  updateCarForLight() {
    car.stop();
  }
}

class Yellow implements TrafficLight {
  // ...

  updateCarForLight() {
    car.drive();
  }
}

class Green implements TrafficLight {
  // ...

  updateCarForLight() {
    car.drive();
  }
}
```

메서드명을 적절히 바꿔 이 메서드 처리가 끝났음을 알린다.

```typescript
class Red implements TrafficLight {
  // ...

  updateCar() {
    car.stop();
  }
}

class Yellow implements TrafficLight {
  // ...

  updateCar() {
    car.drive();
  }
}

class Green implements TrafficLight {
  // ...

  updateCar() {
    car.drive();
  }
}
```

4. 원래 함수의 본문을 새 메서드에 대한 호출로 바꾼다.

```typescript
function updateCarForLight(current: TrafficLight) {
  current.updateCar();
}
```

#### 추가 자료

이 리팩터링은 마틴 파울러의 메서드 이동과 동일하다. 하지만 클래스로의 코드 이관이 의도와 강력함을 더 잘 전달한다고 한다.

### 리팩터링 패턴: 메서드의 인라인화

#### 설명

이 책의 두 가지 중요한 주제는 코드 추가, 코드 제거이다. 이 리팩터링 패턴은 후자를 지원한다. 더 이상 가독성에 도움이 되지 않는 메서드를 제거하는 것이다.

이 책에서는 흔히 메서드가 한 줄만 있는 경우 이 작업을 수행한다.

하지만 다음과 같은 함수는 낮은 수준의 연산을 수행하는 함수는 인라인화 하지 않는 것이 가독성에 더 좋다.

```typescript
const NUMBER_BITS = 32;
function absolute(x: number) {
  return (x ^ (x >> (NUMBER_BITS - 1))) - (x >> (NUMBER_BITS - 1));
}
```

#### 절차

1. 메서드 이름을 임시로 변경. 함수 사용하는 모든 곳에서 컴파일러 오류 발생
2. 메서드 본문을 복사하고 매개변수를 기억해둠.
3. 오류가 생긴 모든 곳의 호출을 복사된 코드로 교체하고 인자를 매개변수에 매핑
4. 오류 없으면 원래 메서드 삭제.

#### 예제

한 계좌에서 돈을 인출해 다른 계좌에 입금하는 은행 거래의 예시이다. 코드는 두 부분으로 분할되어 있는데 이는 실수로 메서드 호출을 잘못하면 출금 없이 돈이 입금될 수도 있음을 의미한다. 그래서 두 메서드를 결합하기로 했다.

```typescript
// 변경 전
function deposit(to: string, amount: number) {
  let accountId = database.find(to);
  database.updateOne(accountId, { $inc: { balance: amount } });
}

function transfer(from: string, to: string, amount: number) {
  deposit(from, -amount);
  deposit(to, amount);
}
```

절차를 따라 변경해보자.

1. 메서드이 이름을 임시로 변경해 함수를 사용하는 모든 곳에서 오류가 나게 한다.

```typescript
function deposit2(to: string, amount: number) {
  // ...
}
```

2. 메서드 본문을 복사하고 매개변수를 기억.
3. 컴파일러에서 오류가 발생한 모든 곳에서 메서드 호출을 복사된 코드로 변경하고 인자를 매개변수에 매핑

```typescript
function transfer(from: string, to: string, amount: number) {
  let fromAccountId = database.find(from);
  database.updateOne(fromAccountId, { $inc: { balance: -amount } });
  let toAccountId = database.find(to);
  database.updateOne(toAccountId, { $inc: { balance: amount } });
}
```

4. 오류가 없으면 원래 메서드를 지운다.

이 코드가 좋은지 아닌지는 논란의 여지가 있다. 나중에 다시 살펴본다.
