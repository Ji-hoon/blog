---
title: Vitest, React Testing Library를 이용한 테스트 코드 작성하기 (2)
updated: 2024-03-15 12:00
---

> '가계부를 부탁해' 프로젝트를 진행하며 작성한 내용입니다.

[![project-logo](https://github.com/Ji-hoon/Home-accountant/raw/master/client/public/img-logo.png)](https://github.com/Ji-hoon/Home-accountant){:target="_blank"}

&nbsp;

이전 글에서 Vitest의 환경 설정과 샘플 테스트 코드를 작성을 해보았습니다. 이번에는 **Mock Service Worker**와 **Custom Util** 함수 등을 사용하여 테스트 코드를 재구성해보도록 하겠습니다.

> 이전 글 - Vitest, React Testing Library를 이용한 테스트 코드 작성하기 (1) : [링크](https://ji-hoon.github.io/blog/run-test-with-vitest-and-rtl)

&nbsp;

## 목차

[1. 테스트 시나리오 작성하기](#1-테스트-시나리오-작성하기)

[2. 중복되는 환경 설정을 util 함수로 분리하기](#2-중복되는-환경-설정을-util-함수로-분리하기)

[3. MSW로 mocking handler 설정하기](#3-msw로-mocking-handler-설정하기)

<!-- [4. 로그인 테스트 시나리오 검증하기](#4-로그인-테스트-시나리오-검증하기)

[5. 지출 내역, 자산 조회 시나리오 검증하기](#5-지출-내역-자산-조회-시나리오-검증하기)

[6. 그룹 관리 시나리오 검증하기](#6-그룹-관리-시나리오-검증하기) -->

&nbsp;

---

&nbsp;

## 1. 테스트 시나리오 작성하기

실제 테스트 코드를 작성하기 전에 어떤 비즈니스 요구사항을 검증해야 하는 지 시나리오를 작성해보는 것은 불필요한 테스트 코드를 작성하게 되는 것을 방지해주고, 또한 어떻게 실제 애플리케이션의 코드를 작성해야 하는지 가이드를 제시해줄 수 있습니다. "**가계부를 부탁해**" 애플리케이션의 요구사항 중 일부를 시나리오 형식으로 작성해보았습니다.

> 참고 - 계산기 애플리케이션 테스트 시나리오 : [링크](https://github.com/Ji-hoon/tdd-calculator){:target="_blank"}

#### 로그인 시나리오

1. 사용자는 비로그인 상태에서 홈페이지 진입 시 랜딩 페이지를 확인할 수 있다.
2. 사용자는 랜딩 페이지에서 로그인 버튼을 클릭 시 로그인 페이지로 진입해야 한다.
3. 사용자는 로그인 성공 시, 애플리케이션 메인 페이지로 진입해야 한다.

#### 지출 내역 시나리오

1. 지출 내역 페이지 상단의 총 지출 금액과 지출 내역 리스트의 금액 값의 합이 일치해야 한다.
2. 페이지 좌측의 월간 지출 내역 메뉴 클릭 시 날짜 표시 방식이 변경되고, 지출 내역이 업데이트 된다. (yyyy년 mm월)
3. 개별 지출 내역을 클릭 시 "지출 내역 수정" 모달이 표시된다.
4. 지출 내역 수정 모달 내에서 정보를 수정 후 하단의 컨펌 버튼 클릭 시 지출 내역이 업데이트 된다.

#### 자산 조회 시나리오

1. 페이지 상단 헤더 영역의 "자산 관리" 버튼을 클릭하면 월간 자산 조회 페이지로 이동된다.
2. 자산 조회 페이지로 이동 시 날짜 단위가 월로 변경되며, 등록된 자산 정보와 합산 금액을 확인할 수 있다.

#### 그룹 관리 시나리오

1. 페이지 상단 헤더 영역의 "그룹 관리" 버튼을 클릭하면 그룹 관리 > 참여 멤버 페이지로 이동된다.
2. 참여 멤버 페이지에서는 그룹에 참여중인 멤버의 정보를 확인할 수 있다.
3. 페이지 좌측의 그룹 정보 메뉴를 클릭하면 그룹 정보를 수정 및 조회 할 수 있는 페이지로 이동된다.
4. 그룹 명을 변경 후 그룹 정보를 업데이트 할 수 있다.

&nbsp;

## 2. 중복되는 환경 설정을 util 함수로 분리하기

앞서 테스트 단위마다 `it` 메소드를 사용하여 코드를 작성한다고 알려드렸습니다. 필수적으로 `render` 함수를 통해서 화면에 렌더링할 요소를 선언하게 되는데, 이 때 여러 테스트에 걸쳐 중복으로 사용되는 코드들에 대해서 별도의 util 함수로 분리해서 사용하면, 테스트 코드에 작성할 코드의 양을 줄이고 더 효율적으로 코드를 작성할 수 있습니다.

&nbsp;

#### 중복되는 기능

1. `Route path`에따라 렌더링할 컴포넌트, `react-router` 코드
2. `userEvent`를 `setup`하는 코드
3. 렌더링할 컴포넌트에서 사용할 `useLoaderData`를 가져오는 코드
4. `Recoil` 등 상태 관리 변수를 사용하기 위한 `Provider` 설정

> /src/tests/utils/utils.tsx

```typescript
import userEvent, { UserEvent } from "@testing-library/user-event";
import { render, RenderResult } from "@testing-library/react";
import React, { Suspense } from "react";
import { RecoilRoot } from "recoil";
import { RouterProvider } from "react-router";
import { createBrowserRouter } from "react-router-dom";
import { QueryClientProvider } from "@tanstack/react-query";
import { queryClient } from "../../global/reactQuery";

export function userEventSetup(
  renderTree: {
    path: string;
    jsx: React.ReactNode;
  }[],
  loader: () => {},
): RenderResult & {
  user: UserEvent;
} {
  const renderTreeArray = renderTree.map((tree) => {
    return {
      path: tree.path,
      loader: loader,
      element: tree.jsx,
    };
  });
  const router = createBrowserRouter(renderTreeArray);

  return {
    user: userEvent.setup(),
    ...render(
      <QueryClientProvider client={queryClient}>
        <RecoilRoot>
          <Suspense fallback={<>로딩...</>}>
            <RouterProvider router={router} />
          </Suspense>
        </RecoilRoot>
      </QueryClientProvider>,
    ),
  };
}
```

작성한 `utils` 파일의 `userEventSetup`를 import하여 테스트 코드에 적용합니다. `describe` 메소드를 사용하면, 여러 테스트 코드들을 그룹 단위로 묶어 관리할 수 있습니다.

&nbsp;

#### 테스트 옵션 (단일 테스트 단위에도 적용 가능)

1. describe.**skip** : skip이 붙은 테스트 그룹은 실행하지 않습니다.
2. describe.**only** : only가 붙은 테스트 그룹만 실행합니다. (파일 내에서만 적용)

> /src/tests/Sample.test.tsx

```typescript
import { expect, it, vi } from "vitest";
import { screen } from "@testing-library/react";
import { userEventSetup } from "./utils/utils";
import { useState } from "react";

// loader 인자로 전달할 mock value 값
const emptyMockUserLoaderData = vi.fn().mockReturnValue({
  result: {},
});

function SampleComponent() {
  const [value, setValue] = useState(false);
  return (
    <>
      <h1>샘플 코드입니다.</h1>
      <button onClick={() => setValue(!value)}>샘플 버튼</button>
      {value && <>클릭</>}
    </>
  );
}

describe("샘플 테스트 그룹", () => {
  it("heading이 존재하는지 검증", async () => {
    userEventSetup(
      [{ path: "/", jsx: <></> }],
      emptyMockUserLoaderData
    );
    const sampleHeading = await screen.findByRole("heading", {
      name: "샘플 코드입니다.",
    });
    expect(sampleHeading).toBeInTheDocument();
  });

  it("버튼이 동작하는지 검증", async () => {
    const { user } = userEventSetup(
      [{ path: "/", jsx: <></> }],
      emptyMockUserLoaderData
    );

    const buttonElement = await screen.findByRole("button", {
      name: "샘플 버튼",
    });
    await user.click(buttonElement);

    const queryText = screen.queryByText("클릭");
    expect(queryText).toBeInTheDocument();
  });
});
```

#### 코드 설명

1. `utils` 파일에서 `userEventSetup` 메소드를 불러와서 `path와` `jsx` 로 이루어진 객체 배열을 첫 번째 인자로, `mockUserData`를 인자로 전달해서 `user` 응답값을 가져옵니다.
2. 이 컴포넌트는 **샘플 버튼**이라는 버튼 요소를 클릭하여 `value`값이 `true`인 경우 **클릭** 이라는 문자열이 표시됩니다.
3. 비동기 작업으로 `user.click()`메소드의 인자로 버튼 요소를 할당합니다.
4. 이벤트를 발생 시킨 뒤, "**클릭**" 이라는 텍스트가 화면에 존재하는지 검증합니다.

테스트를 돌려보면 이전과는 다르게 2개의 테스트가 실패했다고 표시됩니다. `render` 메소드의 인자로 `빈 fragment`를 전달했기 때문인데요. 이번에는 `<SampleComponent />` 를 전달해보면 테스트가 성공하게 됩니다.

&nbsp;

![test failes](/blog/assets/posts/asset-tdd-result-of-sample-describe-group-fail.png)*테스트 실패*{: .caption}

&nbsp;

![test passed](/blog/assets/posts/asset-tdd-result-of-sample-describe-group-passed.png)*테스트 성공*{: .caption}

&nbsp;

이 때 코드를 자세히 보시면 이전 글에서와는 다르게 `getByRole`이 아닌 `findByRole`을 사용했는데요. 비동기 작업으로 인해 즉시 렌더링되지 않는 요소의 경우, `await`를 활용한 `findBy***` 메소드를 활용해야 합니다.

&nbsp;

![type of queries](/blog/assets/posts/asset-tdd-rtl-query-table.png)*테스트의 종류 -- 출처 : testing library ([링크](https://testing-library.com/docs/queries/about/#types-of-queries){:target="_blank"})*{: .caption}

&nbsp;

## 3. MSW로 mocking handler 설정하기

**[MSW(Mock Service Worker)](https://mswjs.io){:target="_blank"}**는 코드 상에 정의된 실제 API 주소로 응답을 주고받는게 아닌, 임의로 설정한 응답 값을 사용할 수 있도록 도와주는 라이브러리 입니다. 사용하려면 먼저 라이브러리를 추가합니다.

```shell
app-name$ app-name$ npm install --save-dev msw
```

그 다음으로 **tests** 폴더 하위에 **mocks** 폴더를 생성한 뒤, `server.ts` 파일과 `handlers.ts` 파일을 각각 생성해줍니다.

> /src/tests/mocks/handlers.ts

```typescript
/* eslint-disable @typescript-eslint/no-unused-vars */
import { http, HttpResponse, HttpHandler } from "msw";

export const handlers: HttpHandler[] = [
  http.get(
    "http://localhost:5001/api/test",
    async () => {
      return HttpResponse.json("클릭");
    },
  ),
];
```

> /src/tests/mocks/server.ts

```typescript
import { setupServer } from "msw/node";
import { handlers } from "./handlers";

export const server = setupServer(...handlers);
```

이제 `setupTest.js` 파일에도 관련 설정을 추가합니다.

> /src/setupTest.js

```javascript
import "@testing-library/jest-dom";
import "@testing-library/react";

import { cleanup } from "@testing-library/react";
import { beforeEach, beforeAll, afterEach, afterAll } from "vitest";
import { server } from "./src/tests/mocks/server";

beforeAll(() => server.listen()); // mock server가 API 요청을 intercept 할 수 있도록 설정

beforeEach(() => {
  cleanup(); // 이전 테스트에서 실행된 코드를 초기화하는 함수
})

afterEach(() => server.resetHandlers()); // handler를 초기화

afterAll(() => server.close()); // mock server 종료
```

마지막으로, 테스트 코드의 샘플 컴포넌트에 적용해보겠습니다. "클릭"이라는 텍스트 값을 API 응답값으로 받아와서 표시하도록 코드를 수정합니다. 테스트 코드에 `screen.debug()` 메소드를 추가하면, 테스트 결과에 어떤 요소들이 렌더링 되었는지 확인할 수 있습니다.

> /src/tests/Sample.test.tsx

```typescript
import { expect, it, vi } from "vitest";
import { logRoles, screen } from "@testing-library/react";
import React, { useEffect, useState } from "react";
import { userEventSetup } from "./utils/utils";
import axios from "axios";

function SampleComponent() {
  const [value, setValue] = useState(false);
  const [result, setResult] = useState("");

  useEffect(() => {
    axios.get("http://localhost:5001/api/test").then((response) => {
      setResult(response.data);
    });
  }, []);

  return (
    <>
      <h1>샘플 코드입니다.</h1>
      <button onClick={() => setValue(!value)}>샘플 버튼</button>
      {value && <>{result}</>}
    </>
  );
}

describe("샘플 테스트 그룹", () => {
  ...
  it("버튼이 동작하는지 검증", async () => {
    ...
    screen.debug();
  })
  
});
```

&nbsp;

![debug mode](/blog/assets/posts/asset-tdd-result-of-sample-test-debug.png)*mock handler에서 설정한 응답값이 표시*{: .caption}

&nbsp;

이번 글에서는 테스트 시나리오와 환경 설정, 그리고 MSW를 활용한 API mocking에 대해서 다뤄보았습니다. 이어지는 글에서는 마지막으로 애플리케이션 테스트 시나리오에 기반한 실제 테스트 코드를 작성해보도록 하겠습니다. 🥰


<!-- 
&nbsp;

## 4. 로그인 테스트 시나리오 검증하기

먼저 **Vite**를 사용하여 프로젝트를 생성해줍니다.

&nbsp;

## 5. 지출 내역, 자산 조회 시나리오 검증하기

그럼 이제 간단한 샘플 테스트 코드를 작성해보겠습니다. 테스트 코드를 작성할 파일명은 `*.test.tsx` 형태로 **.test**를 붙여주어야 합니다. 테스트 코드는 기본적으로 **it** 명령어로 테스트 단위를 구분하며, **render** 메소드로 테스트할 컴포넌트를 불러오며, **expect** 메
-->

&nbsp;

#### 참고하면 좋은 링크

- Vitest로 테스트하기 : [링크](https://velog.io/@vrdhan212/Vitest%EB%A1%9C-%ED%85%8C%EC%8A%A4%ED%8A%B8%ED%95%98%EA%B8%B0){:target="_blank"}
- TDD 관련 장/단점 : [링크](https://blog.barogo.io/%EA%B0%9C%EB%B0%9C%EC%9D%B8%ED%84%B4-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EB%B0%A9%EC%8B%9D%EC%97%90-%EB%8C%80%ED%95%9C-%EC%A0%81%EC%A0%88%EC%84%B1-%EB%B0%8F-%EA%B3%A0%EB%AF%BC-with-tdd-6e19d3fb32bd){:target="_blank"}

&nbsp;

> 태그 Tag : #react #vitest #react-testing-library #vite #tdd



