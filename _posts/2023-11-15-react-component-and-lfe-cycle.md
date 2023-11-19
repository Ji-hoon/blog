---
title: React 컴포넌트(Component)와 생명주기(Life Cycle)
updated: 2023-11-15 12:00
---

> Elice SW Track 7기 - 10주차 진행 내용입니다.


![Elice Banner](/blog/assets/elice/SW7_top_banner.png)

&nbsp;

React는 컴포넌트(Component) 단위의 개발 방식을 통해 재사용성, 효율성 그리고 확장성 높은 개발을 할 수 있도록 도와주는 라이브러리 입니다. React를 일반적인 자바스크립트 프로젝트에서 import하여 사용할 수도 있고, CRA(Create React App)이나 Vite, Svelte 같은 빌드 툴을 통해서 React 프로젝트를 구성하여 시작할 수 있습니다. (빌드 툴에 대해서는 다음에 자세히 다루도록 하겠습니다.)

> 참고 - React로 사고하기 : [링크](https://ko.legacy.reactjs.org/docs/thinking-in-react.html)

이번에는 React를 이해하기 위한 핵심 개념인 컴포넌트와 컴포넌트의 생명주기(Life Cycle)에 대해 다뤄보겠습니다.


&nbsp;

## 목차
[1. 컴포넌트란?](#1-컴포넌트란)

[2. React 컴포넌트의 종류](#2-react-컴포넌트의-종류)

[3. 컴포넌트의 생명주기와 메서드](#3-컴포넌트의-생명주기와-메서드)

[4. 함수형 컴포넌트에서의 생명주기 관리](#4-함수형-컴포넌트에서의-생명주기-관리)



&nbsp;

---

&nbsp;
## 1. 컴포넌트란?

컴포넌트란, 코드 레벨에서 중복해서 사용되는 기능을 하나의 모듈로써 구성하고 재사용이 가능하게끔 작성된 코드 묶음을 의미합니다. 컴포넌트에서는 UI 요소를 구성하고 상태를 관리할 수 있으며, 다른 컴포넌트와 조합하여 복잡한 UI를 구축할 수 있습니다. 

&nbsp;

물론 React가 나오기 이전에도 컴포넌트라는 개념은 이미 존재했지만, React에서 제공하는 가상 DOM, Props, 그리고 State라는 개념들을 통해 동적으로 변화하는 정보들을 제공하는 애플리케이션을 더욱 효율적으로 개발을 할 수 있게 되었습니다.

&nbsp;
## 2. React 컴포넌트의 종류

현재 React 18.2 버전 기준으로, 이전 버전부터 사용되어 왔던 클래스(Class)형 컴포넌트와 함수(Function)형 컴포넌트, 2가지 방식이 사용되고 있습니다. 2가지 방식은 아래와 같은 차이점이 있습니다.

> 구현 방식

클래스형 컴포넌트는 ES6의 클래스 문법을 사용하여 컴포넌트를 정의하게 됩니다. `render()` 메서드를 통해 JSX를 반환하고, `this.state` 명령어로 컴포넌트의 상태(state)를 관리할 수 있습니다. 반면 함수형 컴포넌트는 일반적인 JavaScript 함수로 컴포넌트를 정의합니다. 마찬가지로 JSX를 반환하며, 상태를 관리하기 위해 `useState()` 훅(Hook)을 사용할 수 있습니다.

> 코드 길이와 가독성

클래스형 컴포넌트는 컴포넌트의 **생명주기** 메소드를 구현해야 할 때 클래스의 구조로 인해 코드가 더욱 복잡해질 수 있습니다. 반면 함수형 컴포넌트는  [**Hooks**](https://ko.legacy.reactjs.org/docs/hooks-intro.html)를 사용하면 컴포넌트의 상태와 생명 주기와 관련된 로직을 함수 내에서 처리할 수 있어 코드의 구조가 단순해집니다.

> 컴포넌트의 성능

클래스형 컴포넌트는 불필요한 렌더링을 방지하기 위해 성능에 영향을 줄 수 있는 `shouldComponentUpdate()` 메서드를 사용해야 합니다. 하지만 함수형 컴포넌트는 `React.useMemo()` 를 사용하여 불필요한 렌더링을 방지하고 더 효율적인 렌더링을 할 수 있어 성능이 향상될 수 있습니다.

&nbsp;

최근에는 React 16.8 버전부터 등장한 Hook 개념을 사용할 수 있는 함수형 컴포넌트가 권장되고 있으며, 컴포넌트 간 로직 재사용을 위한 `custom hooks` 기능을 통해 클래스형 컴포넌트의 단점을 보완할 수 있습니다. 하지만 Hook을 사용하게 된 배경을 제대로 이해하려면 클래스형 컴포넌트에서 생명주기를 어떻게 다루는지 이해하는 것도 중요하기 때문에, 클래스형 컴포넌트에서 생명주기를 어떻게 다루는 방식에 대해 먼저 살펴보겠습니다.

&nbsp;
## 3. 컴포넌트의 생명주기와 메서드

앞에서 잠깐 언급했듯이 컴포넌트를 제대로 사용하려면 생명주기라는 개념을 이해해야 합니다. 생명주기란, 앱이 실행되고 종료되는 과정을 특정 시점 별로 나눠둔 것을 의미합니다. React의 생명주기는 컴포넌트가 이벤트를 다룰 수 있는 특정 시점을 말하며, ① 가상 DOM이 아닌 실제 DOM에 컴포넌트가 삽입되는 시점을 **마운트** mount, ② 컴포넌트의 정보가 변화하여 리렌더링(Re-rendering) 하는 시점을 **업데이트** update, ③ 마지막으로 실제 DOM에서 컴포넌트가 제거되는 시점을 **언마운트** unmount 라고 합니다. 

&nbsp;

아래 메소드들은 **클래스형 컴포넌트**에서 사용되는 대표적인 생명주기 메서드입니다.

1. **`constructor()`**: **`State`** 데이터를 초기화 하는 메서드
2. **`render()`**: 클래스 컴포넌트에서 반드시 구현되어야 하는 메서드
3. **`componentDidMount()`**: 컴포넌트가 마운트 된 직후 호출되는 메서드 (**마운트**)
4. **`componentDidUpdate()`**: 업데이트가 진행된 직후에 호출되는 메서드 (**업데이트**)
5. **`componentWillUnmount()`**: 컴포넌트가 마운트 해제되어 제거되기 직전에 호출되는 메서드 (**언마운트**)

&nbsp;

위 메서드들을 이용해서 간단한 예시 코드를 작성해보았습니다. App 컴포넌트 하위에 CustomComponent를 mount 하고, Remove 버튼으로 Component를 unmount 하는 동작을 수행하는 예시 입니다. 

```jsx
import React from 'react';
import ReactDOM from 'react-dom/client'	

class CustomComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { color: "Red" };
  }
  componentDidMount() {
    console.log("didMount: ", this.state.color);
  }
  componentDidUpdate() {
    console.log("didUpdate: ", this.state.color);
  }
  componentWillUnmount() {
    console.log("unMounted");
  }
  render() {
      return (
        <>
          <h1>My Favorite Color is 
            <em> {this.state.color}</em>
          </h1>
          <button onClick={() => {this.setState({color: "Orange"});
            }}>Orange</button>
          <button onClick={() => {this.setState({color: "Red"});
            }}>Red</button>
        </>
      )
  }
}

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {visible: true};
  }
  render() {
    return (
      <div>
        {this.state.visible && <CustomComponent /> }
        <button onClick={ () => { this.setState({visible: !this.state.visible }); }}>
          {this.state.visible? "Remove" : "Mount"}
        </button>
      </div>
    );
  }
}
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);
```
위 코드를 실행하면 App이 렌더링되며 컴포넌트가 DOM에 mount될 때, 버튼의 클릭 이벤트를 통해 컴포넌트의 상태가 update될 때, 그리고 컴포넌트가 DOM에서 제거될 때 unmount될 때 로그가 찍히는 것을 확인할 수 있습니다.

![React console](/blog/assets/posts/asset-react-component-lifecycle.gif)


&nbsp;
## 4. 함수형 컴포넌트에서의 생명주기 관리

위에서 클래스형 컴포넌트에서 생명주기를 관리하는 메서드들을 살펴봤습니다. 생명주기 별로 독립된 메서드를 호출하는 형태로 사용되던 것과 비교하여, 함수형 컴포넌트에서 생명주기 메서드는 `useEffect` 훅 하나로 생명주기를 관리할 수 있습니다. 위에서 클래스형 컴포넌트로 작성했던 기능과 동일한 동작을 수행하는 함수형 컴포넌트 예시 코드를 작성해 보겠습니다.

```jsx
import React, { useState, useEffect, useRef } from 'react';
import ReactDOM from 'react-dom/client'	

function CustomComponent() {
  const [color, setColor] = useState("Red");
  const isFirstRender = useRef(true);

  // didMount와 unMount시점에 실행
  useEffect( () => {
    console.log("didMount: ", color);

    return( () => {
      console.log("unMounted!");
    });
  }, []); // 빈 배열을 넣을 경우 컴포넌트의 마운트/언마운트 시점에 각각 1번만 실행

  // didUpdate 시점에 실행
  useEffect( () => {
    if (isFirstRender.current) {
      // 컴포넌트가 처음 마운트될 때에는 실행되지 않도록 분기 처리
      isFirstRender.current = false;
    } else {
      console.log("didUpdate: ", color);
    }
  }); // 배열을 넣지 않을 경우 업데이트 시점마다 실행
  
  return (
    <>
      <h1>My Favorite Color is <em>{color}</em></h1>
      <button onClick={() => {setColor("Orange");}}>Orange</button>
      <button onClick={() => {setColor("Red");}}>Red</button>
    </>
  )
}

function App() {
  const [visible, setVisible] = useState(true);
  
  return (
      <div>
        {visible && <CustomComponent /> }
        <button onClick={ () => { setVisible(!visible ); }}>
          {visible ? "Remove" : "Mount"}
        </button>
      </div>
  );
}
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);
```
첫 번째 useEffect 훅에는 빈 배열을 할당하여 didMount와 unMount 생명주기에 대응하는 코드를 작성하고, 두 번째 useEffect 훅에는 update 시 실행할 코드를 작성했습니다. 함수형 컴포넌트에서는 비교적 간단하게 생명주기를 관리할 수 있습니다.


&nbsp;

---
&nbsp;

[![Elice UTM Banner](/blog/assets/elice/SW7_jihoonkim_bottom_banner.png)](https://elice.training/track/sw?utm_source=sw7&utm_medium=blog&utm_campaign=challenge&utm_content=m2gzitm8b)
&nbsp;
> 태그 Tag : #엘리스트랙 #엘리스트랙후기 #리액트네이티브강좌 #온라인코딩부트캠프 #온라인코딩학원 #프론트엔드학원 #개발자국비지원 #개발자부트캠프 #국비지원부트캠프 #프론트엔드국비지원 #React #Styledcomponent #React Router Dom #Redux #Typescript #Javascript