---
title: "[F-Lab 모각코 챌린지] 50일차"
excerpt: "five lines of code - 5"
categories:
  - Refactoring
tags:
  - f-lab
  - 에프랩
  - TIL
  - Typescript
  - Five lines of code
last_modified_at: 2023-07-08
toc: true
---

파이브 라인스 오브 코드 4장을 요약해보자.

# 타입 코드 처리하기

## 간단한 if 문 리팩터링

if 문과 이어지는 else if문을 어떻게 다룰지에 대한 새로운 규칙을 도입해보자.

### 규칙: if 문에서 else를 사용하지 말 것

#### 정의

프로그램에서 이해하지 못하는 타입인지를 검사하지 않는 한 if 문에서 else를 사용하지 마라.

#### 설명

사람들은 너무나 쉽게 if-else 문을 작성하지만 그렇지 않는 게 좋다. if-else를 사용하면 코드에서 결정이 내려지는 지점을 결정하게 되고 이는 다른 변형을 수용할 유연성이 떨어지게 된다. 마치 하드코딩처럼.

그래서 else 구문을 사용하지 않는 게 좋은데, 그러려면 무엇을 검사하는지에 주의를 기울여야 한다. 예를 들어 e.key를 사용해 어떤 키가 눌렸는지 검사할 때에는 string 타입을 검사하므로 else if를 피할 수 없다.

하지만 이 경우 외부에서 값을 가져오기에 문제가 되지 않는다. 가장 먼저 할 일은 외부의 데이터 타입을 내부에서 제어 가능한 데이터 타입으로 매핑하는 것이다.

```typescript
window.addEventListener("keydown", (e) => {
  if (e.key === LEFT_KEY || e.key === "a") inputs.push(Input.LEFT);
  if (e.key === UP_KEY || e.key === "w") inputs.push(Input.UP);
  if (e.key === RIGHT_KEY || e.key === "d") inputs.push(Input.RIGHT);
  if (e.key === DOWN_KEY || e.key === "s") inputs.push(Input.DOWN);
});
```

예제에서 KeyboardEvent 와 string 중 어느 것도 우리가 결정할 수 없다. 이런 else id 체인은 데이터의 입출력에 직접 연결되어야 하며 애플리케이션의 나머지 부분과는 분리되어야 한다.

참고로 독립된 if 문은 '검사'로 간주하고, if-else 문은 '의사결정' 으로 간주한다.

```typescript
// 변경 전
function average(ar: number[]) {
  if (size(ar) === 0) {
    throw "빈 배열은 안돼요";
  } else {
    return sum(ar) / size(ar);
  }
}
```

```typescript
// 변경 후
function assertNotEmpty(ar: number[]) {
  if (sizr(ar) === 0) {
    throw "빈 배열은 안돼요";
  }
}

function average(ar: number[]) {
  assertNotEmpty(ar);
  return sum(ar) / size(ar);
}
```

위 예제를 보면 if문을 검사로만 수행하기 위해 리팩터링을 했다.

#### 스멜

이 규칙은 이른 바인딩(early binding) 스멜과 관련이 있다. 프로그램은 if-else와 같은 의사결정 동작은 컴파일 시 고정되므로 개컴파일 없이는 수정할 수 없다. 반대로 실행 순간에 동작이 결정되는 것이 늦은 바인딩(late binding)이다.

#### 의도

객체지향 프로그래밍에는 if보다 강력한 제어 흐름 연산자인 객체가 있다. 인터페이스를 사용한 여러 다른 구현이 있는 경우 인스턴스화하는 클래스에 따라 실행할 코드를 결정할 수 있다.

#### 참조

늦은 바인딩은 '클래스로 타입 코드 대체'와 '전략 패턴의 도입'이라는 리팩터링 패턴에서 자세히 다룬다.

### 리팩터링 패턴: 클래스로 타입 코드 대체

#### 설명

이 리팩터링 패턴에서 열거형은 인터페이스로 변환되고 열거형의 값들은 클래스가 된다. 이는 각 값에 속성을 추가하고 해당 특정 값과 관련된 기능을 특성에 맞게 만들 수 있다.

열거형의 값을 클래스로 변환할 때는 다른 열거 값을 고려하지 않고 해당 값과 관련된 기능을 그룹화할 수 있다. 이 프로세스는 기능과 데이터를 함께 제공한다.

타입 코드 또한 열거형이 아닌 형태로 변환할 수 있다. 가장 일반적으로 int 와 enum을 사용한다.

```typescript
const SMALL = 33;
const MEDIUM = 37;
const LARGE = 42;
```

int의 경우 타입 코드의 사용을 추적하기 더 까다롭다. 따라서 타입 코드를 볼 때 항상 즉시 열거형으로 변환한다.

```typescript
enum TShirtSizes {
  SMALL = 33,
  MEDIUM = 37,
  LARGE = 42,
}
```

#### 절차

1. 임시 이름을 가진 새 인터페이스 도입. 인터페이스에는 열거형의 각 값에 대한 메서드가 있어야 함.
2. 열거형의 각 값에 해당하는 클래스 작성. 클래스에 해당하는 메서드를 제외한 인터페이스의 모든 메서드는 false를 반환.
3. 열거형의 이름을 다른 이름으로 바꿈. 컴파일러가 열거형을 사용하는 모든 위치에서 오류 발생시킴.
4. 타입을 이전 이름에서 임시 이름으로 변경하고 일치성 검사를 새로운 메서드로 대체.
5. 남아있는 열거형 값에 대한 참조 대신 새 클래스를 인스턴스화해 교체.
6. 오류가 더 이상 없으면 인터페이스의 이름을 모든 위치에서 영구적인 것으로 변경.

#### 예제

신호등의 신호를 가리키는 열거형과 차가 지나가도 되는지 여부를 결정하는 함수가 있는 간단한 예를 보자.

```typescript
enum TrafficLight {
  RED,
  YELLOW,
  GREEN,
}

const CYCLE = [TrafficLight.RED, TrafficLight.GREEN, TrafficLight.YELLOW];

function updateCarForLight(current: TrafficLight) {
  if (current === TrafficLight.RED) {
    car.stop();
  } else {
    car.drive();
  }
}
```

이제 절차를 따라 수정해보자.

1. 임시 이름을 가진 새 인터페이스 도입. 인터페이스에는 열거형의 각 값에 대한 메서드가 있어야 함.

```typescript
interface TrafficLight2 {
  isRed(): boolean;
  isYellow(): boolean;
  isGreen(): boolean;
}
```

2. 열거형의 각 값에 해당하는 클래스 작성. 클래스에 해당하는 메서드를 제외한 인터페이스의 모든 메서드는 false를 반환.

```typescript
class Red implements TrafficLight2 {
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
class Yellow implements TrafficLight2 {
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
class Green implements TrafficLight2 {
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
```

3. 열거형의 이름을 다른 이름으로 바꿈. 컴파일러가 열거형을 사용하는 모든 위치에서 오류 발생시킴.

```typescript
enum RawTrafficlight {
  RED,
  YELLOW,
  GREEN,
}
```

4. 타입을 이전 이름에서 임시 이름으로 변경하고 일치성 검사를 새로운 메서드로 대체.

```typescript
function updateCarForLight(current: TrafficLight2) {
  if (current.isRed()) {
    car.stop();
  } else {
    car.drive();
  }
}
```

5. 남아있는 열거형 값에 대한 참조 대신 새 클래스를 인스턴스화해 교체.

```typescript
const CYCLE = [new Red(), new Green(), new Yellow()];
```

6. 오류가 더 이상 없으면 인터페이스의 이름을 모든 위치에서 영구적인 것으로 변경.

```typescript
enum TrafficLight {}
// ...
```

이 리팩터링 패턴은 당장은 별로 가치가 없어보이지만 나중에 환상적인 개선을 가능하게 한다.

아직 갈 길이 멀다. 내일도 쭉 이어서 가보자.
