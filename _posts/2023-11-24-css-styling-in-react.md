---
title: Styled-components와 tailwind를 활용한 스타일링
updated: 2023-11-24 12:00
---

> Elice SW Track 7기 - 11주차 진행 내용입니다.


![Elice Banner](/blog/assets/elice/SW7_top_banner.png)

&nbsp;

React에서는 다양한 CSS 스타일링 방식을 사용할 수 있습니다. VanillaJS 처럼 클래스명을 할당한 뒤 별도 CSS 파일을 만들어서 스타일링 하거나, return 구문안에 직접 작성하는 방법, 그리고 Styled-components를 활용하는 방법, 마지막으로 Tailwind 같은 외부 라이브러리를 활용하는 방법이 있습니다.

> 참고 - Styled-components : [링크](https://styled-components.com){:target="_blank"} |  Tailwind css with Vite : [링크](https://tailwindcss.com/docs/guides/vite){:target="_blank"}




&nbsp;

## 목차

[1. className을 활용한 CSS 스타일링](#1-classname을-활용한-css-스타일링)

[2. Inline CSS 스타일링](#2-inline-css-스타일링)

[3. CSS-in-JS (Styled-component)](#3-css-in-js-styled-component)

[4. Tailwind를 활용한 스타일링](#4-tailwind를-활용한-스타일링)



&nbsp;

---

&nbsp;
## 1. className을 활용한 CSS 스타일링

React에서도 외부 CSS 파일을 불러와서 id 또는 className에 스타일을 선언하여 사용할 수 있습니다. 아래 예시 코드처럼 js파일 내에서 `return` 구문 안의 요소에 `className`으로 할당할 클래스 네임을 지정합니다.

> App.js

```javascript
import React, { useState } from 'react';
import './App.css';

export default function App() {
    const [count, setCount] = useState(0);

    return( 
        <>
            <button className="button" onClick={() => setCount(count+1)}>클릭</button>
            <div>클릭한 횟수 : {count}</div>
        </>
    )
}
```

> App.css

```css
.button {
    outline: none;
    padding: 4px 12px;
    border-radius: 5px;
}
```


&nbsp;
## 2. Inline CSS 스타일링

외부 CSS를 import하지 않아도, inline CSS로도 동일한 스타일을 지정할 수 있습니다. `style` 어트리뷰트 내에 객체로 할당하며, 속성 이름은 `camelCase`로 지정하고 값은 `number`가 아닌 경우 `string` 형태로 지정합니다.

> App.js

```javascript
import React, { useState } from 'react';

export default function App() {
    const [count, setCount] = useState(0);

    return( 
        <>
            <button style={{ outline: "none", padding: "4px 12px", borderRadius: 5}} 
                className="button" onClick={() => setCount(count+1)}>클릭</button>
            <div>클릭한 횟수 : {count}</div>
        </>
    )
}
```

&nbsp;
## 3. CSS-in-JS (Styled-component)

다음은 `js` 파일 안에서 CSS를 정의하는 방법입니다. 장점은 컴포넌트를 스타일링하는 별도 파일을 확인할 필요 없이 1개의 파일 내에서 스타일을 관리할 수 있습니다. 아래 예시 코드에서는 `Styled-components`를 사용한 스타일링 방식입니다.

```javascript
import React, { useState } from 'react';
import styled from 'styled-components';

export default function App() {
    const [count, setCount] = useState(0);

    return( 
        <>
            <ClickButton onClick={() => setCount(count+1)}>클릭</ClickButton>
            <div>클릭한 횟수 : {count}</div>
        </>
    )
}

const ClickButton = styled.button`
    outline: none;
    padding: 4px 12px;
    border-radius: 5px;

    &:hover {
        background-color: #f5f5f5;
    }
`;
```

styled-components를 사용하려면 먼저 styled-components를 설치해야 합니다. 다음 명령어로 설치합니다.

```shell
$ npm install styled-components
```
위 코드에서 `styled.button`은 HTML 엘리먼트를 의미하고, `ClickButton`은 해당 button 엘리먼트를 사용하는 컴포넌트를 의미합니다. CSS-in-jS의 경우 예시 코드 처럼 별도 파일을 분리하지 않아도 로직과 스타일링을 확인할 수 있다는 장점이 있습니다.

&nbsp;
## 4. Tailwind를 활용한 스타일링

다음은 Tailwind를 활용해서 스타일링 하는 방법 입니다. Tailwind는 Bootstrap 처럼 외부 라이브러리에 정의된 스타일을 className을 사용해서 필요한 CSS를 할당할 수 있습니다. 



&nbsp;

---
&nbsp;

[![Elice UTM Banner](/blog/assets/elice/SW7_jihoonkim_bottom_banner.png)](https://elice.training/track/sw?utm_source=sw7&utm_medium=blog&utm_campaign=challenge&utm_content=m2gzitm8b)
&nbsp;
> 태그 Tag : #엘리스트랙 #엘리스트랙후기 #리액트네이티브강좌 #온라인코딩부트캠프 #온라인코딩학원 #프론트엔드학원 #개발자국비지원 #개발자부트캠프 #국비지원부트캠프 #프론트엔드국비지원 #React #Styledcomponent #React Router Dom #Redux #Typescript #Javascript