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

[2. Flux Pattern과 useReducer](#2-flux-pattern과-usereducer)

[3. Redux](#3-redux)

[4. Redux toolkit](#4-redux-toolkit)


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

**Flux pattern**이란 `Dispatcher`, `Stores`, `Views`(React 컴포넌트) 3가지 핵심적인 부분으로 이루어진 소프트웨어 디자인 패턴 중 하나입니다. MVC 패턴에서 하나의 유저 인터렉션 발생 시 그 인터렉션으로 발생한 업데이트가 다른 연쇄 업데이트를 만들어낼 수 있습니다. 따라서 업데이트의 근원을 추적하기 힘든 반면 Flux 패턴은 연쇄 업데이트가 아닌 **단방향 업데이트**만을 만들어낼 수 있습니다.

![flux architecture](https://haruair.github.io/flux/img/flux-simple-f8-diagram-with-client-action-1300w.png){: .align-center}
*Flux architecture*

&nbsp;



&nbsp;

---
&nbsp;

[![Elice UTM Banner](/blog/assets/elice/SW7_jihoonkim_bottom_banner.png)](https://elice.training/track/sw?utm_source=sw7&utm_medium=blog&utm_campaign=challenge&utm_content=m2gzitm8b)
&nbsp;
> 태그 Tag : #엘리스트랙 #엘리스트랙후기 #리액트네이티브강좌 #온라인코딩부트캠프 #온라인코딩학원 #프론트엔드학원 #개발자국비지원 #개발자부트캠프 #국비지원부트캠프 #프론트엔드국비지원 #React #Styledcomponent #React Router Dom #Redux #Typescript #Javascript