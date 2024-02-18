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

[2. Portal로 사용할 DOM Element 설정하기](#2-portal로-사용할-dom-element-설정하기)

[3. Portal 바깥에서 dropdown trigger 설정하기](#3-portal-바깥에서-dropdown-trigger-설정하기)

[4. 컴포넌트의 표시 위치 계산하기](#4-컴포넌트의-표시-위치-계산하기)

[5. 드롭다운 고유 키를 사용한 분기 처리](#5-드롭다운-고유-키를-사용한-분기-처리)

&nbsp;

---

&nbsp;

## 1. React portal을 왜 써야할까?

![dropdown UI pattern](/blog/assets/posts/asset-dropdown-pattern.png)*다양한 Dropdown 패턴 -- feat. Notion, Rallit, Gmail*{: .caption}

&nbsp;

먼저 웹 애플리케이션에서 드롭다운 패턴이라함은, UI 관점에서 바라보면 **평소에는 보여지지 않지만 특정 액션에 대한 선택지를 제공하거나 부가적인 정보나 기능을 제공하려고 할 때** 사용하게 됩니다. 예컨데 **옵션 메뉴 버튼을 클릭했을 때 선택 가능한 옵션 목록을 제공하는 UI** 나 **더보기 같은 버튼을 클릭했을 때 리소스를 추가하거나 변경하는 기능들을 제공하는 UI패턴**을 의미합니다.

&nbsp;

드롭다운 패턴에는 UX 관점에서 중요한 특징이 있는데 ① UI를 호출하기 위한 1개 이상의 **트리거** 요소가 필요하며, ② 트리거 요소를 클릭했을 때 표시되는 **드롭다운 UI 바깥 영역을 클릭 또는 포커스** 되었다면 UI는 사라져야 한다는 점입니다. 개발 관점에서는 ③ 동일한 드롭다운 컴포넌트가 여러 곳에서 반복적으로 사용될 수 있다는 특징이 있습니다.

> 참고 - Dropdown pattern : [링크](https://semantic-ui.com/modules/dropdown.html)

이러한 특징 때문에 드롭다운 컴포넌트를 구현한다면 아래 3가지 제약사항을 고려해야 합니다.

1. UI를 숨기는 이벤트 핸들링을 위해 드롭다운의 z-index는 기본 UI (트리거 요소 포함) 보다 상위에 위치해야 합니다.
2. 드롭다운 UI를 렌더링 할 때 불필요한 리렌더링이 일어나지 않도록 관련 없는 컴포넌트에 영향을 주지 않아야 합니다.
3. 렌더링된 드롭다운은 트리거 컴포넌트 근처에 위치해야 합니다.

React 16 버전에 추가된 `react portal` 을 사용하면 위 제약사항을 만족시키면서 드롭다운 UI를 구현할 수 있습니다. (3번은 별도 유틸 함수로 구현)

> 참고 - React Portals 가이드 : [링크](https://reactjs-kr.firebaseapp.com/docs/portals.html) / createPortal 사용 가이드 : [링크](https://react.dev/reference/react-dom/createPortal)

&nbsp;

## 2. Portal로 사용할 Dom Element 설정하기

가이드에서는 portal에 대한 설명을 다음과 같이 안내하고 있습니다. 

```
createPortal lets you render some children into a different part of the DOM.
```

React 프로젝트 생성 시 제공되는 `index.html`에 `root`라는 `id`를 가진 `div` 하위에 컴포넌트들을 그리게 되는데요. portal로 사용할 별도의 element를 만들고, `react-dom`에서 `createPoral`을 불러와서 사용하면, 해당 위치에 렌더링할 컴포넌트를 보낼 수 있습니다. 아래는 `createPortal`로 `modal`을 핸들링하는 예제 코드입니다.

```javascript
import { useState } from 'react';
import { createPortal } from 'react-dom';
import ModalContent from './ModalContent.js';

export default function PortalExample() {
  const [showModal, setShowModal] = useState(false);
  return (
    <>
      <button onClick={() => setShowModal(true)}>
        Show modal using a portal
      </button>
      {showModal && createPortal(
        <ModalContent onClose={() => setShowModal(false)} />,
        document.body
      )}
    </>
  );
}
```

위 예제에서는 `button element` 클릭 시 `showModal`이라는 상태를 `true`로 바꾸게 되고, 이 때 `showModal`이 `true`라면 `createPortal`로 `ModalContent`라는 컴포넌트를 `document.body` 하위에 렌더링합니다.

&nbsp;

기본적인 개념에 대해 알아보았으니 실제 앱에 적용해보도록 하겠습니다. 먼저 다른 portal들과 순서가 명확히 구분될 수 있도록 `index.html`에는 `dropdown`이라는 `id`를 가진 `div`를 `root` 다음 순서에 추가합니다.

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/Frame.png" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>가계부를 부탁해</title>
    <meta
      name="description"
      property="og:description"
      content="카드 지출내역, 영수증을 직접 등록하여 지출 내역을 관리해보세요."
    />
    <meta property="og:url" content="http://35.231.16.39" />
    <meta property="og:site_name" content="가계부를 부탁해" />
    <meta property="og:title" content="가계부를 부탁해" />
    <meta property="og:image" content="https://github.com/Ji-hoon/Home-accountant/raw/master/client/public/img-meta-image-1200.png" />
    <link rel="stylesheet" as="style" crossorigin href="https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/variable/pretendardvariable.min.css" />
  </head>
  <body>
    <div id="root"></div>
    <div id="dropdown"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

그리고 `createPortal` 메서드를 `Dropdown` 이라는 컴포넌트안에 정의하여 여러 컴포넌트에서 재사용 가능하도록 만들어줍니다. 이렇게 생성된 컴포넌트는 트리거 컴포넌트에서 import해서 사용할 수 있습니다.

```typescript
import styled from "styled-components";
import { useRecoilState } from "recoil";
import { dropdownOpenAtom } from "../../atoms/globalAtoms";
import { createPortal } from "react-dom";
import { PortalProps } from "../../global/customType";
import { COLORS, SIZES } from "../../global/constants";

export default function Dropdown({ children }: { children: React.ReactNode }) {
  const [showDropdown, setShowDropdown] = useRecoilState(dropdownOpenAtom);

  const DropdownPortal = ({ children }: PortalProps) => {
    return createPortal(
      children,
      document.getElementById("dropdown") as HTMLElement,
    );
  };

  return (
    <DropdownPortal>
      {showDropdown && (
        <DropdownContainer>
          <DropdownBackdrop
            className="dropdown-backdrop"
            onClick={() => setShowDropdown("")}
          />
          {children}
        </DropdownContainer>
      )}
    </DropdownPortal>
  );
}

const DropdownContainer = styled.div`
  position: absolute;

  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 111;

  & > *:not(.dropdown-backdrop) {
    z-index: 113;
  }
`;

const DropdownBackdrop = styled(DropdownContainer)`
  z-index: 112;
  position: fixed;
`;

export const DropdownUIContainerStyle = styled.div<{
  data: {
    x: number;
    y: number;
    width: number;
    height: number;
  };
}>`
  position: absolute;
  left: ${(props) => props.data.x + props.data.width - 200}px;
  top: ${(props) => props.data.y + props.data.height}px;
  height: auto;
  width: 200px;
  max-height: calc(100vh - 100px);

  overflow-x: hidden;
  overflow-y: auto;

  margin-top: 6px;
  background-color: #fff;
  border-radius: 5px;
  background-color: ${COLORS.BASIC_WHITE};
  box-shadow: 0 2px 7px 0 ${COLORS.GRAY_07_OVERAY};
  max-width: ${SIZES.MAX_WIDTH * 0.65}px;
`;
```

위 코드를 보면 `Dropdown` 컴포넌트에서 `showDropdown`이라는 recoil state가 빈 문자열이 아닌 경우 `DropdownContainer`를 렌더링 하게 되고, `DropdownContainer` 하위에는 `backdrop` 엘리먼트와 props로 전달된 `children`을 렌더링하게 됩니다. 이 때 화면 전체를 덮는 `backdrop` 요소를 클릭 하게되면 `showDropdown`상태를 `빈 문자열`로 할당하여 `DropdownContainer`를 숨김 처리할 수 있습니다.


&nbsp;

## 3. Portal 바깥에서 dropdown trigger 설정하기

이제 트리거 컴포넌트에서 `Dropdown` 컴포넌트를 불러와서 사용해보겠습니다. `Dropdown` 컴포넌트에서 사용하는 `showDropdown` 이라는 상태값을 이용 하고 리렌더링의 영향을 최소화 하기 위해 가장 작은 단위의 컴포넌트를 트리거로 사용합니다. 아래는 `Button_IconType`이라는 컴포넌트에 클릭 이벤트 핸들러를 사용해서 구현한 코드 입니다.

```typescript
import Dropdown from "../dropdown/Dropdown";
import Dropdown_Member from "../dropdown/Dropdown.Member";
import { useRecoilState } from "recoil";

export default function Button_Icontype({
  children,
}: {
  children: React.ReactElement | string;
}) {
  const [showDropdown, setShowDropdown] = useRecoilState(dropdownOpenAtom);

  return (
    <>
      <IcontypeButton
        onClick={setShowDropdown("MEMBER")}
      >
        {children}
      </IcontypeButton>
      {showDropdown && (
        <Dropdown>
          <Dropdown_Member />
        </Dropdown>
      )}
    </>
  );
}
```

위 코드에서는 버튼 클릭 시, `showDropdown` 상태 값을 업데이트 하여 `Dropdown_Member` 컴포넌트를 `dropdown portal`에서 렌더링하게 됩니다. 코드를 실행해보면 의도한 대로 동작하지만, 2가지 문제가 발생하는 것을 확인할 수 있습니다.

1. 드롭다운 컴포넌트의 위치가 **좌측 상단**으로 고정됨
2. 동일한 버튼 컴포넌트를 사용하고 있는 **모든 아이콘 버튼 클릭 시 드롭다운 컴포넌트가 표시**됨

문제를 해결하기 위해, 그리고 위에서 언급한 제약사항 3번(렌더링된 드롭다운은 트리거 컴포넌트 근처에 위치해야 합니다.)을 만족시킬 수 있도록 드롭다운의 컴포넌트의 위치를 계산하는 유틸 함수를 작성해 보도록 하겠습니다.

&nbsp;

## 4. 컴포넌트의 표시 위치 계산하기

이제 `showDropdown` 상태를 변경하는 트리거 컴포넌트의 위치를 계산하는 함수를 추가해보겠습니다.

```typescript
import Dropdown from "../dropdown/Dropdown";
import Dropdown_Member from "../dropdown/Dropdown.Member";
import { useRecoilState } from "recoil";
import { useRef } from "react";

export default function Button_Icontype({
  children,
}: {
  children: React.ReactElement | string;
}) {
  const [showDropdown, setShowDropdown] = useRecoilState(dropdownOpenAtom);

  const targetRef = useRef(null);

  function calculateElementPositionAndSize({
    target,
  }: {
    target: Element;
  }) {
    const targetRect = target?.getBoundingClientRect();
    const targetPosition = {
        x: targetRect.left,
        y: targetRect.top,
        width: targetRect.width,
        height: targetRect.height,
    };

    return targetPosition;
  }

  return (
    <>
      <IcontypeButton
        ref={targetRef}
        onClick={setShowDropdown("MEMBER")}
      >
        {children}
      </IcontypeButton>
      {showDropdown && (
        <Dropdown>
          <Dropdown_Member data={calculateElementPositionAndSize(targetRef)} />
        </Dropdown>
      )}
    </>
  );
}
```

추가된 함수는 `target`으로 받은 인자에 `getBoundingClientRect` 메소드를 실행하여 반환된 `left`, `top`, `width`, `height` 값을 각각 `x`, `y`, `width`, `height` 값으로 리턴합니다. 이 함수의 반환된 결과값을 `Dropdown_Member` 컴포넌트의 `data` props로 할당해주고 props로 내려받은 값을 아래 처럼 위치 정보로 처리할 수 있습니다.

```typescript
import { MenuGroup_ListType } from "../compound/MenuGroup.listType";
import Button_Boxtype from "../basic/Button.boxType";
import { DropdownProps } from "../../global/customType";
import { LABELS } from "../../global/constants";
import { DropdownUIContainerStyle } from "./Dropdown";

export default function Dropdown_Member({ data }: DropdownProps) {
  return (
    <DropdownUIContainerStyle data={data}>
      <MenuGroup_ListType>
        <li>
          <Button_Boxtype>{LABELS.LABEL_WITHDRAW_MEMBER}</Button_Boxtype>
        </li>
      </MenuGroup_ListType>
    </DropdownUIContainerStyle>
  );
}
```

&nbsp;

## 5. 드롭다운 고유 키를 사용한 분기 처리

이제 드롭다운의 위치 좌표를 트리거 엘리먼트의 기준 좌표에 따라 가변적으로 배치할 수 있습니다. 다만 아직도 해결해야 하는 문제는, `showDropdown` 이라는 1개의 상태값만 사용하고 있기 때문에 `IcontypeButton` 컴포넌트를 사용하는 상위 컴포넌트가 여러개 존재한다고 가정한다면, 1개의 클릭 이벤트만 발생하더라도 해당 컴포넌트를 사용하고 있는 모든 컴포넌트에서 `Dropdown_Member` 컴포넌트를 호출하게 됩니다.

&nbsp;

![duplicated dropdown](/blog/assets/posts/asset-dropdown-duplicated.png)

&nbsp;

이 문제를 해결하기 위해, `showDropdown` 상태가 클릭된 컴포넌트에 따라 고유한 값을 가지도록 개선하여 **하나의 액션 당 하나의 드롭다운 컴포넌트만 표시**될 수 있도록 개선해보겠습니다. 먼저 여러 컴포넌트에서 사용할 수 있도록 useDropdown 이라는 커스텀 훅으로 공통 로직을 분리합니다.

```typescript
import { throttle } from "lodash";
import { useState, useEffect, useRef } from "react";
import { useRecoilState } from "recoil";
import { dropdownOpenAtom } from "../../atoms/globalAtoms";
import { calculateElementPositionAndSize } from "../../util/handleElement";

export function useDropdown({
  dropdownType,
  dropdownId,
}: {
  dropdownType?: string | undefined;
  dropdownId?: string | undefined;
}) {
  const [showDropdown, setShowDropdown] = useRecoilState(dropdownOpenAtom);
  const [targetPosition, setTargetPosition] = useState({
    x: 0,
    y: 0,
    width: 0,
    height: 0,
  });

  const [windowWidth, setWindowWidth] = useState(window.innerWidth);

  const targetRef = useRef(null);

  // custom hook의 props로 전달된 2개의 값을 조합하여 드롭다운 unique key를 생성
  const showDropdownUniqueKey = `${dropdownType}_${dropdownId}`;

  const handleResize = throttle(() => {
    setWindowWidth(window.innerWidth);
  }, 500);

  useEffect(() => {
    window.addEventListener("resize", handleResize);
    return () => {
      // cleanup
      window.removeEventListener("resize", handleResize);
    };
  }, [handleResize]);

  /* resize 이벤트 발생 시 data props 갱신 */
  useEffect(() => {
    if (showDropdown && targetRef.current) {
      const targetPos = calculateElementPositionAndSize({
        target: targetRef.current as Element,
      });
      setTargetPosition(targetPos);
    }
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [windowWidth]);

  function handleDropdownTrigger() {
    const targetPos = calculateElementPositionAndSize({
      target: targetRef.current as unknown as Element,
    });
    setTargetPosition(targetPos);
    setShowDropdown(showDropdownUniqueKey);
  }

  return {
    targetRef,
    showDropdown,
    targetPosition,
    handleDropdownTrigger,
    showDropdownUniqueKey,
  };
}
```

위 커스텀 훅에서는 props로 전달받은 2개의 값을 조합하여 `showDropdownUniqueKey` 라는 값을 생성하고, `handleDropdownTrigger` 함수 호출 시 `showDropdown` 상태에 `showDropdownUniqueKey`을 할당하여 트리거한 컴포넌트에 따라 고유한 상태 값을 가질 수 있도록 합니다. 아래 코드처럼 커스텀 훅을 적용합니다.

```typescript
import { TYPES } from "../../global/constants";
import Dropdown from "../dropdown/Dropdown";
import Dropdown_Member from "../dropdown/Dropdown.Member";
import { useDropdown } from "../hooks/useDropdown";

export default function Button_Icontype({
  children,
  onClick,
  dropdownId,
  dropdownType,
}: {
  children: React.ReactElement | string;
  onClick?: (e: React.SyntheticEvent) => void;
  dropdownId?: string;
  dropdownType?: string;
}) {
  const {
    targetRef,
    showDropdown,
    targetPosition,
    handleDropdownTrigger,
    showDropdownUniqueKey,
  } = useDropdown({
    dropdownType,
    dropdownId,
  });

  return (
    <>
      <IcontypeButton
        ref={targetRef}
        className={showDropdown === showDropdownUniqueKey ? "active" : ""}
        onClick={dropdownType ? handleDropdownTrigger : onClick}
      >
        {children}
      </IcontypeButton>
      {showDropdown === showDropdownUniqueKey &&
        dropdownType === TYPES.DROPDOWN_KEY_MEMBER_MORE && (
          <Dropdown>
            <Dropdown_Member data={targetPosition} />
          </Dropdown>
        )}
    </>
  );
}
```

이제 `showDropdown` 상태값이 `showDropdownUniqueKey` 인 경우에만 `dropdown` 컴포넌트를 렌더링하게 됩니다.

&nbsp;

![dropdown components](/blog/assets/posts/asset-dropdown-demo.gif)

&nbsp;

> 태그 Tag : #react #typescript #react-portal