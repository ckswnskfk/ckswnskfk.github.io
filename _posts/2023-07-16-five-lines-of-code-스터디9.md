---
title: "[F-Lab 모각코 챌린지] 58일차"
excerpt: "five lines of code - 9"
categories:
  - Refactoring
tags:
  - f-lab
  - 에프랩
  - TIL
  - Typescript
  - Five lines of code
last_modified_at: 2023-07-16
toc: true
---

오늘도 이어서 파이브 라인스 오브 코드의 규칙들을 알아보자.

# 유사한 코드 융합하기

## 유사한 클래스 통합하기

### 리팩터링 패턴: 유사 클래스 통합

#### 설명

공통된 상수 메서드가 클래스에 따라 다른 값을 반환할 때마다 이 패턴을 사용해 클래스를 통합할 수 있다. 이 상수 메서드 집합을 `기준`이라고 한다. 그리고 상수 메서드가 두 개일 때 두 개일 때 `두 개의 접점을 가진 기준`이라고 한다.

클래스의 수가 적어진다는 건 일반적으로 더 많은 구조를 발견했다는 것이므로 클래스를 통합하는 건 좋다.

#### 절차

1. 첫 단계는 모든 비기준 메서드를 동일하게 만드는 것이다. 이런 메서드에 각각 다음을 수행한다.
   - 각 메서드 버전 본문의 기준 코드 상위에 `if (true) {}`를 추가한다.
   - if문의 `true`를 모든 기본 메서드를 호출하여 그 결과를 상수 값과 비교하는 표현식으로 바꾼다.
   - 각 버전의 본문을 복사하고 `else`와 함께 다른 모든 버전에 붙여 넣는다.
2. 기준 메서드만 다르므로 기준 메서드에 각 메서드에 대한 필드를 도입하고 생성자에서 상수를 할당한다.
3. 상수 대신 도입한 필드를 반환하도록 메서드를 변경한다.
4. 컴파일해 문제가 없는지 확인한다.
5. 각 클래스에 대해 한 번에 하나의 필드씩 다음을 수행한다.
   - 필드의 기본값을 매개변수로 지정하게 한다.
   - 컴파일러 오류를 살펴보고 기본값을 인자로 전달한다.
6. 모든 클래스가 동일하면 통합한 클래스 중 하나를 제외한 모두를 삭제하고, 삭제하지 않은 클래스로 바꾸어 모든 컴파일러 오류를 수정한다.

#### 예제

세 개의 신호등 클래스가 있는데, 서로 유사하다. 이것들을 통합해보자.

```typescript
function nextColor(t: TrafficColor) {
  if (t.color() === "red") return new Green();
  else if (t.color() === "green") return new Yellow();
  else if (t.color() === "yellow") return new Red();
}

interface TrafficColor {
  color(): string;
  check(car: Car): void;
}

class Red implements TrafficColor {
  color(): string {
    return "red";
  }

  check(car: Car): void {
    car.stop();
  }
}

class Yellow implements TrafficColor {
  color(): string {
    return "yellow";
  }

  check(car: Car): void {
    car.stop();
  }
}

class Green implements TrafficColor {
  color(): string {
    return "green";
  }

  check(car: Car): void {
    car.drive();
  }
}
```

이제 절차를 따라가보자.

1. 기준 메서드는 `color()` 인데 클래스마다 다른 상수를 반환하므로 `check()` 메서드를 동일하게 만들어야 한다.
   - 각 버전의 `check()`의 본문의 깆존 코드 주위에 `if (true) {}`를 추가한다.

```typescript
class Red implements TrafficColor {
  color(): string {
    return "red";
  }

  check(car: Car): void {
    if (true) {
      car.stop();
    }
  }
}

class Yellow implements TrafficColor {
  color(): string {
    return "yellow";
  }

  check(car: Car): void {
    if (true) {
      car.stop();
    }
  }
}

class Green implements TrafficColor {
  color(): string {
    return "green";
  }

  check(car: Car): void {
    if (true) {
      car.drive();
    }
  }
}
```

- 기준 메서드를 호출해 `true`와 그 결과를 상수 값과 비교하는 표현식으로 바꾼다.

```typescript
class Red implements TrafficColor {
  color(): string {
    return "red";
  }

  check(car: Car): void {
    if (this.color() === "red") {
      car.stop();
    }
  }
}

class Yellow implements TrafficColor {
  color(): string {
    return "yellow";
  }

  check(car: Car): void {
    if (this.color() === "yellow") {
      car.stop();
    }
  }
}

class Green implements TrafficColor {
  color(): string {
    return "green";
  }

  check(car: Car): void {
    if (this.color() === "green") {
      car.drive();
    }
  }
}
```

- 각 버전의 본문을 복사해 다른 모든 버전의 본문에 `else` 와 함께 붙여 넣는다.

```typescript
class Red implements TrafficColor {
  color(): string {
    return "red";
  }

  check(car: Car): void {
    if (this.color() === "red") {
      car.stop();
    } else if (this.color() === "yellow") {
      car.stop();
    } else if (this.color() === "green") {
      car.drive();
    }
  }
}

class Yellow implements TrafficColor {
  color(): string {
    return "yellow";
  }

  check(car: Car): void {
    if (this.color() === "red") {
      car.stop();
    } else if (this.color() === "yellow") {
      car.stop();
    } else if (this.color() === "green") {
      car.drive();
    }
  }
}

class Green implements TrafficColor {
  color(): string {
    return "green";
  }

  check(car: Car): void {
    if (this.color() === "red") {
      car.stop();
    } else if (this.color() === "yellow") {
      car.stop();
    } else if (this.color() === "green") {
      car.drive();
    }
  }
}
```

2. 이제 check 메서드들은 동일하고 기준 메서드인 `color()`만 다르다. `color()`메서드에 대한 필드를 도입하고 생성자에서 상수를 할당하는 것으로 시작한다.

```typescript
class Red implements TrafficColor {
  constructor(private col: string = "red") {}

  color(): string {
    return "red";
  }

  check(car: Car): void {
    if (this.color() === "red") {
      car.stop();
    } else if (this.color() === "yellow") {
      car.stop();
    } else if (this.color() === "green") {
      car.drive();
    }
  }
}

class Yellow implements TrafficColor {
  constructor(private col: string = "yellow") {}

  color(): string {
    return "yellow";
  }

  check(car: Car): void {
    if (this.color() === "red") {
      car.stop();
    } else if (this.color() === "yellow") {
      car.stop();
    } else if (this.color() === "green") {
      car.drive();
    }
  }
}

class Green implements TrafficColor {
  constructor(private col: string = "green") {}

  color(): string {
    return "green";
  }

  check(car: Car): void {
    if (this.color() === "red") {
      car.stop();
    } else if (this.color() === "yellow") {
      car.stop();
    } else if (this.color() === "green") {
      car.drive();
    }
  }
}
```

3. 상수 대신 새 필드를 반환하도록 메서드를 변경한다.

```typescript
class Red implements TrafficColor {
  constructor(private col: string = "red") {}

  color(): string {
    return this.col;
  }

  check(car: Car): void {
    if (this.color() === "red") {
      car.stop();
    } else if (this.color() === "yellow") {
      car.stop();
    } else if (this.color() === "green") {
      car.drive();
    }
  }
}

class Yellow implements TrafficColor {
  constructor(private col: string = "yellow") {}

  color(): string {
    return this.col;
  }

  check(car: Car): void {
    if (this.color() === "red") {
      car.stop();
    } else if (this.color() === "yellow") {
      car.stop();
    } else if (this.color() === "green") {
      car.drive();
    }
  }
}

class Green implements TrafficColor {
  constructor(private col: string = "green") {}

  color(): string {
    return this.col;
  }

  check(car: Car): void {
    if (this.color() === "red") {
      car.stop();
    } else if (this.color() === "yellow") {
      car.stop();
    } else if (this.color() === "green") {
      car.drive();
    }
  }
}
```

4. 문제 확인을 위해 컴파일.
5. 각 클래스에 대해 한 필드씩 다음 절차 수행
   - 필드의 기본값 매개변수화

```typescript
class Red implements TrafficColor {
  constructor(private col: string) {} // 기본 값 제거

  color(): string {
    return this.col;
  }

  check(car: Car): void {
    if (this.color() === "red") {
      car.stop();
    } else if (this.color() === "yellow") {
      car.stop();
    } else if (this.color() === "green") {
      car.drive();
    }
  }
}

class Yellow implements TrafficColor {
  constructor(private col: string) {} // 기본 값 제거

  color(): string {
    return this.col;
  }

  check(car: Car): void {
    if (this.color() === "red") {
      car.stop();
    } else if (this.color() === "yellow") {
      car.stop();
    } else if (this.color() === "green") {
      car.drive();
    }
  }
}

class Green implements TrafficColor {
  constructor(private col: string) {} // 기본 값 제거

  color(): string {
    return this.col;
  }

  check(car: Car): void {
    if (this.color() === "red") {
      car.stop();
    } else if (this.color() === "yellow") {
      car.stop();
    } else if (this.color() === "green") {
      car.drive();
    }
  }
}
```

- 컴파일 오류를 살펴보며 기본값을 인자로 전달

```typescript
function nextColor(t: TrafficColor) {
  if (t.color() === "red") return new Green();
  else if (t.color() === "green") return new Yellow();
  else if (t.color() === "yellow") return new Red("red");
}
```

6. 모든 클래스가 동일하게 되면 통합 클래스 중 하나를 제외한 다른 모두를 삭제하고 컴파일 오류가 난 곳은 삭제하지 않은 클래스로 변경

```typescript
function nextColor(t: TrafficColor) {
  if (t.color() === "red") return new Red("green");
  else if (t.color() === "green") return new Red("yellow");
  else if (t.color() === "yellow") return new Red("red");
}
```

이제 전체 코드는 아래와 같다. 인터페이스가 필요 없어졌고 `Red` 클래스의 이름을 바꿔주기만 하면 될 것이다.

```typescript
function nextColor(t: TrafficColor) {
  if (t.color() === "red") return new Red("green");
  else if (t.color() === "green") return new Red("yellow");
  else if (t.color() === "yellow") return new Red("red");
}

interface TrafficColor {
  color(): string;
  check(car: Car): void;
}

class Red implements TrafficColor {
  constructor(private col: string) {}

  color(): string {
    return this.col;
  }

  check(car: Car): void {
    if (this.color() === "red") {
      car.stop();
    } else if (this.color() === "yellow") {
      car.stop();
    } else if (this.color() === "green") {
      car.drive();
    }
  }
}
```
