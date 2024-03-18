---
title: Vitest, React Testing Library를 이용한 테스트 코드 작성하기 (1)
updated: 2024-03-14 12:00
---

> '가계부를 부탁해' 프로젝트를 진행하며 작성한 내용입니다.

[![project-logo](https://github.com/Ji-hoon/Home-accountant/raw/master/client/public/img-logo.png)](https://github.com/Ji-hoon/Home-accountant){:target="_blank"}

&nbsp;

새롭게 개인 프로젝트를 진행하면서 다뤘던 기술들에 대해서 기록을 남겨보려 합니다. 오늘은 다섯 번째 주제로, **Vitest & React Testing Library**로 테스트 코드를 작성해 보았습니다.

> 프로젝트 보러가기 👉 : [링크](http://35.231.16.39/)

&nbsp;

## 목차

[1. TDD와 테스트 코드를 작성하는 이유](#1-tdd와-테스트-코드를-작성하는-이유)

[2. 테스트의 종류와 도구](#2-테스트의-종류와-도구)

[3. Jest보다 Vitest를 선택한 이유](#3-jest보다-vitest를-선택한-이유)

[4. Vitest 환경 설정하기](#4-vitest-환경-설정하기)

[5. 테스트 코드 작성하기](#5-테스트-코드-작성하기)

&nbsp;

---

&nbsp;

## 1. TDD와 테스트 코드를 작성하는 이유

**TDD(Test Driven Development), 테스트 주도 개발**이라는 개념을 많이 들어보셨을 텐데요. 테스트 주도 개발이라 함은 간단히 말하자면 사용자 시나리오에 따라 테스트 코드를 먼저 작성한 뒤, 실제 코드를 작성해 나가는 개발 방법론입니다. 실제 코드를 작성하기 전이기에 테스트 코드는 반드시 **실패**합니다. 실패하는 테스트 코드를 먼저 작성한 뒤, 테스트가 **성공**하는 실제 코드를 작성하고 리팩토링하는 프로세스를 따르게 됩니다. (Red-Green 테스트)

> 참고 - TDD란? 테스트주도개발에 대한 편견과 실상, 방법론 : [링크](https://media.fastcampus.co.kr/knowledge/dev/tdd/){:target="_blank"}

&nbsp;

![why tdd](/blog/assets/posts/asset-tdd-why-tdd.png)*TDD를 하는 이유 -- 출처 : udemy ([링크](https://kmooc.udemy.com/course-dashboard-redirect/?course_id=4621352){:target="_blank"})*{: .caption}

&nbsp;

그럼 반드시 테스트 주도 개발을 해야 할까요? 이 부분에 대해서는 장단점을 먼저 짚어보겠습니다.

> 장점

1. 코드가 내 손을 벗어나기 전에 가장 빠르게 피드백 받을 수 있다.
2. 필요한 코드만 작성하기 때문에 오버 엔지니어링을 방지할 수 있다.
3. 한 번 테스트 코드를 작성해두면, 애플리케이션 코드가 변경되었을 때 다시 테스트 코드를 작성할 필요가 없어 효율적이다.

> 단점

1. 테스트 코드를 작성하지 않는 경우에 비해서 초기 비용(시간/리소스) 증가로 인해 생산성 저하가 발생할 수 있다.
2. 팀원들이 테스트 코드 작성과 TDD에 적응하는 기간이 필요하다.

이렇듯 TDD는 반드시 차용해야 하는 만능 방법론이라기 보다는, 조직과 팀의 상황에 따라 장단점을 비교하여 도입하면 될 것 같습니다.

&nbsp;

## 2. 테스트의 종류와 도구

만약 팀에서 TDD를 적용하기로 결정이 되었다면, 테스트 방식에는 어떤 것들이 있고 어떤 도구들을 사용하는지에 대한 지식이 필요하게 될텐데요. 이번에는 테스트의 종류와, 사용하는 도구들에 대해 알아보도록 하겠습니다.

&nbsp;

![type of tests](/blog/assets/posts/asset-tdd-type-of-tests.png)*테스트의 종류 -- 출처 : udemy ([링크](https://kmooc.udemy.com/course-dashboard-redirect/?course_id=4621352){:target="_blank"})*{: .caption}

&nbsp;

#### 유닛(단위) 테스트 — Unit tests

* 사용 툴 : **Jasmine**, **Jest**, **Karma**, **Mocha**
* 컴포넌트 또는 특정 함수 단위로 분할하여 검증하는 테스트 방식입니다. 작은 단위의 컴포넌트나 기능 등에 국한하여 검증하기에 비용이 가장 적습니다.

#### 기능 테스트 — Functional tests

* 사용 툴 : **Jest**, **Testing Library**
* 애플리케이션의 비즈니스 요구 사항을 만족하는 지 검증하는 테스트 방식입니다. 

#### 통합 테스트 — Integration tests

* 사용 툴 : **Supertest**
* 여러 단위 또는 구성 요소 간의 상호 작용을 검증하는 테스트 방식입니다. 데이터베이스와의 상호 작용을 테스트합니다.
  
#### 인수 테스트 / E2E 테스트 —  Acceptance / End to End tests

* 사용 툴 : **Cypress**, **Puppeteer**, **Selenium**
* 시스템의 시작부터 끝까지 전체 흐름을 확인하며 검증하는 테스트 방식입니다. 실제 환경과 똑같이 구성해야 하기에 비용이 가장 높습니다.

> 참고 - 테스트 코드는 왜 만들까? : [링크](https://yozm.wishket.com/magazine/detail/1964/){:target="_blank"}

&nbsp;

## 3. Jest보다 Vitest를 선택한 이유

**Jest**는 가장 많이 사용되는 테스트용 라이브러리 입니다. **React**에서는 **React Testing Library**와 같이 사용하게 되는데요. **CRA**가 아닌 **Vite**를 사용하게 될 경우 **Vitest**가 설정이 더 간편하고 (별도 테스트 설정 파일 대신 `vite.config.ts` 파일에 설정), **Jest**보다 속도가 더 빠르다는 장점이 있어 선택했습니다.

&nbsp;

| - | Jest | Vitest |
|-------|-------|-------|
| 문서화 및 참고 자료 | 다양함 | 상대적으로 적음 |
| 속도 | - | 상대적으로 빠름 |
| ES Module 사용 | 설정이 어려움 | 설정이 쉬움 |
| 문법 | test("", () => {}) | it("", () => {}) |

&nbsp;

> 참고 - Vitest 소개 : [링크](https://careerly.co.kr/comments/98604){:target="_blank"} / Vitest Config 방법 : [링크](https://vitest.dev/config/){:target="_blank"}


&nbsp;

## 4. Vitest 환경 설정하기

먼저 **Vite**를 사용하여 프로젝트를 생성해줍니다.

```shell
$ npm create vite@latest app-name -- --template react-ts
$ cd app-name
```

프로젝트를 생성하면 생성된 app 폴더 하위에 `vite.config.ts` 파일이 생성됩니다. 이제 **Vitest** 와 **React Testing Library (react, js-dom)** 를 추가하고 test 설정 코드를 추가합니다.

```shell
app-name$ npm install --save-dev vitest
app-name$ npm install --save-dev @testing-library/react
app-name$ npm install --save-dev @testing-library/js-dom
```

&nbsp;

> vite.config.ts

```typescript
// "reference"를 선언
/// <reference types="vitest" />
/// <reference types="vite/client" />

import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  // "test" 객체를 추가하여 설정 코드를 추가
  test: {
    globals: true,
    environment: "jsdom",
    setupFiles: "./setupTest.js", // setupTest.js 파일과 경로를 입력
    css: true, // css 테스트를 하지 않는다면 사용하지 않아도 됨
  },
  resolve: {
    alias: {
      "@": "/src",
    },
  },
});
```
### 코드 설명

1. **globals** : 테스트 파일에서 전역으로 사용할 변수를 설정합니다. true 로 설정하면 vitest의 globals를 사용할 수 있습니다.
2. **environment** : 테스트 환경을 설정합니다. “jsdom” 으로 설정하면 브라우저 환경을 시뮬레이션하는 JSDOM 환경을 사용할 수 있습니다.
3. **setupFiles**: 테스트 실행 전에 실행할 파일을 설정합니다. 파일명은 자유롭게 설정하면 됩니다.
4. **css**: CSS 테스트를 할지 여부를 설정합니다. `true` 로 설정하면 CSS 테스트를 수행합니다. 설정하지 않으면 `false` 로 간주합니다.

&nbsp;

> setupTest.js

```javascript
import "@testing-library/jest-dom";
import "@testing-library/react";

import { cleanup } from "@testing-library/react";
import { beforeEach, beforeAll, afterEach, afterAll } from "vitest";

beforeAll(() => {});

beforeEach(() => {
  cleanup(); // 이전 테스트에서 실행된 코드를 초기화하는 함수
})

afterEach(() => {});

afterAll(() => {});
```
### 코드 설명

1. **beforeAll** : 테스트 파일 단위로 전체 테스트 시작 전에 실행할 코드를 작성합니다.
2. **beforeEach** : 각 테스트 시작 전에 실행할 코드를 작성합니다.
3. **afterEach** : 각 테스트 완료 후에 실행할 코드를 작성합니다.
4. **afterAll** : 전체 테스트 완료 후에 실행할 코드를 작성합니다.

&nbsp;

만약 TypeScript를 사용한다면, 

## 5. 테스트 코드 작성하기

그럼 이제 간단한 샘플 테스트 코드를 작성해보겠습니다. 테스트 코드를 작성할 파일명은 `*.test.tsx` 형태로 **.test**를 붙여주어야 합니다. 테스트 코드는 기본적으로 **it** 명령어로 테스트 단위를 구분하며, **render** 메소드로 테스트할 컴포넌트를 불러오며, **expect** 메소드로 테스트 통과 여부를 검증합니다.

&nbsp;

> /src/Sample.test.tsx

```typescript
import { expect, it } from "vitest";
import { render, screen } from "@testing-library/react";

it("샘플 테스트", () => {
  render(<h1>샘플 코드입니다.</h1>);
  const sampleHeading = screen.getByRole("heading", {
    name: "샘플 코드입니다.",
  });
  expect(sampleHeading).toBeInTheDocument();
});
```

### 코드 설명

1. vitest로부터 expect와 it 메소드를 import 합니다.
2. @testing-library/react로부터 render와 screen 메소드를 import 합니다.
3. it 메소드의 인자로는 테스트 이름 (string), 콜백 함수를 작성합니다.
4. render 메소드에 렌더링할 컴포넌트 또는 jsx를 작성합니다.
5. screen.***ByRole 메소드로 렌더링된 내용 중 "샘플 코드입니다."와 동일한 문자를 가지고 있는 "heading" 요소를 찾아 sampleHeading 엘리먼트에 할당합니다.
6. expect 메소드로 sampleHeading가 화면에 존재하는지 검증합니다.

&nbsp;

만약 렌더링할 컴포넌트에서 TypeScript를 사용한다면,`tsconfig.json` 파일에 다음 코드를 추가해 줍니다.

```json
{
  "compilerOptions": {
    // ... other options
    "types": ["@testing-library/jest-dom", "vitest/globals"],
    // ... other options
  }
}
```

> 참고 - Property 'toBeInTheDocument' does not exist on type 에러 : [링크](https://github.com/testing-library/jest-dom/issues/546#top){:target="_blank"}

&nbsp;

지금까지 테스트 코드를 작성하기 위한 기본적인 지식과 환경 설정을 진행했는데요. 본격적인 테스트 코드 작성은 이어지는 글에서 다뤄보도록 하겠습니다. 🥰

&nbsp;

<details>
<summary>참고하면 좋은 링크 (클릭하여 펼치기)</summary>

- RTL 쿼리 : [링크](https://testing-library.com/docs/queries/about/){:target="_blank"}
- RTL 쿼리 cheatseat : [링크](https://testing-library.com/docs/react-testing-library/cheatsheet/){:target="_blank"}
- RTL 쿼리 우선순위 : [링크](https://testing-library.com/docs/queries/about/#priority){:target="_blank"}
- Jest-dom matchers : [링크](https://github.com/testing-library/jest-dom){:target="_blank"}
  
</details>

&nbsp;

> 태그 Tag : #react #vitest #react-testing-library #vite #tdd



