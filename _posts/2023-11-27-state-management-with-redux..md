---
title: Redux로 전역 상태 관리하기
updated: 2023-11-27 12:00
---

> Elice SW Track 7기 - 12주차 진행 내용입니다.


![Elice Banner](/blog/assets/elice/SW7_top_banner.png)

&nbsp;

React에서 상태를 관리하는 `useState hook` 같은 경우, 최상위 컴포넌트에서 정의된 상태를 하위 컴포넌트에서 사용하기 위해서는 `Props`로 필요한 상태를 넘겨주어야 합니다. 하지만 중첩되는 컴포넌트가 많아지고, 관리해야 하는 상태가 많아지게 되면 전역 상태를 관리하기 위한 해결책이 필요합니다. 이번에는 전역 상태 관리 라이브러리 중에서 `Redux`에 대해서 알아보겠습니다.

> 참고 - React 상태 관리 기술 : [링크](https://ui.toast.com/posts/ko_20210707){:target="_blank"}




&nbsp;

## 목차

[1. 전역 상태 관리가 필요한 이유](#1-전역-상태-관리가-필요한-이유)

[2. Flux Pattern과 useReducer hook](#2-flux-pattern과-usereducer-hook)

[3. Redux](#3-redux)

[4. Redux toolkit을 활용한 전역 상태 관리](#4-redux-toolkit을-활용한-전역-상태-관리)


&nbsp;

---

&nbsp;
## 1. 전역 상태 관리가 필요한 이유

> Props drilling

컴포넌트 구조가 복잡해지는 경우 부모 자식 컴포넌트간 깊이가 커지게 됩니다. 이 때 전역 상태 관리 기술을 사용하지 않고 `Props`를 통해 상태를 넘겨준다고 하면, 최하단의 자식 컴포넌트가 데이터를 쓰기위해 최상단 컴포넌트부터 데이터를 보내야 하는 상황이 발생합니다. 이런 경우, 사용하지 않는 컴포넌트에도 `Props`를 전달해줘야 하는 `Props drilling` 문제가 발생하게 됩니다.

> 데이터 캐싱과 재활용

기본적으로 하나의 HTML을 사용하는 SPA형태의 앱에서 페이지 로딩 시 마다 모든 데이터를 API요청을 통해 갱신하게 된다면 MPA(Multi Page Application)와 비교해서 속도면에서 이점이 없게 될 것입니다. 따라서 상태에 따라서 매번 새로운 데이터를 갱신해야 하는 상태, 정기적으로 갱신해야 하는 상태, 한 번 받아온 데이터를 캐시해서 활용해도 되는 상태 등을 나누는 최적화가 필요하게 됩니다. 이 때 상태 관리 기술을 사용하면 위와 같은 문제를 해결 할 수 있습니다.

&nbsp;
## 2. Flux Pattern과 useReducer Hook

React에서는 기본적으로 이러한 상태 관리를 위한 `useReducer hook`을 제공하고 있습니다. 별도의 라이브러리 설치를 하지 않아도 **Flux pattern**에 기반한 상태 관리를 구현할 수 있습니다.

> Flux pattern

**Flux pattern**이란 `Dispatcher`, `Stores`, `Views`(React 컴포넌트) 3가지 핵심적인 부분으로 이루어진 소프트웨어 디자인 패턴 중 하나입니다. **MVC 패턴**의 경우, 하나의 유저 인터렉션 발생 시 그 인터렉션으로 발생한 업데이트가 다른 연쇄 업데이트를 만들어낼 수 있기 때문에 업데이트의 근원을 추적하기 힘든 반면, **Flux 패턴**은 연쇄 업데이트가 아닌 **단방향 업데이트**만을 만들어낼 수 있습니다.

&nbsp;

![flux architecture](https://haruair.github.io/flux/img/flux-simple-f8-diagram-with-client-action-1300w.png)
*Flux 패턴의 데이터 흐름 ([출처](https://haruair.github.io/flux/docs/overview.html){:target="_blank"})*{: .caption}

&nbsp;

위 이미지에서 보이는 것 처럼 `React View` 에서 사용자의 인터렉션이 발생했을 때, 그 `view`는 중앙의 `dispatcher`를 통해 `action`을 전파하게 됩니다. 어플리케이션의 데이터와 비지니스 로직을 가지고 있는 `store`는 `action`이 전파되면 이 `action`에 영향이 있는 모든 `view`를 갱신하게 됩니다.

> useReducer Hook

`useRedcuer Hook`이 **Flux pattern**에 따라서 상태 관리를 할 수 있도록 구현된 내장 Hook 입니다. 예시 코드를 통해 살펴보겠습니다.

```javascript
/*
 * App.js
 */
import React from 'react';
import Counter from './Counter.js';

// 초기 상태 값을 객체 형태로 선언
export const initialState = {
  number: 0,
};

// state와 action 을 인자로 받는 reducer 함수를 작성
export function reducer(state, action) {
	switch (action.type) {
		case 'increase': {
			return {
			  ...state,
              number: state.number + 1,
      }
		}
		case 'decrease': {
			return {
			  ...state,
              number: state.number - 1,
      }
		}
		default: {
			return state;
		}
	}
}

function App() {
    return (
        <div className="App">
          <Counter />
        </div>
    )
}

/*
 * Counter.js
 */
import React, { useReducer } from 'react';
import { initialState, reducer } from './App.js';

export default function Counter() {
    // useReducer 메소드를 사용해서 state와 dispatch를 정의
    const [state, dispatch] = useReducer(reducer, initialState); // initialState : { number: 0 }

    // 상태 변경이 필요한 경우 dispatch 메소드를 호출
    function handleIncreae() {
        dispatch({ type: 'increase' });
    }
    function handleDecrease() {
        dispatch({ type: 'decrease' });
    }
    return(
        <div>
            <p>{state.number}</p>
            <button onClick={handleIncreae}>+</button>
            &nbsp;
            <button onClick={handleDecrease}>-</button>
        </div>
    )
}
```
`useReducer` 메소드의 인자로 `reducer`와 `initialState`를 통해서 `state`와 `dispatch`를 선언합니다. `view`에서 상태를 변경하려고 할 때 `dispatch` 메소드에 `type`과 `payload`를 담아서 호출하고, `store`에 담긴 상태가 변경되고 관련된 모든 뷰가 갱신됩니다.

&nbsp;

![useReducer를 사용한 상태 관리](/blog/assets/posts/asset-reducer-example.gif)



&nbsp;
## 3. Redux

앞에서 `useReducer hook`을 사용한 상태 관리 방식에 대해 살펴보았습니다. 하지만 비동기 처리나 복잡한 상태를 관리하기 위해서는 `useReducer` 만으로는 어렵기 때문에, 이번에는 `Redux`에 대해서 알아보도록 하겠습니다.

> Redux - [링크](https://redux.js.org/https://redux.js.org/){:target="_blank"}

`Redux`는 useReducer와 마찬가지로 많은 개념들이 `Flux pattern` 차용된 상태 관리를 위한 라이브러리로, 복잡한 비동기 처리나 logger 혹은 devtool을 활용한 상태 관리를 하기 위해 사용됩니다.

> Redux의 핵심 원칙 - [링크](https://redux.js.org/understanding/thinking-in-redux/three-principles){:target="_blank"}

1. 단일 정보 소스 - store는 1개이며, 모든 앱의 상태가 이곳에 보관됩니다.
2. 불변성 - 상태(state)는 오로지 읽을 수만 있습니다(read-only). dispatch action을 통해 새로운 상태를 생성합니다.
3. 순수 함수 - 상태 변경은 어떠한 사이드 이펙트도 만들지 않아야 합니다.

> action

**action**은 상태의 변경을 나타내는 개념 (하나의 동작만을 수행)으로 `type`과 `payload`를 포함하는 JS 객체 입니다.

> action creator

**action**이 1개의 액션만을 생성한다면, **action creator**는 여러개의 액션을 생성하는 함수 입니다. 직접 action을 생성하는 것 보다 재사용성이 높아집니다.

> store

**store** 는 앱 전체의 상태를 보관하는 곳입니다. **action**에 따라서 **reducer**에서는 새로운 상태를 반환하며, 다시 **store**에 그 결과 값을 저장합니다.

> reducer

**action** 인자로 받아서 새로운 **state** 객체를 만들어 반환합니다. 상태 변경 시, 사이드 이펙트가 발생하지 않도록 함수를 설계해야 합니다.

> dispatch

생성된 **action** 을 **reducer** 로 보내는 함수입니다. 이 때 필요한 미들웨어가 있다면 **reducer**에 도달하기 전에 사용합니다.

&nbsp;
## 4. Redux toolkit을 활용한 전역 상태 관리

마지막으로 Redux와 Redux toolkit을 활용한 전역 상태 관리 방법에 대해서 살펴보겠습니다.


&nbsp;

---
&nbsp;

[![Elice UTM Banner](/blog/assets/elice/SW7_jihoonkim_bottom_banner.png)](https://elice.training/track/sw?utm_source=sw7&utm_medium=blog&utm_campaign=challenge&utm_content=m2gzitm8b)
&nbsp;
> 태그 Tag : #엘리스트랙 #엘리스트랙후기 #리액트네이티브강좌 #온라인코딩부트캠프 #온라인코딩학원 #프론트엔드학원 #개발자국비지원 #개발자부트캠프 #국비지원부트캠프 #프론트엔드국비지원 #React #Styledcomponent #React Router Dom #Redux #Typescript #Javascript