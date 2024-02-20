---
title: Loading Indicator로 제품 사용성 강화하기
updated: 2024-01-22 12:00
---

> '가계부를 부탁해' 프로젝트를 진행하며 작성한 내용입니다.

[![project-logo](https://github.com/Ji-hoon/Home-accountant/raw/master/client/public/img-logo.png)](https://github.com/Ji-hoon/Home-accountant)

&nbsp;

새롭게 개인 프로젝트를 진행하면서 다뤘던 기술들에 대해서 기록을 남겨보려 합니다. 오늘은 세 번째 주제로, Loading Indicator를 활용하여 제품의 사용성을 강화하는 내용을 다뤄보려 합니다.

> 프로젝트 보러가기 👉 : [링크](http://35.231.16.39/)

&nbsp;

## 목차

[1. 로딩 인디케이터와 사용자 경험](#1-로딩-인디케이터와-사용자-경험)

[2. Loader 라이브러리 선택하기](#2-loader-라이브러리-선택하기)

[3. 데이터를 불러오는 중일 때 로딩 인디케이터 구현하기](#3-데이터를-불러오는-중일-때-로딩-인디케이터-구현하기)

[4. 데이터를 보내는 중일 때 로딩 인디케이터 구현하기](#4-데이터를-보내는-중일-때-로딩-인디케이터-구현하기)

&nbsp;

---

&nbsp;

## 1. 로딩 인디케이터와 사용자 경험

![loading indicators](https://media.licdn.com/dms/image/D4D12AQGOVO3YnUK39w/article-inline_image-shrink_1000_1488/0/1689576267966?e=1714003200&v=beta&t=VJCU8ZRHhfd4GIOgPcL_0WoFuCzqmQpjBt8h8xYxLEk)*다양한 타입의 loading indicator -- [출처](https://www.linkedin.com/pulse/beyond-loading-bar-elevating-user-experience-dynamic-progress-murthy/)*{: .caption}

&nbsp;

웹/모바일 애플리케이션 중에서 **로딩 인디케이터 Loading Indicator**가 없는 제품은 상상할 수 없을 만큼 필수적으로 사용되고 있습니다. 과거처럼 서버와 클라이언트가 통신중일 때 무조건 화면 전체를 덮는 형태의 로딩 인디케이터를 노출하는 방식은 최근의 SPA 형태의 애플리케이션에서는 사용되지 않습니다. 이는 사용자 경험을 망칠 수 있고, 또 사용성을 해칠 수 있기에 아래와 같은 최소한의 가이드라인에 맞춰 구현해야 합니다.

1. 특정 데이터를 로딩중 이더라도 동작이 제한되어야 하는 경우(중복 요청, 에러를 유발할 가능성이 있는 행동)를 제외하고 제품의 다른 기능에 대한 접근이 제한되어서는 안됩니다.
2. 로딩중에 다른 기능에 대한 접근을 제한하는 경우, 명시적으로 어떤 기능으로 인해 제한되었는지 알려줄 수 있어야 합니다.
3. 변경이 잦은 데이터의 경우, 로딩 인디케이터를 노출하는 영역을 최소화해야 합니다.

> 참고 - 로딩 피드백 패턴 : [링크](https://pencilandpaper.io/articles/ux-pattern-analysis-loading-feedback/) & 간략한 번역 : [링크](https://news.hada.io/topic?id=10588)

이전 포스팅에서 tanstack query를 사용한 무한 스크롤을 구현해면서 적용했던 Skeleton UI도 로딩 인디케이터의 한 종류 인데요. 이번 포스팅에서는 **로딩 스피너 Loading Spinner** 와 **투명도 Opacity** 를 사용해서 로딩 인디케이터를 구현해보도록 하겠습니다.

&nbsp;

## 2. Loader 라이브러리 선택하기

React 프로젝트에서 사용할 loader 라이브러리를 검색해보면 굉장히 많은 패키지를 찾을 수 있습니다. 하지만 제품의 사용자 인터페이스 디자인에 적합한 Loader 를 찾다보니 loader 만을 제공하는 패키지 보다는 컴포넌트 단위로 제공하는 패키지가 적합하다고 판단하였고, 사이즈와 컬러, 속도를 튜닝하기 용이한 [rsuite](https://rsuitejs.com/components/loader/)의 **Loader**를 적용했습니다.

```typescript
import { useExpenses } from "../Expenses/Expenses.hooks";
import { Loader } from "rsuite";
import styled from "styled-components";
import { SIZES, COLORS } from "../../../../global/constants";

// eslint-disable-next-line react-refresh/only-export-components
export default function Expenses_Amounts() {
  const { pages, fetchStatus } = useExpenses();
  const amounts = pages[0]?.amounts;

  return (
    <ValueWrapper>
      {fetchStatus === "fetching" && <Loader size="sm" />}
      <span>-{amounts.toLocaleString()}원</span>
    </ValueWrapper>
  );
}

// eslint-disable-next-line react-refresh/only-export-components
export const ValueWrapper = styled.div`
  display: flex;
  align-items: center;
  gap: 8px;

  & .rs-loader-spin {
    width: ${SIZES.XL}px;
    height: ${SIZES.XL}px;

    &::before,
    &::after {
      width: inherit;
      height: inherit;
      border-color: ${COLORS.GRAY_01_OVERAY};
    }
    &:after {
      border-color: ${COLORS.GRAY_07_OVERAY} transparent transparent;
    }
  }
`;
```
위 코드는 지출 비용을 표시하는 `Expenses_Amounts` 컴포넌트의 코드 입니다. `rsuite` 설치 후 `Loader` 컴포넌트만 불러온 다음, 지출 내역 화면 헤더 영역에 배치하고 컬러와 사이즈를 디자인에 맞게 조정해줍니다. 

&nbsp;

`useInfiniteQuery` 가 반환하는 `fetchStatus` 값이 제공하는 타입을 살펴보면 아래와 같습니다. 각각 **불러오는중**, **중단됨** , **쉬는중** 상태를 나타냅니다.

```typescript
type FetchStatus = 'fetching' | 'paused' | 'idle';
```

이 `fetchStatus` 값이 `fetching` 인 경우에만 **Loader**를 표시하여 데이터가 로딩중이라는 상태를 사용자에게 알려줄 수 있습니다.

&nbsp;

## 3. 데이터를 불러오는 중일 때 로딩 인디케이터 구현하기

위에서 언급한 데이터가 로딩중 (API 통신)이라는 상태를 좀 더 쪼개어 보면 **① 데이터를 불러오는 상태**와 **② 데이터를 보내는 상태**로 나눠볼 수 있습니다. 먼저 데이터를 불러오는 상태에 대한 로딩 인디케이터를 고도화 해보도록 하겠습니다. 

아래 코드는 지출 내역 리스트를 표시하는 `Expenses_List` 컴포넌트의 코드 중 일부입니다.

```typescript
function Expenses_List() {
  ...

  const { pages, setTarget, hasNextPage, fetchStatus, isFetchingNextPage } =
    useExpenses();

  ...

  return (
    <ul
      className={
        fetchStatus === "fetching" && !isFetchingNextPage ? "fetching" : ""
      }
    >
    ...
    </ul>
  )

  ...
  & > ul{
      min-height: calc(100vh - 162px);
      margin: -${SIZES.LG * 3 + 84}px 0 0;
      padding: 0 0 84px;
      list-style: none;
      display: flex;
      flex-direction: column;
      gap: ${SIZES.XXS / 3}px;

      -webkit-transition: opacity 150ms ease-out;
      transition: opacity 150ms ease-out;

      & li.skeleton-item {
        margin-bottom: -70px;
      }

      &.fetching {
        opacity: 0.3;
        pointer-events: none;
      }
    }
  ...
}
```
위 코드에서는 `fetchStatus`값이 `fetching` 이고, `isFetchingNextPage`값이 `false` 라면 className에 `fetching` 이라는 클래스를 할당하고 있습니다. 새롭게 사용한 `isFetchingNextPage` 값은 `useInfiniteQuery` 의 리턴값으로 `fetchNextPage` 메소드가 실행중인지 여부에 따라 `true` 또는 `false` 값을 반환합니다. 

따라서 최초로 데이터를 불러오는 중일 때만 `fetching` 클래스가 할당되고, 투명도를 `0.3`으로 처리함으로써 해당 영역의 데이터가 갱신되고 있음을 사용자에게 알려줄 수 있습니다. 추가적으로 `pointer-events` 값도 `none`으로 할당해줌으로써 조작할 수 없는 상태임을 확실하게 고지하고 있습니다.

&nbsp;

![opacity loading indicator](/blog/assets/posts/asset-opacity-loading-indicator.gif)

&nbsp;

## 4. 데이터를 보내는 중일 때 로딩 인디케이터 구현하기

tanstack query에서 제공하는 `useIsMutating` 훅을 사용하면 `isMutating` 이라는 `number` 값을 리턴합니다. 이 값은 현재 애플리케이션에서 **몇 개의 mutation query가 fetching 중인 지**를 알려줍니다. 아무것도 없다면 `0` 을, 1개인 경우 `1` 을 리턴합니다. 이 값을 활용해서 데이터를 보내는 중일 때의 로딩 상태를 구현해보도록 하겠습니다.

> Dialog 컴포넌트

```typescript
import { useIsMutating } from "@tanstack/react-query";

function Dialog() {

  const dialog = useRecoilValue(currentDialogAtom);
  const { hideDialog } = useHandleDialog();
  const { dialogRef, onSubmit } = useDialogSubmit();

  const { handleSubmit } = useForm<InputFormType>();
  const setModalIndex = useSetRecoilState(modalIndexAtom);

  const isMutating = useIsMutating();

  function handleCancleDialog(event: React.SyntheticEvent, index: number) {
    event.preventDefault();
    if (isMutating < 1) hideDialog({ order: index });
  }

  return (
    <DialogPortal>
      <div ref={dialogRef}>
        {dialog.isOpen &&
          dialog.content.length > 0 &&
          dialog.content.map((item, index) => (
            <ModalContainer key={index} className="modal-container">
              <BackdropModal
                $index={index}
                $maxindex={dialog.content.length}
                onClick={(event: React.SyntheticEvent) =>
                  handleCancleDialog(event, index)
                }
              />
              <ModalLayoutContainer
                $index={index}
                $type={item.type}
                onSubmit={handleSubmit(onSubmit)}
              >
                <section className="modal-header">
                  <h3>{item.title}</h3>
                  <Button_Icontype
                    onClick={(event: React.SyntheticEvent) =>
                      handleCancleDialog(event, index)
                    }
                  >
                    <FiX />
                  </Button_Icontype>
                </section>

                <section className="modal-contents">
                  <FormListLayout
                    type={item.type}
                    layout={item.layout}
                    $processing={
                      index === dialog.content.length - 1 && isMutating > 0
                    }
                  />
                </section>
                <section className="modal-actions">
                  <Button_Boxtype
                    onClick={(event: React.SyntheticEvent) =>
                      handleCancleDialog(event, index)
                    }
                  >
                    {LABELS.LABEL_CANCEL}
                  </Button_Boxtype>
                  <Button_Boxtype
                    processing={
                      index === dialog.content.length - 1 && isMutating > 0
                    }
                    onClick={() => {
                      setModalIndex(index);
                    }}
                    type={TYPES.SUBMIT}
                    isAlert={
                      item.title.includes(LABELS.LABEL_DELETE) ? "true" : ""
                    }
                  >
                    {item.title}
                  </Button_Boxtype>
                </section>
              </ModalLayoutContainer>
            </ModalContainer>
          ))}
      </div>
    </DialogPortal>
  )
}
```

> Button_BoxType 컴포넌트

```typescript
import styled from "styled-components";
import { COLORS, SIZES, TYPES } from "../../global/constants";
import { Loader } from "rsuite";

export default function Button_Boxtype({
  children,
  onClick,
  type, //className으로 활용
  disabled,
  isAlert,
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  title,
  processing,
}: {
  children: React.ReactElement | string;
  onClick?: (e: React.SyntheticEvent) => void;
  type?: string;
  disabled?: boolean | undefined;
  isAlert?: string | undefined;
  title?: string | undefined;
  processing?: boolean;
}) {
  return (
    <>
      {type && type === TYPES.SUBMIT && typeof children === "string" && (
        <BoxtypeSubmitButton
          type={type}
          className={type}
          onClick={onClick}
          disabled={disabled || processing}
          $alert={isAlert}
        >
          {processing && <Loader size="sm" />}
          {children}
        </BoxtypeSubmitButton>
      )}
      {type !== TYPES.SUBMIT && (
        <BoxtypeButton
          className={type}
          onClick={onClick}
          disabled={disabled}
          $alert={isAlert}
          id={title}
        >
          {children}
        </BoxtypeButton>
      )}
    </>
  );
}
```

위 코드는 모달을 다루는 `Dialog` 컴포넌트와 `Button_BoxType` 컴포넌트의 코드입니다. (스타일링 내용은 제외) 데이터를 보내는 일반적인 경우라면 리소스를 추가, 수정 또는 삭제를 요청하는 `POST` 나 `PUT`, `DELETE` api 요청일텐데요. 이 때 `isMutating` 값을 적용해서 로딩 인디케이터 표시나 동작을 제한하는 코드를 적용했습니다. (`useMutation` 훅은 `add`, `update`, `delete` 요청에만 적용된 상태입니다.)

1. isMutating 값이 0보다 크다면, 그리고 현재 dialog의 index 값이 dialog 길이값 보다 1작다면 (가장 마지막 순서라면) Button_BoxType 컴포넌트의 processing props로 true 값을 넘겨줍니다.
2. Button_BoxType 컴포넌트에서는 processing 값이 true인 경우 Loader를 버튼 레이블 좌측에 표시하게 됩니다.
3. 현재 모달을 닫는 handleCancleDialog 함수 실행 시에도 마찬가지로 isMutating 값이 1보다 작을 경우에만 hideDialog({ order: index }) 메소드를 실행하는 조건을 추가하여 데이터를 보내는 상태에서는 해당 액션을 제한하고 있습니다.


&nbsp;

![mutating handling](/blog/assets/posts/asset-mutating-loading-indicator.gif)

&nbsp;



> 참고하면 좋은 링크 - 스켈레톤을 무조건 보여주는게 도움이 될까? : [링크](https://tech.kakaopay.com/post/skeleton-ui-idea/) / UX와 성능 개선 방법 : [링크](https://fe-developers.kakaoent.com/2022/220120-ux-and-perf-in-kakaowebtoon/)




> 태그 Tag : #react #typescript #tanstack-query #react-query #loading-indicator #ux #usability