---
title: useState Hook으로 테마 변경 기능 구현하기
updated: 2023-11-26 12:00
---

> Elice SW Track 7기 - 11주차 진행 내용입니다.


![Elice Banner](/blog/assets/elice/SW7_top_banner.png)

&nbsp;

앞서 React에서 `Styled-components`를 활용하여 스타일링을 하는 방법을 살펴 보았습니다. 이번에는 `CSS variable`과 `State hook`을 사용한 테마 변경 기능을 구현해보도록 하겠습니다.

> 참고 - Styled-components와 tailwind를 활용한 스타일링 : [링크](https://ji-hoon.github.io/blog/css-styling-in-react){:target="_blank"}




&nbsp;

## 목차

[1. Color scheme 만들기](#1-color-scheme-만들기)

[2. theme state hook 만들기](#2-theme-state-hook-만들기)

[3. 컴포넌트에 적용하기](#3-컴포넌트에-적용하기)



&nbsp;

---

&nbsp;
## 1. Color scheme 만들기

컬러 테마를 관리하는 상태를 변경 시, 현재 설정된 테마 상태에 따라서 다른 색상이 표시될 수 있도록 하기 위해서 CSS variable를 사용해보겠습니다. 

&nbsp;

`CSS variable`은 CSS 파일에 설정하며, `--color-text: {컬러값}` 형태로 선언합니다. 테마 변경 시 `class name`을 기준으로 표시할 색상을 구분할 예정이므로, .`theme-dark` 클래스와 `theme-light` 클래스에는 동일한 `CSS variable`을 사용하지만 색상은 다르게 할당해줍니다.

&nbsp;

`CSS variable`에는 단순 색상 값 뿐만 아니라, `transition` 속성이나 `font-family`등을 할당하여 재사용할 수 도 있습니다.
> index.css

```css
:root {
    --transition-ease-out : all 200ms ease-out;
    --font-family : Inter, system-ui, Avenir, Helvetica, Arial, sans-serif;
}

.theme-dark {
  --color-text: #FFF;
  --color-background: #1e1e1e;
  --color-gray-0 : #bcbcbc;
  --color-gray-1 : #666;
  --color-gray-2 : #3a3a3a;
  --color-gray-3 : #1e1e1e;
  --color-light-gray-0 : #999;
  
  --color-primary: #CFFF48;
  --color-primary-deep: #CFFF48;
  --color-alert: rgb(218, 26, 26);

  --shadow-header: 0 1px 0 1px rgba(255,255,255,0.12);
}

.theme-light {
  --color-text: #1e1e1e;
  --color-background: #FFF;
  --color-gray-0 : #ececec;
  --color-gray-1 : #bcbcbc;
  --color-gray-2 : #3a3a3a;
  --color-gray-3 : #1e1e1e;
  --color-light-gray-0: #f5f5f5;
  
  --color-primary: #bcf717; 
  --color-primary-deep: #a3d90d;
  --color-alert: rgb(218, 26, 26);

  --shadow-header: 0 1px 0 0 rgba(0,0,0,0.12);
}
```


&nbsp;
## 2. theme state hook 만들기

이제 React 파일에서 CSS를 불러와서 적용해보겠습니다. 아래 코드처럼 CSS를 import하고, 사용할 컴포넌트에 `state hook` 을 사용하여 상태를 정의해줍니다. 이 때 기본값으로는 시작 시 사용할 `theme-dark` 클래스를 할당해줍니다.

&nbsp;

테마 변경 버튼 클릭 시 실행될 함수도 선언해주고, `theme props`를 전달 받을 컴포넌트에 전달하여 줍니다. 이 때 `<main>` 엘리먼트에 `className`으로 `theme` 상태를 전달하여 테마가 변경될 수 있도록 하고, `transition` 속성도 추가하여 테마 변경 시 애니메이션 처리를 추가해줍니다.

```javascript
import React, { useState } from 'react';
import TodoItem from "../../TodoItem.tsx";
import Header from "../../Header.tsx";
import './index.css';

export default function Main() {
    const [theme, setTheme] = useState("theme-dark");

    function toggleTheme():void {
        if(theme === "theme-dark") {
            setTheme("theme-light");
        } else {
            setTheme("theme-dark");
        }
    }
    ...
    return (
        <main className={theme}
            style={ {backgroundColor: "var(--color-background)", transition:"var(--transition-ease-out)"} }>
            <Header theme={theme} toggleTheme={toggleTheme} />
            ...
            <TodoItem theme={theme} />
            ...
        </main>
    )

}
```

&nbsp;
## 3. 컴포넌트에 적용하기

마지막으로 `Header` 컴포넌트에 컬러 테마를 변경하는 버튼에 이벤트를 할당하고, 테마 상태에 따라서 아이콘을 다르게 처리하는 기능까지 작성해보도록 하겠습니다.

```javascript
import { FiMoon, FiSun } from "react-icons/fi";

export default function Header({
    theme,
    toggleTheme,}) {
        return (
            <header>
                <div onClick={toggleTheme}>
                    {theme==="theme-dark" && <FiMoon size={24}/>}
                    {theme==="theme-light" && <FiSun size={24}/>}
                </div>
            </header>    
        )
    }
```

위 코드에서 확인할 수 있듯 `theme props`가 `theme-dark`일 때 `FiMoon` 아이콘을 렌더링하고, `theme-light` 라면 `FiSun` 아이콘을 렌더링하게 됩니다. 또한 클릭 시 props로 전달받은 `toggleTheme` 콜백함수를 실행하여 theme state를 변경할 수 있도록 구현할 수 있습니다.

&nbsp;

![color theme result](/blog/assets/posts/asset-theme-switcher.gif)



&nbsp;

---
&nbsp;

[![Elice UTM Banner](/blog/assets/elice/SW7_jihoonkim_bottom_banner.png)](https://elice.training/track/sw?utm_source=sw7&utm_medium=blog&utm_campaign=challenge&utm_content=m2gzitm8b)
&nbsp;
> 태그 Tag : #엘리스트랙 #엘리스트랙후기 #리액트네이티브강좌 #온라인코딩부트캠프 #온라인코딩학원 #프론트엔드학원 #개발자국비지원 #개발자부트캠프 #국비지원부트캠프 #프론트엔드국비지원 #React #Styledcomponent #React Router Dom #Redux #Typescript #Javascript