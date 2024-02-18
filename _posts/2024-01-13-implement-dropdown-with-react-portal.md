---
title: React portal을 활용한 드롭다운 컴포넌트 구현하기 
updated: 2024-01-13 12:00
---

> '가계부를 부탁해' 프로젝트를 진행하며 작성한 내용입니다.

[![project-logo](https://github.com/Ji-hoon/Home-accountant/raw/master/client/public/img-logo.png)](https://github.com/Ji-hoon/Home-accountant)

&nbsp;

새롭게 개인 프로젝트를 시작하면서, 처음 사용해보았던 기술들에 대해서 기록을 남겨보려 합니다. 오늘은 첫 번째 주제로, 웹 앱에서 자주 사용되는 `드롭다운(Dropdown)` 컴포넌트를 `React portal`을 사용해서 효율적으로 구현해보도록 하겠습니다.

> 프로젝트 보러가기 : [링크](http://35.231.16.39/)

&nbsp;

## 목차

[1. React portal을 왜 써야할까?](#1-react-portal을-왜-써야할까)

[2. Portal로 사용할 dom 설정하기](#2-portal로-사용할-dom-설정하기)

[3. Portal 바깥에 dropdown trigger 설정하기](#3-portal-바깥에-dropdown-trigger-설정하기)

[4. 컴포넌트의 표시 위치 계산하기](#4-컴포넌트의-표시-위치-계산하기)

[5. 드롭다운 고유 키를 사용한 분기 처리](#5-드롭다운-고유-키를-사용한-분기-처리)

&nbsp;

---

&nbsp;

## 1. React portal을 왜 써야할까?

![dropdown UI pattern](/assets/posts/asset-dropdown-pattern.png)*다양한 Dropdown 패턴 feat. Notion, Rallit, Gmail*{: .caption}

먼저 웹 애플리케이션에서 드롭다운 패턴이라함은, UI 관점에서 바라보면 **평소에는 보여지지 않지만 특정 액션에 대한 선택지를 제공하거나 부가적인 정보나 기능을 제공하려고 할 때** 사용하게 됩니다. 예컨데 **옵션 메뉴 버튼을 클릭했을 때 선택 가능한 옵션 목록을 제공하는 UI** 나 **더보기 같은 버튼을 클릭했을 때 리소스를 추가하거나 변경하는 기능들을 제공하는 UI패턴**을 의미합니다.

&nbsp;

드롭다운 패턴에는 UX 관점에서 중요한 특징이 있는데 ① UI를 호출하기 위한 1개 이상의 **트리거** 요소가 필요하며, ② 트리거 요소를 클릭했을 때 표시되는 **드롭다운 UI 바깥 영역을 클릭 또는 포커스** 되었다면 UI는 사라져야 한다는 점입니다. 개발 관점에서는 ③ 동일한 드롭다운 컴포넌트가 여러 곳에서 반복적으로 사용될 수 있다는 특징이 있습니다.

> 참고 - Dropdown pattern : [링크](https://semantic-ui.com/modules/dropdown.html)

이러한 특징 때문에 드롭다운 컴포넌트를 구현한다면 아래 2가지 제약사항을 고려해야 합니다.

1. UI를 숨기는 이벤트 핸들링을 위해 드롭다운의 z-index는 기본 UI (트리거 요소 포함) 보다 상위에 위치해야 합니다.
2. 드롭다운 UI를 렌더링 할 때 불필요한 리렌더링이 일어나지 않도록 관련 없는 컴포넌트에 영향을 주지 않아야 합니다.

React 16 버전에 추가된 `react portal` 을 사용하면 위 제약사항을 만족시키면서 드롭다운 UI를 구현할 수 있습니다.

> 참고 - React Portals 가이드 : [링크](https://reactjs-kr.firebaseapp.com/docs/portals.html) / createPortal 사용 가이드 : [링크](https://react.dev/reference/react-dom/createPortal)

가이드에서는 portal에 대한 설명을 다음과 같이 안내합니다. 

```
createPortal lets you render some children into a different part of the DOM.
```

React 프로젝트 생성 시 제공되는 `index.html`에는 `root`라는 `id`를 가진 `div` 하위에 컴포넌트들을 그리게 되는데요. portal로 사용할 별도의 element를 만들고, 해당 위치에 렌더링할 컴포넌트를 보내는 기능을 제공합니다.

&nbsp;

## 2. Portal로 사용할 dom 설정하기

2번 주제

&nbsp;

## 3. Portal 바깥에 dropdown trigger 설정하기

3번 주제

&nbsp;

## 4. 컴포넌트의 표시 위치 계산하기

4번 주제

&nbsp;

## 5. 드롭다운 고유 키를 사용한 분기 처리

5번 주제