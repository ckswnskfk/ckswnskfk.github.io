---
title: "[F-Lab 모각코 챌린지] 63일차"
excerpt: "합성 vs 상속 (in react)"
categories:
  - React
tags:
  - f-lab
  - 에프랩
  - TIL
  - Component
  -
last_modified_at: 2023-07-21
toc: true
---

오늘은 리액트 공식문서(이제는 레거시가 된)에서 소개하는 합성과 상속에 대해 알아보고자 한다. 프론트엔드에서, 리액트에서는 어떻게 적용이 될까?

# 합성 (Composition) vs 상속 (Inheritance)

> React는 강력한 합성 모델을 가지고 있으며, 상속 대신 합성을 사용하여 컴포넌트 간에 코드를 재사용하는 것이 좋습니다.

## 컴포넌트에서 다른 컴포넌트를 담기

항상 자식 컴포넌트로 무엇이 올지 알 수는 없다. 그럴 때 특수한 children prop을 사용하여 자식 엘리먼트를 출력에 그대로 전달하는 것이 좋다. (예: 범용적인 ‘박스’ 역할을 하는 Sidebar 혹은 Dialog와 같은 컴포넌트)

이러한 컴포넌트에서는 특수한 children prop을 사용하여 자식 엘리먼트를 출력에 그대로 전달하는 것이 좋다.

```jsx
function FancyBorder(props) {
  return (
    <div className={"FancyBorder FancyBorder-" + props.color}>
      {props.children}
    </div>
  );
}
```

다른 컴포넌트에서 JSX를 중첩해 임의의 자식을 전달할 수 있다.

```jsx
function WelcomeDialog() {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">Welcome</h1>
      <p className="Dialog-message">Thank you for visiting our spacecraft!</p>
    </FancyBorder>
  );
}
```

예제를 보면 `<FancyBorder>` JSX 태그 안에 있는 것들이 `FancyBorder` 컴포넌트의 children prop으로 전달된다. `FancyBorder`는 `{props.children}`을 `<div>` 안에 렌더링하므로 전달된 엘리먼트들이 최종 출력된다.

그리고 흔하진 않지만 종종 컴포넌트에 외부에서 주입?받을 여러 개의 자리가 필요할 수도 있다. 이런 경우에는 children 대신 자신만의 고유한 방식을 적용할 수도 있다.

```jsx
function SplitPane(props) {
  return (
    <div className="SplitPane">
      <div className="SplitPane-left">{props.left}</div>
      <div className="SplitPane-right">{props.right}</div>
    </div>
  );
}

function App() {
  return <SplitPane left={<Contacts />} right={<Chat />} />;
}
```

`<Contacts />`와 `<Chat />`같은 React 엘리먼트는 객체이기 때문에 다른 데이터처럼 prop으로 전달할 수 있다. 이러한 접근은 다른 라이브러리의 “슬롯 (slots, vue에만 있나?)“과 비슷해보이지만 React에서 prop으로 전달할 수 있는 것에는 제한이 없다.

## 특수화

이번에는 반대로, 어떤 컴포넌트의 '특수한 경우'인 컴포넌트를 고려해야 하는 경우가 있다. 예를 들어, `WelcomeDialog`는 `Dialog`의 특수한 경우라고 할 수 있다.

React에서는 이 역시 합성을 통해 해결할 수 있다. 더 구체적인 컴포넌트가 일반적인 컴포넌트를 렌더링하고 props를 통해 내용을 구성한다.

```jsx
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">{props.title}</h1>
      <p className="Dialog-message">{props.message}</p>
    </FancyBorder>
  );
}

function WelcomeDialog() {
  return (
    <Dialog title="Welcome" message="Thank you for visiting our spacecraft!" />
  );
}
```

클래스로 정의된 컴포넌트에서도 동일하다.

```jsx
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">{props.title}</h1>
      <p className="Dialog-message">{props.message}</p>
      {props.children}
    </FancyBorder>
  );
}

class SignUpDialog extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.handleSignUp = this.handleSignUp.bind(this);
    this.state = { login: "" };
  }

  render() {
    return (
      <Dialog
        title="Mars Exploration Program"
        message="How should we refer to you?"
      >
        <input value={this.state.login} onChange={this.handleChange} />
        <button onClick={this.handleSignUp}>Sign Me Up!</button>
      </Dialog>
    );
  }

  handleChange(e) {
    this.setState({ login: e.target.value });
  }

  handleSignUp() {
    alert(`Welcome aboard, ${this.state.login}!`);
  }
}
```
