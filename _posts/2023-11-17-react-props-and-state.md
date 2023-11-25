---
title: Props와 State 이해하기
updated: 2023-11-17 12:00
---

> Elice SW Track 7기 - 10주차 진행 내용입니다.


![Elice Banner](/blog/assets/elice/SW7_top_banner.png)

&nbsp;

React를 이해하는데에 중요한 개념이 바로 `Props`와 `State`, 그리고 `Hooks` 라고 이전 포스팅에서 소개했었습니다. 이번에는 `Props`와 `State`의 개념과 컴포넌트에 어떻게 적용되는 지, 그리고 사용할 때 유의할 점에 대해서 다루도록 하겠습니다.

> 참고 - 컴포넌트와 Props : [링크](https://ko.legacy.reactjs.org/docs/components-and-props.html) |  State와 생명주기 : [링크](https://ko.legacy.reactjs.org/docs/state-and-lifecycle.html)




&nbsp;

## 목차
[1. Props vs State](#1-props-vs-state)

[2. 컴포넌트에서 적용하기](#2-컴포넌트에서-적용하기)

[3. State 변경과 리렌더링 관리](#3-state-변경과-리렌더링-관리)



&nbsp;

---

&nbsp;
## 1. Props vs State

먼저 `Props`와 `State`의 개념에 대해서 알아보겠습니다. `props`는 컴포넌트에 전달하는 속성인 반면, `State`는 현재 컴포넌트 레벨에서만 관리하는 속성입니다. 

&nbsp;

`Props`는 하위 컴포넌트로 전달하여 사용할 수 있는 속성이기 때문에 상위 엘리먼트에서 컴포넌트 호출 시 attribute 형태로 작성하고 값을 전달합니다. `Props`로 전달할 수 있는 타입에는 제한이 없으므로 문자열(String), 숫자(Number), 불린(Boolean), 배열(Array), 객체(Object), 그리고 함수(Function)까지 전달할 수 있습니다. (**콜백 함수** 까지 포함하며, Props는 하위 컴포넌트에서 그 값을 변경할 수 없습니다)

&nbsp;

`State`는 컴포넌트 단위로 관리할 수 있는 속성으로, `State`가 업데이트되면 컴포넌트를 다시 렌더링하게 됩니다. 이를 `리렌더링(Re-rendering)` 이라고 표현합니다. 따라서 `State`를 어디에 정의하느냐에 따라서 애플리케이션의 성능에 영향을 미칠 수 있습니다.

&nbsp;
## 2. 컴포넌트에서 적용하기

그럼 컴포넌트 타입마다 props를 다루는 아래 예시 코드를 통해 살펴보겠습니다.


> index.tsx

```javascript
import React from 'react';
import { useState } from 'react';
import InputElement from './Input.jsx';
import ReactDOM from 'react-dom/client'

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <InputElement name={"nickname"} placeholder={"내용을 입력하세요."} />
  </React.StrictMode>
)
```

위 코드에서는 `Input` 컴포넌트를 불러와서, `name`과 `placeholder`라는 2개의 `Props`를 해당 컴포넌트로 넘겨주고 있습니다.

&nbsp;

이 때 클래스형 컴포넌트와 함수형 컴포넌트에서 `Props`를 처리하는 코드가 상이한데, 먼저 클래스형 컴포넌트의 경우 예시 코드를 작성해보겠습니다.

> Input.tsx

```javascript
import React, { Component } from "react";

class InputElement extends Component {
  render() {
    // props에서 값을 구조분해하여 변수에 할당
    const { name, placeholder } = this.props;

    return (
      <input className={name} 
        placeholder={placeholder} />
    );
  }
}

export default InputElement;
)

```

`this.props`로 `Props`들을 변수로 할당하고, 그 값을 `className`, 그리고 `placeholder`에 할당하고 있습니다.

&nbsp;

다음 예시 코드는 함수형 컴포넌트에서 `Props`를 처리하는 코드입니다.

> Input.tsx

```javascript
import React from "react";

export default function InputElement({name, placeholder}) {
  
  return (
    <input className={name} 
        placeholder={placeholder} />
  )
}
```

함수형 컴포넌트에서는 별도의 `Props`를 정의하지 않고도 컴포넌트로 전달된 `Props`객체들을 바로 사용할 수 있습니다.


&nbsp;
## 3. State 변경과 리렌더링 관리

앞서 `State`가 변경되면 `State`가 정의된 컴포넌트 단위로 리렌더링이 발생한다고 했습니다. 다음은 InputElement에 입력된 내용이 변경될 때 마다 리렌더링하게되는 예시 코드 입니다.

```javascript
import React, { useState } from "react";

export default function InputElement({name, placeholder}) {
  const [username, setUsername] = useState("");
  console.log(username);
  return (
    <input className={name} 
           placeholder={placeholder}
           value={username} 
           onChange={(event) => {
                setUsername(event.target.value);
                 }}/>
  )
}
```

위 컴포넌트에서는 Input의 입력 글자에 변경이 일어날 때, setUsername 메서드를 호출하여 username `State`를 변경하게 되므로, 입력된 글자에 변경이 일어날 때 마다 리렌더링을 하게 됨을 알 수 있습니다.

![state console log](/blog/assets/elice/asset-props-and-state.gif)






&nbsp;

---
&nbsp;

[![Elice UTM Banner](/blog/assets/elice/SW7_jihoonkim_bottom_banner.png)](https://elice.training/track/sw?utm_source=sw7&utm_medium=blog&utm_campaign=challenge&utm_content=m2gzitm8b)
&nbsp;
> 태그 Tag : #엘리스트랙 #엘리스트랙후기 #리액트네이티브강좌 #온라인코딩부트캠프 #온라인코딩학원 #프론트엔드학원 #개발자국비지원 #개발자부트캠프 #국비지원부트캠프 #프론트엔드국비지원 #React #Styledcomponent #React Router Dom #Redux #Typescript #Javascript