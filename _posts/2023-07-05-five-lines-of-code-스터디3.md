---
title: "[F-Lab 모각코 챌린지] 47일차"
excerpt: "five lines of code - 3"
categories:
  - Refactoring
tags:
  - f-lab
  - 에프랩
  - TIL
  - Typescript
  - Five lines of code
last_modified_at: 2023-07-05
toc: true
---

이제부터 1부의 시작으로, 4단계의 개선을 각 장으로 구성했다. 그 시작이 되는 3장을 요약해본다.

# 긴 코드 조각내기

나름대로 노력을 해도 코드는 쉽게 지저분하고 혼란스러워진다. 주된 원인은 다음과 같다.

- 메서드가 여러 가지 일을 수행
- 낮은 수준의 원시 연산
- 주석이나 좋은 변수명 같이 사람이 읽을 수 있는 텍스트 부족

이번 장에서는 너무 많은 역할을 하는 메서드를 식별하는 구체적인 방법을 설명한다.

책의 예제를 통해 리팩터링 패턴인 메서드 추출을 배운다. 메서드 추출이 어떤 문제를 완화하고 가독성을 높이는지 살펴볼 것이다.

메서드와 함수를 구분해서 사용할 것이다.

## 첫 번째 규칙: 왜 다섯 줄인가?

이 책에서 가장 근본적인 규칙인 다섯 중 제한을 소개한다. 이 책에서 이것은 궁극의 목표이다.

### 규칙: 다섯 줄 제한

#### 정의

메서드는 중괄호를 제외하고 5줄을 초과하면 안된다.

#### 설명

공백과 중괄호를 제외한 문장이 5줄을 초과하면 안된다.

모든 메서드를 이 규칙을 준수하도록 바꿀 수 있다.

특정 수치보다 제한이 있다는 것 자체가 중요하다. 기본 데이터 구조를 탐색하는 데 필요한 값으로 제한을 설정하는 것이 효과적인데, 이 책에서 사용되는 예제 코드는 2차원 배열이 기본 데이터 구조이다. 5줄이 되는 아래의 예시를 보자.

```typescript
// 2차원 배열에 짝수가 존재하는지 확인하는 함수
function containsEven(arr: number[][]) {
  for (let x = 0; x < arr.length; x++) {
    for (let y = 0; y < arr[x].length; y++) {
      if (arr[x][y] % 2 === 0) {
        return true;
      }
    }
  }

  return false;
}
```

아래는 각각 문장이 4개라서 4줄 짜리 코드이다.

```typescript
function isTrue(bool: boolean) {
  if (bool) return true;
  else return false;
}
```

#### 스멜

메서드가 길다는 게 스멜이다.

코드마다 기본 데이터 구조가 다르겠지만, 실제로 5줄 정도로 끝나는 경우가 많다.

#### 의도

5줄의 코드가 있는 4개의 메서드가 20줄인 하나의 메서드보다 훨씬 이해하기 쉽다. 각 메서드의 이름으로 의도도 전달할 수 있다.

#### 참조

메서드 추출 리팩터링을 참고하면 이 규칙을 습득하는 데 도움이 된다.
Clean Code(C. 마틴)와 리팩토링(마틴 파울러)을 참고하라.

## 함수 분해를 위한 리팩터링 패턴 소개

### 리팩터링 패턴: 메서드 추출

#### 설명

메서드 추출은 한 메서드의 일부를 취해 자체 메서드로 추출한다.

#### 절차

1. 추출할 줄의 주변을 빈 줄이나 주석으로 표시해서 그룹화
2. 원하는 이름으로 새로운 빈 메서드 만들기
3. 그룹의 맨 위 부분에서 새로운 메서드 호출
4. 그룹의 모든 줄을 잘라내 새 메서드의 본문에 붙혀넣기
5. 컴파일
6. 매개변수를 도입해서 호출하는 쪽에 오류 발생시킴
7. 이러한 매개변수 중 하나를 반환 값으로 할당해야 할 경우
   - 새로운 메서드의 마지막에 return p;
   - 새로운 메서드를 호출하는 쪽에서 반환 값 할당
8. 컴파일
9. 호출 시 인자를 전달해 오류 해결
10. 사용하지 않는 빈 줄이나 주석 제거

#### 예제

절차가 어떻게 작동하는지 예를 들어 알아보자.

```typescript
// 2차원 배열에서 최소 항목을 찾는 함수
function minimum(arr: number[][]) {
  let result = Number.POSITIVE_INFINITY;
  for (let x = 0; x < arr.length; x++) {
    for (let y = 0; y < arr[x].length; y++) {
      if (result > arr[x][y]) result = arr[x][y]; // 추출하고자 하는 코드
    }
  }
  return result;
}
```

1. 추출할 줄의 주변을 빈 줄이나 주석으로 표시해서 그룹화
2. 새로운 메서드 min 만들기
3. 그룹의 맨 위 부분에서 min 메서드 호출
4. 그룹의 모든 줄을 잘라내 새 메서드의 본문에 붙혀넣기

```typescript
function minimum(arr: number[][]) {
  let result = Number.POSITIVE_INFINITY;
  for (let x = 0; x < arr.length; x++) {
    for (let y = 0; y < arr[x].length; y++) {
      min();
    }
  }
  return result;
}

function min() {
  if (result > arr[x][y]) result = arr[x][y];
}
```

5. 컴파일
6. result, arr, x, y에 대한 매개변수 도입
7. 추출된 함수의 결과 result에 할당.
   - return result; 를 min 메서드의 마지막에 추가
   - 호출하는 쪽에서 반환 값을 result = min(); 과 같이 할당

```typescript
function minimum(arr: number[][]) {
  let result = Number.POSITIVE_INFINITY;
  for (let x = 0; x < arr.length; x++) {
    for (let y = 0; y < arr[x].length; y++) {
      result = min();
    }
  }
  return result;
}

function min(result: number, arr: number[][], x: number, y: number) {
  if (result > arr[x][y]) result = arr[x][y];
  return result;
}
```

8. 컴파일
9. 오류를 발생시칸 인자 result, arr, x, y 전달
10. 필요없는 빈 줄이나 주석 제거

```typescript
function minimum(arr: number[][]) {
  let result = Number.POSITIVE_INFINITY;
  for (let x = 0; x < arr.length; x++) {
    for (let y = 0; y < arr[x].length; y++) {
      result = min(result, arr, x, y);
    }
  }
  return result;
}
```

이 예에서 얻을 수 있는 교훈은 번거롭긴 하지만 안전하다는 것.

이 절차는 코드의 내용을 알지 못해도 아무런 문제 없이 변환할 수 있다는 데에 큰 가치가 있다. 컴파일러가 그걸 도와준다.

3장의 나머지 부분은 내일 요약하자.
