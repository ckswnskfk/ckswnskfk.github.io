---
title: "[F-Lab 모각코 챌린지] 48일차"
excerpt: "five lines of code - 4"
categories:
  - Refactoring
tags:
  - f-lab
  - 에프랩
  - TIL
  - Typescript
  - Five lines of code
last_modified_at: 2023-07-06
toc: true
---

어제에 이어 파이브 라인스 오브 코드 3장을 요약해본다.

# 긴 코드 조각내기

## 추상화 수준을 맞추기 위한 함수 분해

### 규칙: 호출 또는 전달, 한 가지만 할 것

#### 정의

함수 내에서 메서드를 호출하거나 인자로 전달하는 걸 둘 다 섞어서 사용해서는 안된다.

#### 설명

메서드가 많아지고 매개변수로 전달되는 게 많아지면 책임이 고르게 되지 않을 수 있다. 예를 들어, 어떤 함수 내에서 배열에 인덱스를 설정하는 것과 같은 직접적인 작업을 수행하고 그 배열을 더 복잡한 함수에 인자로 전달한다고 해보자. 그러면 그 함수는 배열을 직접 조작하는 낮은 수준의 작업과, 다른 함수에 인자로 전달하는 높은 수준의 호출이 공존해서 메서드 이름 사이의 불일치가 발생해 가독성이 떨어질 수 있다. 동일한 수준의 추상화를 유지하는 것이 코드를 읽기 훨씬 쉽다.

아래의 예제를 보자.

```typescript
// 배열의 평균을 구하는 함수
function average(arr: number[]) {
  return sum(arr) / arr.length;
}
```

이 예제는 규칙을 위반하고 있다. 어떻게 추상화 수준을 맞출까?

```typescript
function average(arr: number[]) {
  return sum(arr) / size(arr);
}
```

#### 스멜

'함수의 내용은 동일한 추상화 수준에 있어야 한다'는 말은 그 자체가 스멜일 정도로 강력하다. 전달된 인자로 메서드가 어떻게 사용되었는지 식별하는 간단한 방법이 있는데, 인자로 전달된 변수 옆의 . 으로 쉽게 찾을 수 있다.

#### 참조

이 규칙은 메서드 추출 리팩터링 패턴을 참조하는 게 도움이 된다. 클린 코드에서 찾을 수 있다.

## 좋은 함수 이름의 속성

좋은 이름이 가져야 할 몇 가지 속성은 다음과 같다.

- 정직해야 한다. 함수의 의도를 설명해야 한다.
- 완전해야 한다. 함수가 하는 모든 것을 담아야 한다.
- 도메인에서 일하는 사람이 이해할 수 있어야 한다. 작업중인 도메인에서 사용하는 단어를 사용해라. 의사소통이 더욱 효율적이게 될 것이다.

먼저 코드가 무엇을 하는지 고려해야 한다. 다음은 책에서 사용하는 예제 코드이다.

```typescript
// 변경 전
function draw() {
  let canvas = document.getElementById("GameCanvas") as HTMLCanvasElement;
  let g = canvas.getContext("2D");

  g.clearRect(0, 0, canvas.width, canvas.height);

  drawMap(g);
  drawPlayer(g);
}
```

```typescript
// 변경 후
function creatGraphics() {
  let canvas = document.getElementById("GameCanvas") as HTMLCanvasElement;
  let g = canvas.getContext("2D");

  g.clearRect(0, 0, canvas.width, canvas.height);
  return g;

  drawMap(g);
  drawPlayer(g);
}

function draw() {
  let g = creatGraphics();

  drawMap(g);
  drawPlayer(g);
}
```

함수명을 지을 때는 항상 함수가 더 작아졌을 때 이름을 개선할 수 있는지를 평가해 봐야 한다.

## 너무 많은 일을 하는 함수 분리하기

### 규칙: if문은 함수의 시작에만 배치

#### 정의

if문이 있는 경우 해당 if문은 함수의 첫 번째 항목이어야 한다.

#### 설명

함수에서는 한 가지 일만 해야한다. 무언가 확인하는 것은 한 가지 일이다. 따라서 함수에 if가 있는 경우 함수의 첫 번째 항목이어야 한다. 또한 그 후에 아무것도 해서는 안된다는 의미에서 유일한 것이어야 한다. 메서드를 추출해서 if 문 다음에 다른 작업을 하는 것을 피할 수 있다.

if문이 메서드가 하는 유일한 일이어야 한다는 말은 곧 그 본문을 추출할 필요가 없으며, 또한 else문과 분리해서는 안된다는 말이다. 동작과 구조는 밀접하게 연결되어 있으며 리팩터링할 때 동작을 변경해서는 안되므로 구조도 변경해서는 안된다.

```typescript
// 2에서 n까지의 모든 소수를 출력하는 함수
function reportPrimes(n: number) {
  for (let i = 2; i < n; i++>) {
    if (isPrime(i)) {
      console.log(`${i} 는 소수입니다.`);
    }
  }
}
```

두 가지 작업이 존재한다.

1. 숫자 반복
2. 소수인지 확인

```typescript
function reportPrimes(n: number) {
  for (let i = 2; i < n; i++) {
    reportIfPrime(i);
  }
}

function reportIfPrime(n: number) {
  if (isPrime(n)) {
    console.log(`${i} 는 소수입니다.`);
  }
}
```

#### 스멜

다섯 줄 제한과 같이 이 규칙은 함수가 한 가지 이상의 작업을 수행하는 스멜을 막기 위해 존재한다.

#### 의도

if문이 하나의 작업이기 때문에 이어지는 else if 와 if를 분리할 수 없는 하나의 원자 단위로 본다. 즉 if 문이 else if 문과 함께 있을 때 메서드 추출을 수행하는 단위는 if 문과 이어지는 else if문 까지이다.

#### 참고

이 규칙은 메서드 추출을 참조해라. 클릭 코드에서 찾을 수 있다.
