---
title: 좋은 코드 작성하기
updated: 2024-08-15 12:00
---

> ERP 제품 '로지메이트'의 프론트엔드 개발을 진행하며 작성한 내용입니다.

![redesign preview](/blog/assets/logimate/HFS_LOGO.png)

&nbsp;

저는 24년 하반기부터 로지메이트라는 ERP(Enterprise Resource Planning, 전사적 자원 관리) 제품의 프론트엔드 개발을 담당하고 있습니다. 입사 후 운영 개발 건 들을 진행하면서 좋은 코드에 대해 고민하고 실제 프로뎍션 코드에도 반영했던 내용들을 공유해보고자 합니다.

&nbsp;

## 목차

[1. 중복 코드 제거하기](#1-중복-코드-제거하기)

[2. 가독성 확보하기](#2-가독성-확보하기)

[3. 성능 최적화하기](#3-성능-최적화하기)

[4. 예외 처리하기](#4-예외-처리하기)

&nbsp;

---

&nbsp;

## 1. 중복 코드 제거하기

중복 코드는 무조건 제거되어야 할 안좋은 코드이고, 작성할 때부터 중복이 발생되지 않아야 하는 것은 아니지만, 3번 이상 동일한 코드가 반복해서 사용되고 있다면 중복 코드를 리팩토링을 하기에 좋은 시점이라고 생각합니다.

> iterator 활용

기존 레거시 코드에서는 Form 엘리먼트 내에 포함된 검색 조건들을 모아서 조회 결과를 요청하는 API로 GET 요청을 보낼 때 필요한 `search params`를 각각 수동으로 추가하는 형태로 처리하고 있었습니다. 이럴 경우 `searchObj`라는 객체 에 포함되는 파라미터의 이름이 바뀔 때 마다 필요한 필드들을 수동으로 변경해야 하는 수고로움이 발생했기에 이를 ES6+ 스펙인 **이터레이터(iterator)**를 활용한 코드로 개선하고, 추후 여러 컴포넌트에서 사용될 수 있도록 공통 함수로 작성 했습니다.

```typescript
// 변경전
... getResults = () => {
  let baseUrl = '/lists';
  
  baseUrl += `?paramA=${searchObj.paramA}&`;
  baseUrl += `paramB=${searchObj.paramB}&`;
  baseUrl += `paramC=${searchObj.paramC}`;
  ...
}
...

// 변경후
...
export const generateSearchParams = (searchObj) => {
  let params = [];
  const searchParams = Object.keys(searchObj);

  for(let param of searchParams) {
    url.push(`${param}=${searchParams[param]}`);
  }
  return params.join('&');
}

const getResults = () => {
  ...
  let baseUrl = '/lists';
  baseUrl += `?${generateSearchParams(searchObj)}`;
  ...
}
...
```

> Form event는 하나의 함수로 처리

설정한 검색 조건에 대한 조회 결과를 받아오기 위해 보통 Form 엘리먼트 요소를 사용합니다. 기존 레거시 코드에서도 Form 엘리먼트를 사용하고 있었지만 trigger button에도 `onClick` 이벤트에 `handleSubmit` 함수를 연결하고, Form 엘리먼트의 `onKeyDown` 이벤트에도 `handleSubmit` 함수를 연결하는 형태로 동일한 함수를 여러번 중복으로 호출하고 있었습니다. 이 부분을 Form 엘리먼트의 `onSubmit` 이벤트 하나로 핸들링할 수 있게 개선했습니다.

```typescript
// 변경전
...
const handleSubmit = (e:React.KeyboardEvent) => {
  e.preventDefault();
  getResult();
}
...
<form onKeyDown={(e) => {
    if(e.keyCode === 13) handleSubmit(e);
  }}>
  ...
  <button type="button" onClick={getResult}>검색</button>
</form>
...

// 변경후
...
const handleSubmit = (e?:React.FormEvent) => {
  e?.preventDefault();
  getResult();
}
...
<form onSubmit={handleSubmit}>
  ...
  <button type="submit">검색</button>
</form>
...
```

&nbsp;

## 2. 가독성 확보하기

여러 인원들이 같이 개발하는 프로젝트의 경우 혼자서만 보는 코드가 아니기 때문에 작성자 뿐만 아니라 해당 코드를 다른 개발자들도 해석하고 또 변경해야 하는 일이 빈번하게 일어납니다. 그렇기 때문에 코드에 대한 컨벤션을 정의하고, 또 코드의 가독성을 확보하는 일도 생산성을 높이는데 중요한 역할을 합니다.

> 변수와 함수의 이름은 의미를 잘 담을 수 있도록 작성

변수나 함수의 네이밍은 동작이나 담고 있는 값에 대해 예상 가능할 수 있는 가이드로서 활용될 수 있어야 코드의 가독성을 높여줄 수 있습니다. 네이밍이 다소 길어지더라도 코드를 보는 사람이 이해하기 쉽도록 작성해야 가독성을 확보할 수 있다고 생각합니다.

```typescript
// 변경전
const paramingUrl = () => {...};

// 변경후
const generateSearchParams = () => {...};
```

> Optional Chaining, nullish coalescing을 활용

코드를 작성하다보면 특정 로직을 수행해야 할 때 값이 있는 경우에만 실행되도록 한다던지, 값이 없을 경우에만 다른 값을 할당하는 것이 필요한 경우가 자주 있습니다. 이 때 기존에 조건문이나 스위치 구문으로 처리해야 했던 부분들을 ES6+ 문법에 추가된 **옵셔널 체이닝**, **널리시 병합 연산자** 등을 활용하면 코드의 절대적인 양을 줄여 가독성을 확보할 수 있습니다.


```typescript
// optional chaining
const initialPageCount = searchObj?.pageCount; // searchObj에 pageCount 값이 존재할 때만 initialPageCount 변수에 값을 할당

// nullish coalescing
const initialPageCount = searchObj?.pageCount ?? 1; // searchObj에 param 값이 null 또는 undefined일 때 1을 할당
```

&nbsp;

## 3. 성능 최적화하기

React의 함수형 컴포넌트의 경우, 컴포넌트 내 변수를 선언하고 특정 연산의 결과 값을 할당하는 코드가 있다면 렌더링 될 때 마다 변수의 재생성 재연산이 일어나게 됩니다. 관리가 필요한 변수라면 `useState`를 사용해서 상태관리를 하겠지만, 만약 1번만 실행되기를 원하는 코드가 있다면 `useMemo` 훅을 사용해서 의존성 배열에 선언된 값이 바뀌지 않는다면 연산을 수행하지 않도록 최적화 할 수 있습니다.

```typescript
const defaultSearchFilter = useMemo(() => {
  return {
    partnerName : userInfo?.partnerName || null;
  }
},[userInfo]);
```

&nbsp;

또한 특정 컴포넌트가 리렌더링 될 때, 렌더링에 영향을 주지 않는 자식 컴포넌트도 리렌더링되지 않도록 최적화하려면 `React.memo`를 활용해서 렌더링을 최적화 할 수 있습니다.

```typescript
// 부모 컴포넌트
export default function LoginPage () {
  ...
  return (
    <>
      <Aside />
      ...
    </>
  )
}

// 자식 컴포넌트
function Aside() {
  return (
    ...
  )
}

export default React.memo(Aside);
```

&nbsp;

## 4. 예외 처리하기

코드 작성 시 보통은 특정 코드의 실행 값이 존재할 때, 유효할 때를 상정하고 작성합니다. 하지만 현실세계에서 동작하는 코드는 값이 없을 수도 있고, 오류를 발생 시키는 값이 입력되거나 유효하지 않은 값이 입력될 수 있습니다. 이러한 예외가 발생했을 때 어떤 동작을 수행해야 하는지 예외 처리를 작성하는 것은 코드의 커버리지를 이해하는 데 중요한 지표가 될 수 있습니다.

> Early return

값이 없거나 정의되지 않은경우 리턴시켜 로직 실행 시 필요한 값을 명확하게 정의하고 오류를 방지할 수 있습니다.

```typescript
...
export const generateSearchParams = (searchObj) => {
  let params = [];
  const searchParams = Object.keys(searchObj);

  if(searchParams.length === 0) return; //searchObj에 존재하는 key가 없는 경우 early return

  for(let param of searchParams) {
    url.push(`${param}=${searchParams[param]}`);
  }
  return params.join('&');
}
...
```

> 필수 파라미터 검증

특정 데이터를 검색하기 위한 API를 요청할 때, 경우에 따라서는 필수 파라미터가 존재할 수 있습니다. 보통은 API 요청 시 필수 파라미터가 없다면 오류를 응답하도록 구현되어 있겠지만 API를 요청하는 것도 비용이 발생될 수 있기에 클라이언트 사이드에서도 필수 파라미터에 대한 검증을 수행하는 것이 좋습니다. 이러한 필수 파라미터를 검증하고 **단일 책임 원칙 (Single Responsibilty Principle)** 에 부합하는 공통 함수를 작성해서 서로 다른 Form에서 사용될 수 있도록 구현할 수 있습니다.

```typescript
type ParamType = {
  name: string;
  value: number | string;
}

export const validateSearchParams = (...params: ParamType[]) => {
  let result = false;

  for(let param of params) {
    if(!param.value || param.value === '' || param.value === undefined) {
      alert(`${param.name}은(는) 필수값입니다.`);
      result = false;
      return;
    }
    result = true;
  }

  return result;
};

// 검증 함수를 호출할 때 객체 배열을 인자로 전달
const handleSubmit = (e?:React.FormEvent) => {
  e?.preventDefault();
  const validation = validateSearchParams([
    {
      name: 'paramA',
      value: searchObj?.paramA
    },
    {
      name: 'paramB',
      value: searchObj?.paramB
    },
  ]);

  if(!validation) return; // validateSearchParams 리턴값이 false인 경우 return하여 API가 호출되지 않도록 합니다.
  getResult();
}
...
```

&nbsp;

제가 작성한 방식이 정답이다라고 할 수는 없지만, 좋은 코드에 대해서 꾸준히 공부해나가고 반영해보는 과정을 지속적으로 경험하다보면 더 나은 코드를 작성할 수 있게 될 거라고 믿습니다.  

&nbsp;

#### 참고하면 좋은 링크

- 이터러블 객체 : [링크](https://ko.javascript.info/iterable){:target="_blank"}
- 선언적인 코드 작성하기 : [링크](https://toss.tech/article/frontend-declarative-code){:target="_blank"}

&nbsp;

> 태그 Tag : #react #refactoring #es6


