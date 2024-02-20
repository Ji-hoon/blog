---
title: Tanstack query를 활용한 무한 스크롤 UI 구현하기
updated: 2024-01-19 12:00
---

> '가계부를 부탁해' 프로젝트를 진행하며 작성한 내용입니다.

[![project-logo](https://github.com/Ji-hoon/Home-accountant/raw/master/client/public/img-logo.png)](https://github.com/Ji-hoon/Home-accountant){:target="_blank"}

&nbsp;

새롭게 개인 프로젝트를 진행하면서 다뤘던 기술들에 대해서 기록을 남겨보려 합니다. 오늘은 두 번째 주제로, 서버 상태를 손쉽게 관리할 수 있는 기능을 제공하는 `Tanstack query` (a.k.a react query) 를 사용해서 무한 스크롤을 구현해보도록 하겠습니다.

> 프로젝트 보러가기 👉 : [링크](http://35.231.16.39/)

&nbsp;

## 목차

[1. Tanstack query 소개](#1-tanstack-query-소개)

[2. useInfiniteQuery와 Intersection observer](#2-useinfinitequery와-intersection-observer)

[3. 무한 스크롤 구현하기](#3-무한-스크롤-구현하기)

[4. Skeleton UI 구현하기](#4-skeleton-ui-구현하기)

&nbsp;

---

&nbsp;

## 1. Tanstack query 소개

![dropdown UI pattern](https://developers.google.com/static/search/docs/images/ecom-pagination-load-more-infinite-scroll.png?hl=ko)*대량의 데이터 표시 방식 3가지 -- 출처 : google developers ([링크](https://developers.google.com/search/docs/specialty/ecommerce/pagination-and-incremental-page-loading?hl=ko){:target="_blank"})*{: .caption}

&nbsp;

현대의 웹 또는 모바일 애플리케이션에서 사용되는 데이터 표시 방식은 일반적으로 페이지네이션, 더 불러오기, 무한 스크롤 형태가 존재하는데요. 다양한 제품들마다 제공하고자 하는 사용자 경험이 다르고 각 방식마다 장단점이 존재하지만, 검색 목적이 아닌 탐색 목적인 경우 직관적인 무한 스크롤 형태가 많이 사용되고 있습니다.

&nbsp;

React에서도 별도 라이브러리 없이 무한 스크롤 구현이 가능하지만, 강력한 서버 상태 관리 기능, 캐싱, 동기화 기능을 제공하는 tanstack query를 사용하면 보다 손쉽게 기능을 구현할 수 있습니다.

> 참고 - Tanstack query tutorial : [링크](https://github.com/ssi02014/react-query-tutorial){:target="_blank"} / v5 변경사항 : [링크](https://www.moonkorea.dev/React-TanStack-Query-v5-%EC%82%B4%ED%8E%B4%EB%B3%B4%EA%B8%B0-(%EB%A6%AC%EC%95%A1%ED%8A%B8%EC%BF%BC%EB%A6%AC)){:target="_blank"} / tanstack query 없이 infinite 스크롤 구현 : [링크](https://medium.com/@itsanuragjoshi/pagination-vs-infinite-scroll-vs-load-more-data-loading-ux-patterns-in-react-cccd261d3984){:target="_blank"}

&nbsp;

## 2. useInfiniteQuery와 Intersection observer

아래 코드는 [공식 가이드 문서](https://tanstack.com/query/v5/docs/framework/react/guides/infinite-queries#example){:target="_blank"} 에서 제공하는 `useInfiniteQuery` 샘플 코드입니다.

```typescript
import { useInfiniteQuery } from '@tanstack/react-query'

function Projects() {
  const fetchProjects = async ({ pageParam }) => {
    const res = await fetch('/api/projects?cursor=' + pageParam)
    return res.json()
  }

  const {
    data,
    error,
    fetchNextPage,
    hasNextPage,
    isFetching,
    isFetchingNextPage,
    status,
  } = useInfiniteQuery({
    queryKey: ['projects'],
    queryFn: fetchProjects,
    initialPageParam: 0,
    getNextPageParam: (lastPage, pages) => lastPage.nextCursor,
  })

  return status === 'pending' ? (
    <p>Loading...</p>
  ) : status === 'error' ? (
    <p>Error: {error.message}</p>
  ) : (
    <>
      {data.pages.map((group, i) => (
        <React.Fragment key={i}>
          {group.data.map((project) => (
            <p key={project.id}>{project.name}</p>
          ))}
        </React.Fragment>
      ))}
      <div>
        <button
          onClick={() => fetchNextPage()}
          disabled={!hasNextPage || isFetchingNextPage}
        >
          {isFetchingNextPage
            ? 'Loading more...'
            : hasNextPage
              ? 'Load More'
              : 'Nothing more to load'}
        </button>
      </div>
      <div>{isFetching && !isFetchingNextPage ? 'Fetching...' : null}</div>
    </>
  )
}
```

샘플 코드에서는  `initialPageParam`을 `0`으로 설정하고, 한 번 데이터를 불러올 때마다 `cursor` 값을 증가시켜 `fetchNextPage` 메소드를 실행하면서 다음 순서의 데이터를 불러오게 됩니다. 여기서는 다음 페이지를 불러오기 위한 트리거로 버튼 클릭 이벤트를 사용했지만, 무한 스크롤을 위해서는 **불러온 데이터의 마지막 요소가 화면 하단**을 지날때, `fetchNextPage` 메소드를 트리거할 수 있어야 합니다.

이 때 사용되는 메소드가 `intersection observer`라는 브라우저 내장 API입니다. 이 API는 요소들이 화면에 보이는 정도를 감지하고 이벤트를 발생시키는 기능을 제공하는데, 이를 토대로 언제 `fetchNextPage`를 호출할 지 결정할 수 있습니다.

> 참고 - Mozilla 공식 문서 : [링크](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API){:target="_blank"}

아래는 샘플 코드 입니다. 

```typescript
import { useEffect, useRef } from "react";

function App() {
  const targetRef = useRef(null);

  useEffect(() => {
    const observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          console.log('요소가 화면에 보임');
          // 이 때 fetchNextPage를 호출
        } else {
          console.log('요소가 화면에서 사라짐');
        }
      });
    });

    if (targetRef.current) {
      observer.observe(targetRef.current);
    }

    return () => {
      if (targetRef.current) {
        observer.unobserve(targetRef?.current);
      }
    };
  }, []);

  return (
    <>
      <div style={{ width: "100%",height: "110vh", background: "#333"}}></div>
      <div ref={targetRef} style={{ width: "100%",height: '100px', background: 'lightblue' }}></div>
      <div style={{ width: "100%",height:"110vh", background: "#333"}}></div>
    </>
  );
}

export default App
```

`ref` 속성을 사용하여 `targetRef`를 측정하고자 하는 요소에 할당하고, IntersectionObserver를 사용하여 해당 요소의 가시성을 감지하고 로그를 출력하게 됩니다. targetRef 앞뒤로 110vh의 높이값을 갖는 div를 배치했기 때문에 스크롤하게 될 경우 이벤트가 발생하게 됩니다.

&nbsp;

![intersection observer sample](/blog/assets/posts/asset-intersection-observer-demo.gif)


&nbsp;

## 3. 무한 스크롤 구현하기

이제 위에서 작성한 내용을 바탕으로 무한 스크롤을 구현해보도록 하겠습니다. `api`는 이전에 작성해두었던 `/api/expenses` 를 사용합니다. 이 api는 `search query`로 `owner`, `cursor`, `limit`, `startDate`, `endDate`, `currentGroupId`를 전달하고 데이터를 응답 받습니다.

```typescript
import { useInfiniteQuery } from "@tanstack/react-query";
import { useEffect, useRef } from "react";
import ListItem_ExpenseType, { ExpenseType } from "./components/ExpenseList";
import React from "react";

function App() {

  const limit = 7;

  const fetchExpenses = async ({ pageParam }:{ pageParam: number }) => {
    const res = await fetch(`http://localhost:5001/api/expenses/?owner&cursor=${pageParam}&limit=${limit}&startDate=2024-01-01&endDate=2024-01-31&currentGroupId=65b73d1d95fd2333931df1a2`);
    return res.json()
  }

  const {
    data,
    error,
    fetchNextPage,
    hasNextPage,
    isFetching,
    isFetchingNextPage,
    status,
  } = useInfiniteQuery({
    queryKey: ['expenses'],
    queryFn: fetchExpenses,
    initialPageParam: 0,
    getNextPageParam: ( lastPage, allPages ) => {
      const allPageLength = allPages.flatMap(page => page).length;
      console.log("lastPage :", lastPage);
      console.log( allPageLength );
      if(lastPage.length === 0) return undefined;
      return lastPage.length !== 0 && allPageLength+limit;
    },
  })

  console.log({ cursor: data?.pages.flatMap(page => page).length, hasNextPage, isFetching, isFetchingNextPage});
  
  const targetRef = useRef(null);

  useEffect(() => {
    const observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          console.log('요소가 화면에 보임');
          if(hasNextPage) fetchNextPage();
        } else {
          console.log('요소가 화면에서 사라짐');
        }
      });
    });

    if (targetRef.current) {
      observer.observe(targetRef.current);
    }
  }, [data]);

  return status === 'pending' ? (
    <p>Loading...</p>
  ) : status === 'error' ? (
    <p>Error: {error.message}</p>
  ) : (
    <>
      {data.pages.map((page, index) => (
        <React.Fragment key={index}>
          {page.map((item: ExpenseType & { _id: string; }, index: number) => (
            <ListItem_ExpenseType key={index} $item={item} />
          ))}
        </React.Fragment>
      ))}
      {hasNextPage && (
        <div ref={targetRef} style={{ width: "100%",height: '100px', background: 'lightblue' }}>
          <div>{isFetching && isFetchingNextPage ? 'Fetching...' : null}</div>
        </div> 
      )}
    </>
  );
}

export default App
```
`useInfiniteQuery`를 통해서 `data`, `error`, `fetchNextPage`, `hasNextPage`, `isFetching`, `isFetchingNextPage`, `status` 등의 값을 가져올 수 있습니다. 이 값들을 이용해서 fetching 상태에 따라 아래와 같은 UI 핸들링을 구현합니다.

1. status 값이 "pending"로 반환되었다면, 화면에 Loading... 이라는 메시지를 출력합니다.
2. status 값이 "error"로 반환되었다면, 화면에 Error: {error.message} 를 출력합니다.
3. status 값이 "pending" 또는 "error"가 아니라면, data의 pages 배열과 하위의 page 배열을 돌며 ListItem_ExpenseType 컴포넌트를 렌더링합니다.
4. hasNextPage 값이 true라면, 리스트 컴포넌트 하단에 intersectionObserver로 트리거 엘리먼트를 표시합니다. (hasNextPage 값이 false라면 화면에 렌더링되지 않습니다.)
5. 이 때, isFetching 값과 isFetchingNextPage 값이 모두 true 라면 트리거 엘리먼트 내부에 Fetching... 이라는 메시지를 출력합니다.
6. fetchNextPage 실행 시, lastPage의 길이가 0이 아니라면 allPages 배열에 존재하는 모든 배열의 길이에 limit 값을 합산한 값을 리턴하여 다음 pageParam 값으로 활용합니다.

&nbsp;

![infinite query sample](/blog/assets/posts/asset-infinite-sample.gif)

&nbsp;

## 4. Skeleton UI 구현하기

트리거 요소는 다음에 불러올 페이지가 있다면 표시하게 되는데요, 그래서 로딩중임을 알리는 **로딩 엘리먼트**나 해당 리스트의 디자인을 dummy화한 **Skeleton UI**를 만들어 활용하게 됩니다. 본 프로젝트에서는 **Skeleton UI**를 만들어 입혀보도록 하겠습니다.

```typescript
/* eslint-disable react-refresh/only-export-components */
import styled from "styled-components";
import { ListItemContainer } from "../compound/ListItem.expenseType";
import { SIZES, COLORS } from "../../global/constants";

export default function Skeleton_ExpenseListItem() {
  return (
    <DummyListContainer>
      <div></div>
      <div className="list-info">
        <h4 className="businessName">업장명슬롯</h4>
        <p className="date">날짜슬롯</p>
      </div>
      <div className="expense-value">
        <p className="owner">멤버 슬롯</p>
        <h4 className="amounts">금액 슬롯</h4>
      </div>
    </DummyListContainer>
  );
}

const DummyListContainer = styled(ListItemContainer)`
  //pointer-events: none;

  & div:nth-child(1) {
    width: ${SIZES.LG}px;
    height: ${SIZES.LG}px;
    background-color: ${COLORS.GRAY_01_OVERAY};
    border-radius: 4px;
    -webkit-animation: blink 1.5s 50ms ease-in infinite;
    animation: blink 1.5s 50ms ease-in infinite;
  }
  & h4 {
    height: ${SIZES.LG}px;
    font-size: 0;
    background-color: ${COLORS.GRAY_01_OVERAY};
    border-radius: 4px;
    width: 200px;
    -webkit-animation: blink 1.5s 200ms infinite;
    animation: blink 1.5s 200ms infinite;
  }

  & p {
    height: ${SIZES.SM}px;
    font-size: 0;
    background-color: ${COLORS.GRAY_01_OVERAY};
    border-radius: 4px;
    width: 100px;
    -webkit-animation: blink 1.5s 100ms ease-in infinite;
    animation: blink 1.5s 100ms ease-in infinite;
  }

  & .expense-value {
    & h4 {
      width: 120px;
    }
    & p {
      width: 80px;
    }
  }

  @keyframes blink {
    0% {
      opacity: 0.45;
    }
    50% {
      opacity: 1;
    }
    100% {
      opacity: 0.45;
    }
  }
`;
```
`Skeleton_ExpenseListItem` 컴포넌트에는 기존 컴포넌트 구조와 동일한 dom을 구성하고, `animation`과 `animation delay`를 활용해서 깜빡이는 효과를 구현합니다. 완성된 컴포넌트로 기존 코드를 대체하면 끝입니다.

```typescript
function App() {
  ...
      {hasNextPage && (
        <Skeleton_ExpenseListItem ref={targetRef} />
      )}
    </>
  );
}

export default App
```

&nbsp;

![infinite scroll final](/blog/assets/posts/asset-infinite-scroll-final.gif)

&nbsp;


> 태그 Tag : #react #typescript #tanstack-query #react-query #infinitequery