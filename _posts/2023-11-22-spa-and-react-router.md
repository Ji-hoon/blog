---
title: SPA와 React Router
updated: 2023-11-22 12:00
---

> Elice SW Track 7기 - 11주차 진행 내용입니다.


![Elice Banner](/blog/assets/elice/SW7_top_banner.png)

&nbsp;

React, Vue, Angular 등의 라이브러리들은 주로 **SPA**(Single Page Application, 싱글 페이지 애플리케이션/앱)를 만드는데 사용됩니다. SPA는 물리적으로는 하나의 HTML 파일만을 사용하면서, 화면 내에서 변화하는 정보들을 표시하는 UI만 리렌더링하는 형태의 웹 애플리케이션 제공 방식입니다.

> 참고 - SPA 개념 : [링크](https://ko.wikipedia.org/wiki/%EC%8B%B1%EA%B8%80_%ED%8E%98%EC%9D%B4%EC%A7%80_%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98)

SPA를 만들기 위해서 `VanillaJS`에서 제공되는 기능들이 어떤 것들이 있고, `React`에서는 어떤 도구들을 사용하면 되는지 다뤄보도록 하겠습니다.



&nbsp;

## 목차

[1. 라우팅 Routing 과 라우터 Router](#1-라우팅-routing-과-라우터-router)

[2. VanillaJS에서의 라우팅](#2-vanillajs에서의-라우팅)

[3. React에서의 라우팅](#3-react에서의-라우팅)


&nbsp;

---

&nbsp;
## 1. 라우팅 Routing 과 라우터 Router

SPA를 살펴보기 전에 먼저 **라우팅 Routing** 과 **라우터 Router** 라는 개념에 대해서 살펴보면, 아래 MDN 페이지에서는 **라우터**의 정의를 다음과 같이 설명하고 있습니다.

> 참고 - SPA에서 라우터 개념 : [링크](https://developer.mozilla.org/ko/docs/Glossary/Routers)

```
애플리케이션 계층에서 Single-page application의 경우, 라우터는 주어진 URL로 표시되는 웹 페이지를 결정하는 라이브러리입니다. 이 미들웨어 모듈은 다음 페이지를 열기 위해 렌더링 되는 파일에 대한 경로가 제공되는 모든 URL 기능에 사용됩니다.
```
반면 **라우팅**의 경우에는 아래와 같이 설명하고 있습니다.

> 참고 - 웹에서의 라우팅 개념 : [링크](https://developer.mozilla.org/ko/docs/Learn/Tools_and_testing/Client-side_JavaScript_frameworks/Introduction#%EB%9D%BC%EC%9A%B0%ED%8C%85)

```
웹사이트의 링크를 따라가면 브라우저가 서버와 통신하고 표시할 새 콘텐츠를 가져옵니다. 이렇게 하면 주소 표시줄의 URL이 변경되고, 새 URL을 저장하여 나중에 다시 페이지로 돌아오거나 다른 사람과 공유하여 같은 페이지를 쉽게 찾을 수 있습니다. 이를 서버 측 라우팅이라고 합니다.

최신 웹 애플리케이션은 일반적으로 새 HTML 파일을 가져오거나 렌더링하지 않습니다. 단일 HTML 파일을 로드하고, 사용자를 웹의 새 주소로 안내하지 않고 내부 DOM을 지속적으로 업데이트 합니다. 각각의 새 웹페이지는 기본적으로 라우팅이 수행되지 않습니다. 이러한 방식으로 클라이언트 애플리케이션에서 라우팅을 처리하는 경우 클라이언트 측 라우팅이라고 합니다.
```
위에서 설명된 것처럼 앞으로 React에서 사용되는 라우팅의 개념은 **클라이언트 사이드 라우팅**을 의미합니다.


&nbsp;
## 2. VanillaJS에서의 라우팅

VanillaJS에서도 SPA를 구현하기 위한 라우팅 메서드들이 존재합니다. 바로 `history.pushState` 메서드와 `window.location` 객체를 사용하여 SPA을 구현할 수 있습니다. 아래는 간단한 SPA 예시 코드 입니다. 

```html
<!DOCTYPE html>
<html>
<head>
    <title>Vanilla JS SPA</title>
</head>
<body>
    <nav>
        <ul>
            <li><a name="home" href="/" >Home</a></li>
            <li><a name="about" href="/about" >About</a></li>
            <li><a name="contact" href="/contact" >Contact</a></li>
        </ul>
    </nav>
    <div id="app"></div>
    <script>
        const links = document.querySelector('ul');

        links.addEventListener("click", (e) => {
            e.preventDefault(); // 화면이 새로고침 되지 않도록 처리
            if(e.target.name === "home") {
                navigate('/');
            } else if (e.target.name === "about") {
                navigate('/about');
            } 
            else if (e.target.name === "contact") {
                navigate('/contact');
            } 
        });
        
        function navigate(path) {
            history.pushState(null, null, path);
            handleRoute();
        }

        function handleRoute() {
            const path = window.location.pathname;
            if (path === '/') {
                renderHome();
            } else if (path === '/about') {
                renderAbout();
            } else if (path === '/contact') {
                renderContact();
            }
        }

        function renderHome() {
            const app = document.getElementById('app');
            app.innerHTML = '<h1>Home Page</h1>';
        }

        function renderAbout() {
            const app = document.getElementById('app');
            app.innerHTML = '<h1>About Page</h1>';
        }

        function renderContact() {
            const app = document.getElementById('app');
            app.innerHTML = '<h1>Contact Page</h1>';
        }

        window.addEventListener('popstate', handleRoute);
        window.addEventListener('DOMContentLoaded', handleRoute);
        
    </script>
</body>
</html>
```

위 코드를 실행하면, `Home`와 `About`, 그리고 `Contact` 링크 가 담긴 `Anchor tag` 클릭 시 화면 새로고침이 일어나지 않고 `URL pathname`만 바뀐채 `#app` 부분의 내용만 갱신되는 것을 확인할 수 있습니다.

&nbsp;

![SPA example with VanillaJS](/blog/assets/posts/asset-spa-with-vanillajs.gif)

&nbsp;

여기서는 `history`객체의 `pushState` 메서드 호출 시 인자로 `null, null, {pathname}` 을 전달하고, `handleRoute` 메서드를 실행시켜 `window.location` 객체의 `pathname`에 대응되는 `render-` 메서드를 실행하는 형태로 동작합니다.

&nbsp;

하지만 VanillaJS의 경우, 라우팅에 필요한 메서드들이 페이지가 커져감에 따라서 많아지게 됨을 예상할 수 있습니다. 그럼 이제 React에서는 동일한 기능을 구현하기 위해서 어떤 메서드들을 사용하고 있는지 살펴보겠습니다.

&nbsp;
## 3. React에서의 라우팅

React에서는 라우팅을 구현하기위한 라이브러리로 `React Router`를 가장 많이 사용하게 됩니다. 아래 명령어로 설치합니다.

```shell
$ npm install react-router-dom
```
> 참고 - React Router 튜토리얼 : [링크](https://reactrouter.com/en/main/start/tutorial)

현재 `React Router`의 최신 버전은 `6.4`이며, 해당 버전의 문법을 사용하여 동일한 기능을 수행하는 SPA 코드를 작성해보겠습니다.

```javascript
import React from 'react';
import { BrowserRouter as Router, Routes, Route, Link } from 'react-router-dom';

function App() {
  return (
    <Router>
      <nav>
        <ul>
          <li><Link to="/">Home</Link></li>
          <li><Link to="/about">About</Link></li>
          <li><Link to="/contact">Contact</Link></li>
        </ul>
      </nav>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/contact" element={<Contact />} />
      </Routes>
    </Router>
  );
}

function Home() {
  return <h1>Home Page</h1>;
}

function About() {
  return <h1>About Page</h1>;
}

function Contact() {
  return <h1>Contact Page</h1>;
}

export default App;
```

기본적으로 클라이언트 사이드 라우팅에 포함될 모든 요소들을 `Router` 하위에 명시하고, `URL path`와 렌더링할 `요소` 또는 `컴포넌트`들을 `Route` 에 명시합니다.

&nbsp;

이렇게 React Router 라이브러리를 활용하면, 간단하게 SPA를 구축하기 위한 라우팅을 작성할 수 있습니다.

> 참고 - 라이브러리 없이 라우터 구현하기 : [링크](https://fe-developers.kakaoent.com/2022/221124-router-without-library/)


&nbsp;

---
&nbsp;

[![Elice UTM Banner](/blog/assets/elice/SW7_jihoonkim_bottom_banner.png)](https://elice.training/track/sw?utm_source=sw7&utm_medium=blog&utm_campaign=challenge&utm_content=m2gzitm8b)
&nbsp;
> 태그 Tag : #엘리스트랙 #엘리스트랙후기 #리액트네이티브강좌 #온라인코딩부트캠프 #온라인코딩학원 #프론트엔드학원 #개발자국비지원 #개발자부트캠프 #국비지원부트캠프 #프론트엔드국비지원 #React #Styledcomponent #React Router Dom #Redux #Typescript #Javascript