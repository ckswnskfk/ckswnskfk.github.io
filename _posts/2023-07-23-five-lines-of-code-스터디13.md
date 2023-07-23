---
title: "[F-Lab 모각코 챌린지] 65일차"
excerpt: "five lines of code - 13"
categories:
  - Refactoring
tags:
  - f-lab
  - 에프랩
  - TIL
  - Typescript
  - Five lines of code
last_modified_at: 2023-07-23
toc: true
---

다시 파이브 라인스 오브 코드를 이어 가보자.

# 데이터 보호

이 전에 클래스에 동일한 데이터에 대한 기능을 결합해 불변속성을 더 가깝게 모아 지역화했다. 이번 장은 데이터와 기능에 대한 접근을 제한하는 캡슐화에 초점을 맞춰 불변속성이 지역에만 영향을 주게 만드는 데 중점을 준다.

## getter 없이 캡슐화

### 규칙: getter와 setter를 사용하지 말 것

#### 정의

부울(boolean)이 아닌 필드에 setter나 getter를 사용하지 말 것.

#### 설명

여기서 말하는 getter와 setter는 정의에서 설명한 것처럼 부울이 아닌 필드를 직접 할당하거나 반환하는 메서드를 말한다.

getter와 setter는 흔히 private 필드를 다루기 위한 메서드로 캡슐화와 함께 배운다. 그러나 객체의 필드에 대한 getter가 존재하는 순간 캡슐화를 해제하고 불변속성을 전역적으로 만들게 된다. 객체를 반환받은 곳에서는 이 객체를 더 많은 곳에 전달할 수 있고, 우리는 이것을 모두 통제하지 못한다.

setter도 비슷한 문제가 있다. 이론적으로 setter는 내부 데이터 구조를 변경하고 해당 setter를 수정해도 시그니처를 유지할 수 있는 또 다른 간접적인 레이어를 도입할 수 있다. 이런 메서드는 더 이상 setter가 아니므로 문제가 되지 않지만, 실제로 일어나는 일은 setter를 통한 새 데이터 구조를 반환하도록 getter를 수정하는 것이다. 그런 다음 수신자 측에서 이 새로운 데이터 구조를 받을 수 있도록 수정해야 한다. 이것은 우리가 피해야 할 tight coupling의 형태이다.

이런 문제는 가변 객체에서만 발생한다. 그러나 이 규칙은 불변 필드에도 비공개 필드를 적용해 얻는 또 다른 효과, 즉 제안하는 아키텍처 때문에 부울만을 예외로 하고 있다. 필드를 비공개로 하는 것의 가장 큰 장점은 그렇게 하는 것이 푸시 기반의 아키텍처를 장려하기 때문이다. 푸시 기반 아키텍처에서는 가능한 한 데이터에 가깝게 연산을 이관하지만, 풀 기반의 아키텍처에서는 데이터를 가져와 중앙에서 연산을 수행한다.

풀 기반의 아키텍처는 기능을 수행하는 메서드 없이 수많은 수동적인 데이터 클래스와 여기저기서 데이터를 혼합해서 모든 작업을 수행하는 소수의 관리자 클래스로 이어진다. 이 방식은 데이터와 관리자 사이, 그리고 암묵적으로 데이터 클래스 간에도 tight coupling을 가져온다.

푸시 기반 아키텍처에서는 데이터를 가져오는 대신 인자로 데이터를 전달한다. 결과적으로 모든 클래스가 자신의 기능을 가지고 있으며 코드는 그 효용에 따라 분산된다. 이번 장의 예제에서는 블로그 게시물에 대한 링크를 생성한다. 양쪽 모두 동일한 작업을 수행하지만, 하나는 풀 기반 아키텍처로 작성되고 다른 하나는 푸시 기반 아키텍처로 작성된다.

```typescript
// 풀 기반 아키텍처
class WebSite {
  constructor(private url: string) {}
  getUrl() {
    return this.url;
  }
}

class User {
  constructor(private username: string) {}
  getUsername() {
    return this.username;
  }
}

class BlogPost {
  constructor(private author: User, private id: string) {}
  getId() {
    return this.id;
  }
  getAuthor() {
    return this.author;
  }
}

function generatePostLink(website: WebSite, post: BlogPost) {
  let url = website.getUrl();
  let user = post.getAuthor();
  let name = user.getUsername();
  let postId = post.getId();
  return url + name + postId;
}
```

```typescript
// 푸시 기반 아키텍처
class WebSite {
  constructor(private url: string) {}
  generateLink(name: string, id: string) {
    return this.url + name + id;
  }
}

class User {
  constructor(private username: string) {}
  generateLink(website: WebSite, id: string) {
    return website.generateLink(this.username, id);
  }
}

class BlogPost {
  constructor(private author: User, private id: string) {}
  generateLink(website: WebSite) {
    return this.author.generateLink(website, this.id);
  }
}

function generatePostLink(website: WebSite, post: BlogPost) {
  return post.generateLink(website);
}
```

#### 스멜

이 규칙은 흔히 '낯선 사람에게 말하지 말라'로 요약되는 디미터 법칙에서 유래했다. 여기서 낯선 사람이란 우리가 직접 접근할 수는 없지만 참조를 얻을 수 있는 객체이다.

#### 의도

참조를 얻을 수 있는 객체와 상호작용할 때의 문제는 객체를 가져오는 방식과 tight coupling 된다는 것이다. 우리는 객체 소유자의 내부 구조에 대해 어느 정도 알고 있어야 한다. 필드 소유자는 이전 데이터 구조를 획득하는 방법을 계속 지원하지 않는 한 데이터 구조를 변경할 수 없다.

푸시 기반 아키텍처에서는 서비스와 같은 메서드를 노출한다. 이런 메서드의 사용자는 사용하는 내부 구조에 대해 신경 쓰지 않아도 된다.
