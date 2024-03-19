---
title: Vitest, React Testing Library를 이용한 테스트 코드 작성하기 (3)
updated: 2024-03-16 12:00
---

> '가계부를 부탁해' 프로젝트를 진행하며 작성한 내용입니다.

[![project-logo](https://github.com/Ji-hoon/Home-accountant/raw/master/client/public/img-logo.png)](https://github.com/Ji-hoon/Home-accountant){:target="_blank"}

&nbsp;

이전 글에서 Mock Service Worker와 Custom Util 함수 등을 사용하여 테스트 코드를 재구성 해보았습니다. 이번에는 테스트 시나리오를 검증하는 실제 테스트 코드를 작성해 보도록 하겠습니다. (이번 프로젝트에서는 TDD를 고려하지 않고 애플리케이션 코드를 먼저 작성했기 때문에, 일반적인 순서의 테스트 코드 작성 절차를 따르지 않았음을 양해 부탁드립니다)

> 이전 글 - Vitest, React Testing Library를 이용한 테스트 코드 작성하기 (2) : [링크](https://ji-hoon.github.io/blog/run-test-with-vitest-and-rtl-2)

&nbsp;

## 목차

[1. 페이지 내 요소 검증하기](#1-페이지-내-요소-검증하기)

[2. 페이지 이동 검증하기](#2-페이지-이동-검증하기)

[3. loader 데이터에 따른 분기 처리 검증하기](#3-loader-데이터에-따른-분기-처리-검증하기)

[4. React Portal을 사용하는 코드 검증하기](#4-react-portal을-사용하는-코드-검증하기)

&nbsp;

---

&nbsp;

## 1. 페이지 내 요소 검증하기

이번 글에서 검증하려는 요구사항은 아래와 같습니다.

1. [로그인] 사용자는 비로그인 상태에서 홈페이지 진입 시 랜딩 페이지를 확인할 수 있다.
2. [로그인] 사용자는 랜딩 페이지에서 로그인 버튼을 클릭 시 로그인 페이지로 진입해야 한다.
3. [로그인] 사용자는 로그인 성공 시, 애플리케이션 메인 페이지로 진입해야 한다.
4. [지출 내역] 개별 지출 내역을 클릭 시 "지출 내역 수정" 모달이 표시된다.

기 작성된 애플리케이션 코드에서는 페이지 접근 시`useLoaderData` 값으로 **로컬 스토리지**에 담긴 **로그인 정보**를 확인한 뒤, 그 결과값을 `loader data`로 넘겨주고, `RootPage`에서 이를 판별하여 `LandingPage` 또는 `MainPage`로 라우팅 분기를 시켜주고 있습니다. 

먼저 1번 요구사항에 대한 테스트 코드를 작성해보겠습니다.

> /src/tests/Scenario.test.tsx

```typescript
import { describe, expect, it, vi } from "vitest";
import { screen } from "@testing-library/react";
import { userEventSetup } from "./utils/utils";
import { emptyMockUserLoaderData } from "./mocks/useLoaderData";
import LandingPage from "../pages/Landing/LandingPage";

describe("[테스트 시나리오]", () => {
  it("사용자는 비로그인 상태에서 홈페이지 진입 시 랜딩 페이지를 확인할 수 있다.", async () => {
    userEventSetup(
      [{ path: "/", jsx: <LandingPage /> }],
      emptyMockUserLoaderData,
    );

    const loginButtonElements = await screen.findAllByRole("button", {
      name: /로그인 하러가기/i,
    });

    expect(loginButtonElements[0]).toBeInTheDocument();
  });
});
```

> /src/tests/mocks/useLoaderData.ts

```typescript
import { vi } from "vitest";

// loader 인자로 전달할 mock value 값
export const emptyMockUserLoaderData = vi.fn().mockReturnValue({
  result: {},
});
```

**파일**과 **describe** 단위로 시나리오를 구분하여 작성했습니다. 이전 글에서 작성했던 utils 파일에서 `userEventSetup` 메소드를 import하여 사용합니다. 이 때 `loader`에 빈 객체가 담긴 mock data를 전달하여 로그인된 계정 정보가 없는 상태를 만들고, 렌더링된 화면에 "**로그인 하러가기**" 라는 버튼이 존재하는지 `findAllByRole` 메소드로 탐색합니다. 디자인상 동일한 레이블을 가진 버튼이 2개가 존재하므로 여러개를 찾고자 할 경우에 `findAll***` 을 사용합니다. 2개 이상이 match되면 `find***` 메소드를 사용한 경우 테스트가 실패하게 되므로, 찾고자 하는 요소에 따라서 적절히 사용해야 합니다.

&nbsp;

![test failed](/blog/assets/posts/asset-tdd-test-failed-with-single-query.png)

&nbsp;

## 2. 페이지 이동 검증하기

다음은 2번 요구사항에 대한 테스트 코드를 추가해보겠습니다.

> /src/tests/Scenario.test.tsx

```typescript
import { describe, expect, it, vi } from "vitest";
import { screen } from "@testing-library/react";
import { userEventSetup } from "./utils/utils";
import { emptyMockUserLoaderData } from "./mocks/useLoaderData";
import LandingPage from "../pages/Landing/LandingPage";
import LoginPage from "../pages/login/LoginPage";

...

describe("[테스트 시나리오]", () => {
  ...

  it("사용자는 랜딩 페이지에서 로그인 버튼을 클릭 시 로그인 페이지로 진입해야 한다.", async () => {
    const { user } = userEventSetup(
      [
        { path: "/", jsx: <LandingPage /> },
        { path: "/login", jsx: <LoginPage /> },
      ],
      emptyMockUserLoaderData
    );

    const loginButtonElements = await screen.findAllByRole("button", {
      name: /로그인 하러가기/i,
    });
    await user.click(loginButtonElements[0]);

    const loginHeaderElement = await screen.findByRole("heading", {
      name: /자산 관리를 시작해보세요/i,
    });
    expect(loginHeaderElement).toBeInTheDocument();

    const kakaoLoginButtonElement = await screen.findByRole("button", {
      name: /카카오 계정으로 로그인/i,
    });
    expect(kakaoLoginButtonElement).toHaveAttribute("id", "kakao");
    screen.debug();
  });
});
```

새로운 테스트를 작성합니다. 이번에는 **로그인 하러가기** 버튼을 클릭 시 새로운 router path로 이동하는지 검증하기 위해, `/login` path에 렌더링할 `LoginPage` 컴포넌트를 추가해줍니다. 로그인 페이지 컴포넌트에는 "**지금 자산 관리를 시작해보세요**" 라는 문구를 가진 `h3` 엘리먼트와 "**카카오 계정으로 로그인**" 이라는 레이블과 `id`로 `kakao`라는 값을 가지는 버튼이 존재합니다. 1번과 마찬가지로 `findByRole` 메소드로 페이지가 정상적으로 이동되어 해당 요소들이 화면에 존재하는 지 검증합니다. 다음은 마지막 3번 요구사항을 검증하는 테스트 코드를 작성해보겠습니다.

&nbsp;

## 3. loader 데이터에 따른 분기 처리 검증하기

3번 요구사항도 마찬가지로 새로운 테스트 함수로 작성합니다. 이번에는 빈 loader data 대신 **계정 정보**를 담고 있는 `mock data`를 `loader` 값으로 전달하여 **로그인** 상태일 때 `MainPage`로 이동하는지 검증합니다. `MainPage` 렌더링 시 사용되는 API 응답값을 mock handler로 정의해야 하는데요. `/src/tests/mocks/handlers.ts`파일에 정의된 mock handler 대신 현재 테스트에서 임시적으로 사용할 mock handler를 `server.resetHandlers` 메소드를 사용하여  재설정할 수 있습니다. (동일한 API 주소에 대한 mock handler라면, 테스트 내부에 정의된 resetHandlers가 리턴하는 값이 우선순위를 갖습니다.)

> /src/tests/Scenario.test.tsx

```typescript
import { describe, expect, it, vi } from "vitest";
import { screen, waitFor } from "@testing-library/react";
import { userEventSetup } from "./utils/utils";
import { emptyMockUserLoaderData, mockUserLoaderData } from "./mocks/useLoaderData";
import LandingPage from "../pages/Landing/LandingPage";
import LoginPage from "../pages/Login/LoginPage";
import MainPage from "../pages/Main/MainPage";
import { server } from "./mocks/server";
import { http, HttpResponse } from "msw";

...

describe("[테스트 시나리오]", () => {
  ...

  it("사용자는 로그인 성공 시, 애플리케이션 메인 페이지로 진입해야 한다.", async () => {
    server.resetHandlers(
      http.get(`${import.meta.env.VITE_BACKEND_URL}/api/expenses`, async () => {
        return HttpResponse.json([]);
      }),
      http.get(
        `${import.meta.env.VITE_BACKEND_URL}/api/expenses/amounts`,
        async () => {
          return HttpResponse.json(0);
        }
      ),
      http.get(`${import.meta.env.VITE_BACKEND_URL}/api/groups`, async () => {
        return HttpResponse.json({
          message: "그룹 조회에 성공했습니다.",
          groupInfo: {
            _id: "abced",
            name: "훈님의 가계부",
            members: [
              {
                userId: "12345",
                role: "OWNER",
                joinedAt: "2024-01-29T05:52:29.299Z",
                _id: null,
              },
            ],
          },
        });
      })
    );

    userEventSetup(
      [
        { path: "/", jsx: <MainPage /> },
        { path: "/main/expenses", jsx: <MainPage /> },
        { path: "/main/expenses/weekly", jsx: <MainPage /> },
        { path: "/login", jsx: <LoginPage /> },
      ],
      mockUserLoaderData
    );

    await waitFor(
      () => {
        const expenseFloatingButtonElement = screen.getByRole("button", {
          name: /지출 내역 추가/i,
        });
        expect(expenseFloatingButtonElement).toBeInTheDocument();
        screen.debug();
      },
      { timeout: 1000 }
    );
  });
});
```
> /src/tests/mocks/useLoaderData.ts

```typescript
import { vi } from "vitest";

...

// 로그인된 계정 정보를 담고 있는 mock return value를 정의
export const mockUserLoaderData = vi.fn().mockReturnValue({
  result: {
    userId: "12345",
    nickname: "테스터",
    profile: "/img-default-profile.png",
    currentGroup: "abced",
    currentRole: "OWNER",
  },
});
```

이번 테스트 코드에는 `waitFor` 라는 메소드를 추가했습니다. `testing-library`에서 import 할 수 있으며, 이 메소드는 페이지 이동 또는 비동기 작업으로 인해 렌더링이 지연되는 경우 데이터를 정상적으로 가져오지 않을 수 있는 우려가 있다면 `timeout`에 지정된 시간까지 `waitFor` 콜백함수에 작성된 코드를 지연 시킨 후 `timeout` 시간이 지난 후에 실행시키는 함수입니다. (timeout의 단위는 ms, 1000 = 1초)

> 참고 - Async Methods (Testing Library) : [링크](https://testing-library.com/docs/dom-testing-library/api-async/#waitfor){:target="_blank"}

&nbsp;

## 4. React Portal을 사용하는 코드 검증하기

마지막으로 Modal, Dropdown 처럼 `React Portal`을 사용한 `Dialog` 요소를 검증하는 코드를 작성해보도록 하겠습니다.

> /src/tests/Scenario.test.tsx

```typescript
import { describe, expect, it, vi } from "vitest";
import { screen, waitFor } from "@testing-library/react";
import { userEventSetup } from "./utils/utils";
import { emptyMockUserLoaderData, mockUserLoaderData } from "./mocks/useLoaderData";
import LandingPage from "../pages/Landing/LandingPage";
import LoginPage from "../pages/Login/LoginPage";
import MainPage from "../pages/Main/MainPage";
import { server } from "./mocks/server";
import { http, HttpResponse } from "msw";
import Dialog from "../components/dialog/Dialog";

...

describe("[테스트 시나리오]", () => {
  ...

  it("개별 지출 내역을 클릭 시 '지출 내역 수정' 모달이 표시된다.", async () => {
    server.resetHandlers(
      http.get(`${import.meta.env.VITE_BACKEND_URL}/api/expenses`, async () => {
        return HttpResponse.json([
          {
            _id: "65b9c7bc62cbeb63b583e64f",
            amounts: 27500,
            businessName: "컬리",
            date: new Date("2024-01-28T15:00:00.000Z").toISOString(),
            isRecurring: "일시불",
            category: "식비",
            owner: "훈",
          },
        ]);
      }),
    )

    const mainWithDialog = () => {
      return (
        <>
          <MainPage />
          <Dialog />
        </>
      );
    };

    const { user, container } = userEventSetup(
      [
        { path: "/", jsx: <MainPage />, },
        { path: "/main/expenses", jsx: <MainPage /> },
        { path: "/main/expenses/weekly", jsx: mainWithDialog(), },
        { path: "/main/expenses/monthly", jsx: mainWithDialog(), },
        { path: "/login", jsx: <LoginPage /> },
      ],
      mockUserLoaderData,
    );

    // react portal을 사용하는 Dialog 컴포넌트가 사용할 dialog div를 임의 추가
    const portalRoot = document.createElement("div");
    portalRoot.setAttribute("id", "dialog");
    document.body.appendChild(portalRoot);

    await waitFor(
      () => {
        const listItem = screen.getByRole("list", { name: "" });
        expect(listItem.style.pointerEvents).toBe("");
        screen.debug();
      },
      { timeout: 1000 },
    );

    const expenseList = await screen.findAllByTestId("expense-item", {
      exact: false,
    });

    const filteredItem = expenseList.find(
      (item) => item.getAttribute("id") === "65b9c7bc62cbeb63b583e64f",
    );
    expect(filteredItem).toBeInTheDocument();
    await user.click(filteredItem as Element);

    const modalSubmit = await screen.findByRole("button", {
      name: "지출 내역 수정",
    });
    expect(modalSubmit).toBeInTheDocument();

    screen.debug();
  });
});
```

`Dialog` 컴포넌트는 DOM에서 id가 dialog인 div 요소를 찾고, 해당 요소 하위에 Dialog 요소를 렌더링하게 됩니다. 보통 이 요소는 `index.html`에 정의하게 되는데요. 따라서 테스트 코드 상에서 해당 요소를 대신 사용할 신규 요소를 렌더링 시 추가하여 사용합니다.

&nbsp;

또한 이전 섹션에서 다뤘던 `server.resetHandlers`를 통해 지출 내역에 불러올 API 응답값을 mock handler로 작성하고, 1개의 지출 내역이 표시되도록 합니다. 렌더링된 화면에서 해당 id를 가진 요소를 찾고 클릭 이벤트를 실행시킨 뒤, 모달 내에 "**지출 내역 수정**" 레이블을 가진 `submit` 버튼이 존재하는지 검증합니다.

&nbsp;

![test scenario passed](/blog/assets/posts/asset-tdd-test-scenario-passed.png)

&nbsp;


#### 참고하면 좋은 링크

- vite 프로젝트에서 msw, vitest를 이용해서 테스트 코드 작성 : [링크](https://velog.io/@ckstn0777/vite-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8%EC%97%90%EC%84%9C-msw-vitest%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%B4-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EC%BD%94%EB%93%9C-%EC%9E%91%EC%84%B1%ED%95%B4%EB%B3%B4%EA%B8%B0){:target="_blank"}
- it과 test의 차이 : [링크](https://puleugo.tistory.com/141){:target="_blank"}
- React portal을 활용한 드롭다운 컴포넌트 구현하기 : [링크](https://ji-hoon.github.io/blog/implement-dropdown-with-react-portal){:target="_blank"}

&nbsp;

> 태그 Tag : #react #vitest #react-testing-library #vite #tdd



