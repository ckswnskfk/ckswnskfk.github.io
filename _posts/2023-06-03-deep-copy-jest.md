---
title: "[F-Lab 모각코 챌린지] 15일차"
excerpt: "deep copy 함수 test"
categories:
  - Javascript
tags:
  - f-lab
  - 에프랩
  - TIL
  - Javascript
  - Deep Copy
  - structuredClone
  - Jest
last_modified_at: 2023-06-03
toc: true
---

코어 자바스크립트 책의 데이터 타입 챕터에 나오는 deep copy 예제를 좀 보완해 보라는 멘토님의 미션이 있었다. git gist로 작성해서 제출해 봤다. 할 말이 많으신 것 같은데 딱 두 가지만 더 해보라고 하셨다. 보호구문으로 if문 리팩토링, jest로 테스트. 거기에 곁들여 말씀하신 structuredClone 을 써 같이 테스트 해 보면 좋을 것 같다.

# 세팅

jest로 함수 유닛 테스트 할 수 있는 환경은 구축해 놓았다. ES 모듈을 쓰기 위해 바벨 세팅까지 간단히 했다. [저번 포스트](https://ckswnskfk.github.io/programing/Jest/)에서 확인할 수 있다.

# deep copy 함수

먼저 책에 나온 함수를 간단하게 설명하자면 인자로 받은 값이 원시 타입이면 바로 해당 값을 리턴하고, 객체 형식(`typeof targer === 'object'`) 이면 내부 프로퍼티를 순환하며 그 프로퍼티를 인자로 자신을 재귀적으로 호출한다.

단순히 자바스크립트의 메모리가 어떻게 저장되는지, 그래서 얕은 복사와 깊은 복사가 무엇인지 설명하기 위한 예제라 실제로 쓸 만한 함수는 아니다. 그래서 나는 최소한 객체는 객체로, 배열은 배열로, Map은 Map으로, Set은 Set으로 복사되는 수준으로 리팩토링 해보기로 했다.

cloneDeep.js 파일을 만들어 함수를 작성해봤다.

```javascript
const cloneDeep = (target) => {
  let result;

  if (typeof target !== "object" || target === null) {
    result = target;
  }

  if (target.constructor === Array) {
    result = [];
    for (let i = 0; i < target.length; i++) {
      result[i] = cloneDeep(target[i]);
    }
  }

  if (target.constructor === Map) {
    result = new Map();
    for (let [key, value] of target) {
      result.set(cloneDeep(key), cloneDeep(value));
    }
  }

  if (target.constructor === Set) {
    result = new Set();
    for (let value of target) {
      result.add(cloneDeep(value));
    }
  }

  if (target.constructor === Object) {
    result = {};
    for (let prop in target) {
      if (!target.hasOwnProperty(prop)) {
        continue;
      }
      result[prop] = cloneDeep(target[prop]);
    }
  }

  return result;
};

export default cloneDeep;
```

처음엔 switch 문으로 작성했었는데 if문이 더 보기 쉬워 보여서 바꿨다. 근데 왜 포스트를 쓰려고 하니까 함수라던가 다른 타입이 들어오면 어떡하지? 하는 생각이 들었다... 너무 좁게만 생각했구나. 오늘은 어쩔 수 없지. 내일 고민해보자.

# 테스트

이제 테스트를 해 볼 차례다. cloneDeep.test.js 파일을 만들고 아래처럼 작성해봤다.

```javascript
import cloneDeep from "./cloneDeep";

describe("내가 만든 cloneDeep과 structuredClone()의 비교", () => {
  const originObj = {
    a: "abc",
    inner: {
      arr: [1, 2, 3],
      map: new Map([["key", "value"]]),
      set: new Set([1, 2, 3]),
    },
  };

  const clonedObj = cloneDeep(originObj);
  const clonedByStructuredClone = structuredClone(originObj);

  test("originObj와 clonedObj의 속성이 같은 값을 가지는지?", () => {
    expect(clonedObj).toStrictEqual(originObj);
  });

  test("originObj와 clonedObj의 속성중 참조 타입인 값들이 같은 주소값을 가리키지는 않는지?", () => {
    expect(clonedObj).not.toBe(originObj);
    expect(clonedObj.inner).not.toBe(originObj.inner);
    expect(clonedObj.inner.arr).not.toBe(originObj.inner.arr);
    expect(clonedObj.inner.map).not.toBe(originObj.inner.map);
    expect(clonedObj.inner.set).not.toBe(originObj.inner.set);
  });

  test("originObj와 clonedByStructuredClone의 속성이 같은 값을 가지는지?", () => {
    /**
     * structuredClone은 toStrictEqual 로 검사했을 때 항상 fail 한다.
     * 한 편, 일반 객체나 배열만 복사해 toEqual로 검사하면 pass 한다.
     * 하지만 Map 이나 Set 타입을 복사해 toEqual로 검사하면 fail 한다.
     * 이유는 잘 모르겠다..
     */
    expect(clonedByStructuredClone).toStrictEqual(originObj);
  });

  test("originObj와 clonedByStructuredClone의 속성중 참조 타입인 값들이 같은 주소값을 가리키지는 않는지?", () => {
    expect(clonedByStructuredClone).not.toBe(originObj);
    expect(clonedByStructuredClone.inner).not.toBe(originObj.inner);
    expect(clonedByStructuredClone.inner.arr).not.toBe(originObj.inner.arr);
    expect(clonedByStructuredClone.inner.map).not.toBe(originObj.inner.map);
    expect(clonedByStructuredClone.inner.set).not.toBe(originObj.inner.set);
  });
});
```

structuredClone() 함수가 node 17부터 지원한다고 해 함께 테스트해 볼 수 있었다. jest의 toBe 가 내부 프로퍼티들까지 비교하지 않는 것으로 알고 있기에 이렇게 더럽게.. 내부 프로퍼티들까지 확인해봤다. 그래서 테스트의 결과는??

```
 FAIL  ./cloneDeep.test.js
  내가 만든 cloneDeep과 structuredClone()의 비교
    √ originObj와 clonedObj의 속성이 같은 값을 가지는 지? (4 ms)
    √ originObj와 clonedObj의 속성중 참조 타입인 값들이 같은 주소값을 가리키지는 않는지? (1 ms)
    × originObj와 clonedByStructuredClone의 속성이 같은 값을 가지는 지? (6 ms)
    √ originObj와 clonedByStructuredClone의 속성중 참조 타입인 값들이 같은 주소값을 가리키지는 않는지? (1 ms)

```

왜 structuredClone 이 통과하지 못한거지???? 이렇게도 저렇게도 테스트 해 본 결과가 위에 적혀 있는 주석이다. 이유는 모른다.. structuredClone() 때문일까? 아니면 Map과 Set이 특이한 탓일까?

오늘은 이렇게 jest를 사용해 내가 만든 함수를 테스트 해 보았다. 포스트 작성하면서도 아쉽거나 허술한 부분이 많이 보이는데 멘토님은 어떻게 보실지; 걱정도 되지만 있는 지금 내 수준이 이런 걸 인정하고 당당하게 피드백 받자. 위축되는 것 보단 얻는 게 많지 않을까???

\*출처: 코어 자바스크립트(서적)
