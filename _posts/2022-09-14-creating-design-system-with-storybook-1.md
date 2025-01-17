---
title: (작성중) 스토리북(Storybook)으로 디자인 시스템 구축하기
updated: 2022-09-15 12:00
---

> ERP 제품 '로지메이트'의 프론트엔드 개발을 진행하며 작성한 내용입니다.

![redesign preview](/blog/assets/logimate/HFS_LOGO.png)

&nbsp;

저는 24년 하반기부터 로지메이트라는 ERP(Enterprise Resource Planning, 전사적 자원 관리) 제품의 프론트엔드 개발을 담당하고 있습니다. 입사 후 첫 담당 업무인 리디자인을 담당하고 개발을 진행하면서 자체 디자인 시스템 (LDS : Logimate Design System)을 구축하고 적용해본 경험을 공유해보고자 합니다.

&nbsp;

## 목차

[1. 해결하고자 했던 문제 정의하기](#1-해결하고자-했던-문제-정의하기)

[2. 스토리북을 선택한 이유](#2-스토리북을-선택한-이유)

[3. 아토믹 디자인을 활용한 컴포넌트 구조 설계하기](#3-아토믹-디자인을-활용한-컴포넌트-구조-설계하기)

[4. Atom 컴포넌트 작성하기](#4-atom-컴포넌트-작성하기)

[5. Module 컴포넌트 작성하기](#5-module-컴포넌트-작성하기)

[6. Layout 컴포넌트 작성하기](#6-layout-컴포넌트-작성하기)

[7. 스토리북에 컴포넌트 등록하기](#7-스토리북에-컴포넌트-등록하기)

[8. 실제 제품에 적용하기](#8-실제-제품에-적용하기)

&nbsp;

---

&nbsp;

## 1. 해결하고자 했던 문제 정의하기

기존 제품에서는 공통화된 컴포넌트들로 view를 구성하기 보다는 `className`을 사용한 `css` 스타일링과 `inline` 스타일링, `css module`이 혼합된 방식으로 스타일을 적용하고 있었습니다. 때문에 중복되는 스타일 및 html 코드가 존재하고 있었기에, 이를 효율적으로 관리하고자 여러 곳에서 **반복적으로 사용되는 UI 요소나 패턴을 정의**하고 해당 패턴을 컴포넌트화 하여 재사용하기 용이한 디자인 시스템을 구축하여 운영 및 신규 기능 개발 시 생산성을 높일 수 있는 환경을 구축하고자 했습니다.

&nbsp;

이러한 패턴을 파악하기위해 기존 제품에서 사용중인 UI 패턴과 페이지를 수집하고 모듈화 할 수 있는 기능들을 리스트업하고, 피그마로 대표적인 키스크린에 대한 디자인 프로토타입을 작성한 뒤 세부적인 UI들을 컴포넌트화 하는 작업을 선행적으로 수행했습니다.

&nbsp;

## 2. 스토리북을 선택한 이유

![storybook](/blog/assets/posts/asset-storybook-opengraph-image.jpg)*Storybook ([링크](https://storybook.js.org/){:target="_blank"})*{: .caption}

&nbsp;

디자이너로 이전 조직들에서 근무할 때도 스토리북을 사용한 프론트엔드 디자인 시스템을 구축하는 일을 직간접적으로 경험했었습니다. 스토리북으로 디자인 시스템을 구축했을 때 대표적인 장점은 **여러 직군간 협업과 커뮤니케이션을 용이하게 해준다** 정도로 도출할 수 있지만, 현재 조직에서는 디자이너 포지션이 없기 때문에 스토리북으로 디자인 시스템을 혼자 구축해보면서 개발자로서 느꼈던 장점은 아래처럼 꼽아볼 수 있습니다.

> 실제 개발환경과 분리된 환경에서도 컴포넌트 단위의 개발을 수행할 수 있다.

보통 로컬 환경에서 개발 시 `http://localhost:3000` 처럼 `3000` 포트를 디폴트로 사용하는 로컬 서버에서 개발을 진행하고 테스트 합니다. 스토리북은 이러한 로컬 서버와 분리된 별도 서버 (`http://localhost:6006`)를 사용하며, 실제 개발과 분리된 환경에서 개발하고자 하는 컴포넌트에만 집중할 수 있기 때문에 다른 컴포넌트나 코드에 의존적이지 않은 컴포넌트 개발을 수행할 수 있다는 장점이 있습니다.

> 빠른 개발주기를 가진 작은 조직에서도 각각의 컴포넌트에 대한 문서화를 수행할 수 있다.

보통 작은 조직일 수록 구성원 각자가 맡은 책임의 범위와 업무 스코프가 넓습니다. 결국 중요한 업무를 우선적으로 수행하다보니 기록이나 체계를 잡아나가는 일을 병행해서 진행하기 쉽지 않기에 제대로된 문서화도 남기기 어렵게 됩니다. 스토리북에서는 컴포넌트 단위의 상세 스펙을 기록하고 props가 어떤 역할을 하는지, 합성 컴포넌트의 경우 어떠한 컴포넌트가 합성되어 있는지 등을 구체적으로 기록해둘 수 있습니다. 또한 필요하다면 이렇게 작성된 스토리북 문서를 정적 페이지로 배포하여 다른 직군, 개발자들이 항상 접근할 수 있도록 협업 환경을 구축할 수 있다는 장점이 있습니다.

> 컴포넌트의 내부 구현을 모르더라도, 스토리북의 props 선택 기능을 통해 사용하고자 하는 컴포넌트의 코드를 빠르게 얻어낼 수 있다.

디자인 시스템이 없는 조직에서 공통 컴포넌트를 사용하지 않고 개발을 할 경우 중복되는 로직과 코드가 기하급수적으로 늘어날 수 있고, 이는 곧 유지 보수 시 생산성이 급격히 감소함을 의미합니다. 하지만 스토리북을 활용하면 개발자 본인이 작성하지 않은 컴포넌트라도 어떠한 형태로 확장되는 지, 또 어떤 props를 변경했을 때 output을 즉각적으로 확인해볼 수 있고, 코드도 빠르게 얻어낼 수 있다는 장점이 있습니다.

&nbsp;

## 3. 아토믹 디자인을 활용한 컴포넌트 구조 설계하기

[아토믹 디자인](https://bradfrost.com/blog/post/atomic-web-design/){:target="_blank"}이란, UI를 구성하는 요소들을 가장 작은 단위인 **atom**에서부터, **molcule**, **organism**, **templates**, **pages** 까지 점차 확장된 구조로 분리해서 관리하는 방법론입니다. 실제 UI 디자인 작업 시 많이 참고했던 개념이지만 프론트엔드에서 디자인 시스템을 만들때도 유용하게 사용된 개념입니다.

&nbsp;

![atomic design](/blog/assets/posts/asset-atomic-design-process.png)*Atomic Design Concepts (출처 - bradfrost blog : [링크](https://atomicdesign.bradfrost.com/chapter-2){:target="_blank"})*{: .caption}

&nbsp;

다만 프론트엔드 개발에서는 폴더 구조나 컴포넌트 설계 상 depth가 많아지면 많아질수록 코드를 이해하기 어렵고, 최악의 경우에는 과도한 props 드릴링이 불가피해질 수 있는 결과가 발생할 수 있기에 총 6가지 depth 대신 **3가지로 depth로 간소화**해서 도입하고자 했습니다. 또한 조직 구조상 인하우스 디자이너 역할이 없는 환경에서 직접 디자인 작업을 병행해서 작업해야 했었기에, 피그마로 대표적인 키스크린과 작은 **Atom** 단위의 버튼, 타이포그래피, 인풋필드, 컬러칩부터 Atom 단위의 요소가 결합된 **Module (=molecules)** 단위의 요소 - 필드 그룹, 버튼 그룹, 토스트, 리스트 아이템 등을 디자인하고 마지막 **Layout (=organisms)** 단위의 요소 - 드롭다운, 모달, 사이드바 등 으로 확장될 수 있는 컴포넌트 구조로 설계했습니다.

&nbsp;

최종적으로 **Page** 단계에서는 이러한 **Atom**, **Module**, **Layout** 단위의 컴포넌트를 자유롭게 활용함으로써 재사용성이 높고 예측 가능한 UI를 구성할 수 있게 됩니다. 이 때 page는 커스텀훅 또는 자체 로직을 작성하고, 디자인 시스템을 활용한 view를 사용하는 형태의 컴포넌트 단위로 구성했습니다.

&nbsp;

## 4. Atom 컴포넌트 작성하기

Atom 컴포넌트 단위에는 **가장 기본이 되는 UI 요소**를 정의합니다. Atom 컴포넌트 그 자체로는 기능에 대한 어떠한 로직도 포함하고 있지 않으며, 특정 액션이 필요한 경우 (e.g. `onClick`)는 상위 컴포넌트에서 props로 주입받거나 특정 상태를 전달받았을 때 처리할 로직이나 형태를 선언적으로 정의합니다. 다만 전달받는 props에 따른 시각적 형태나 액션을 반영하기위해 **styled component**를 활용해서 다양한 케이스의 아웃풋에 대응할 수 있도록 스타일을 구성합니다.

&nbsp;

#### 4-1. Typography

![Typography](/blog/assets/posts/asset-lds-typography.png)

&nbsp;

제품에서 사용되는 모든 텍스트들을 styled 컴포넌트 객체들로 정의한 **Typography** 컴포넌트 입니다. label, subTitle, title, heading, Tagline 등으로 사이즈 단위로 정의하며, 각 컴포넌트들은 props로 `$color (string)`와 `$weight (string)` props를 전달받아 스타일을 적용합니다.

&nbsp;

#### 4-2. Spacing

![Spacing](/blog/assets/posts/asset-lds-spacing.png)

&nbsp;

레이아웃을 구성할 때 단순히 여백을 구현하고 싶을 때 사용되는 **Spacing** 컴포넌트 입니다. props로 `$direction`과 `$value`를 전달받습니다.

&nbsp;

#### 4-3. Profile

![Profile](/blog/assets/posts/asset-lds-profile.png)

&nbsp;

특정 사용자 또는 계정의 프로필을 표시할 때 사용되는 **Profile** 컴포넌트 입니다. props로 `$type`과 `$size`, `$imageUrl` 등을 전달받습니다. props로 전달받은 `$type`이 어떤 값이냐에 따라 리턴하는 요소를 다르게 렌러딩 합니다. 

&nbsp;

#### 4-4. Badge

![Badge](/blog/assets/posts/asset-lds-badge.png)

&nbsp;

숫자, 레이블 등 뱃지 UI를 표시할 때 사용되는 **Badge** 컴포넌트 입니다. props로 `$type`과 `$size`, `$value` 등을 전달받습니다.

&nbsp;

#### 4-5. Checkbox

![Checkbox](/blog/assets/posts/asset-lds-checkbox.png)

&nbsp;

Form 에서 사용되는 **Checkbox** 컴포넌트 입니다. props로 `$isSelected`와 `$label` 등을 전달받고 input element는 숨기처리 후 커스텀하게 구현된 체크박스 UI를 표시합니다. `$label` 존재 한다면 체크박스와 함께 텍스트 값도 렌더링합니다.

&nbsp;

#### 4-6. Button

![Button](/blog/assets/posts/asset-lds-button.png)

&nbsp;

여러 UI에서 가장 빈번하게 사용되는 **Button** 컴포넌트 입니다. props로 `$type`와 `$label`, `$linkTo` 등을 전달받습니다. 여기서는 `$type`이 **'LINK'** 타입이라면 react-router-dom에서 지원하는 `NavLink` 컴포넌트를 styled 컴포넌트에서 확장한 스타일을 사용해서 적용하고, 아닌 경우에는 `as` 옵션을 사용해서 `button` 엘리먼트로 렌더링할 수 있습니다.

> 참고 - Extending Styles : [링크](https://styled-components.com/docs/basics#extending-styles){:target="_blank"}

&nbsp;

## 5. Module 컴포넌트 작성하기

Module 컴포넌트로는 기본 단위의 **Atom 컴포넌트가 조합된 형태의 UI 요소**를 정의합니다.

&nbsp;

#### 5-1. Inputfield

![Inputfield](/blog/assets/posts/asset-lds-typography.png)

&nbsp;

Form에서 가장 기본적으로 사용되는 **Inputfield** 컴포넌트입니다. 필드의 입력 값 유무에 따라 값을 삭제하는 **Button** 컴포넌트가 선택적으로 렌더링되는 합성 컴포넌트로 구현했습니다. props로는 형태를 결정하는 `$type`과 `$status`, `$fieldName`, `$placeholder` 와  `$register`, `$watch`, `$resetField` 등 reack hook form에서 사용되는 메소드들을 전달 받아 상위 Form 엘리먼트에서 인풋필드의 값을 사용할 수 있도록 구성 합니다.

&nbsp;

#### 5-2. Select

![Select](/blog/assets/posts/asset-lds-typography.png)

&nbsp;

역시 Form에서 기본적으로 사용되는 **Select** 컴포넌트입니다. multi props의 유무에 따라 여러 옵션을 선택할 수 있도록 구현했습니다.

&nbsp;

#### 5-3. List item

![List item](/blog/assets/posts/asset-lds-typography.png)

&nbsp;

다양한 배리에이션이 제공되는 **List Item** 컴포넌트입니다. 드롭다운 리스트에 포함되는 `LIST` 타입, 체크박스 옵션이 포함되는 `TOGGLE` 타입, 클릭 시 메뉴가 전환될 수 있는 `MENU` 타입, 알림 드롭다운에 포함되는 `NOTIFICATION` 타입 등 props로 전달된 타입 정보에 따라 각각 다른 스타일의 output이 도출될 수 있도록 구현했습니다.

&nbsp;

#### 5-4. List Header

![List Header](/blog/assets/posts/asset-lds-typography.png)

&nbsp;

앞서 설명한 List Item과 같이 표시하는 **List Header** 컴포넌트입니다. 메뉴 리스트에 표시되는 클릭하여 리스트를 펼치고 접을 수 있는 `EXPANDALBE` 타입, 정적인 리스트에 표시되는 `STATIC` 타입, 옵션 리스트와 같이 표시되는 `OPTION` 타입 등 props로 전달된 타입 정보에 따라 다른 결과물을 얻을 수 있도록 구현했습니다.

&nbsp;

##### 5-5. List Group

![List Group](/blog/assets/posts/asset-lds-typography.png)

&nbsp;

List Item과 List Header를 조합해서 사용하는 **List Group** 컴포넌트입니다. 드롭다운 리스트 또는 셀렉트 옵션 리스트 등에서 사용되는 리스트 그룹을 정의합니다.

&nbsp;

#### 5-7. Profile Group

![Profile Group](/blog/assets/posts/asset-lds-typography.png)

&nbsp;

Profile 컴포넌트와 text 정보, Button 컴포넌트가 조합된 **Profile Group** 컴포넌트입니다.

&nbsp;

#### 5-8. Tab Navigation

![Tab Navigation](/blog/assets/posts/asset-lds-typography.png)

&nbsp;

여러개의 Tab 컴포넌트가 결합된 **Tab Navigation** 컴포넌트입니다. props로는 `$activeId`와 `$type`을 전달받아 활성탭과 비활성탭, 확장 or 비확장 타입의 다른 output이 도출될 수 있도록 구현했습니다.

&nbsp;

#### 5-9. Tooltip

![Tooltip](/blog/assets/posts/asset-lds-typography.png)

&nbsp;

특정 버튼 또는 마우스 액션이 제공되는 UI의 액션에 대해 부가적인 설명을 제공해주는 **Tooltip** 컴포넌트입니다. props로는 `$position`, `$direction`, `$description` 등을 전달받습니다.

&nbsp;

## 6. Layout 컴포넌트 작성하기

Layout 컴포넌트로는 **Atom과 Module을 결합하여 독립적으로 사용될 수 있는 UI 요소**를 정의합니다. Layout 컴포넌트 단위부터는 동작에 대한 자체 로직을 포함하게 됩니다.

&nbsp;

#### 6-1. Dropdown

(image)

&nbsp;

특정 액션 버튼을 클릭 시, 또는 셀렉트 옵션을 확장 했을 때 표시되는 **Dropdown** 컴포넌트입니다. 이 때 컴포넌트의 children을 외부에서 합성 컴포넌트 형태로 전달받게되며, 컴포넌트 레벨에서는 위치를 계산하고 백드롭 영역을 클릭 시 드롭다운 컴포넌트를 닫는 등 컴포넌트를 조작하는 로직을 같이 작성하여 구현합니다.

&nbsp;

#### 6-2. Modal

(image)

&nbsp;

특정 정보를 전달하거나 액션을 수행하기 위해 필요한 정보를 전달할 때 표시되는 **Modal** 컴포넌트입니다. Dropdown과 마찬가지로 React Portal을 사용해서 구현하였으며, children 요소를 외부에서 합성 컴포넌트 형태로 전달받도록 구현했습니다.

&nbsp;

#### 6-3. Sidebar

(image)

&nbsp;

여러 페이지들에서 공통적으로 사용되는 **Sidebar** 컴포넌트입니다.

&nbsp;

## 7. 스토리북에 컴포넌트 등록하기

컴포넌트에 다양한 배리에이션이 존재하는 경우 스토리북에서는 아래처럼 정해진 props 속성을 리스트화하여 표시할 수 있는 기능이 있습니다. 따라서 스토리북 문서로만 봐도 해당 컴포넌트가 어떤 형태로 사용될 수 있는지 코드를 작성하지 않아도 예측할 수 있다는 장점이 있습니다.

> Provider 설정

(참고 코드 추가)

작성한 컴포넌트를 스토리북에서 확인하려면 `컴포넌트.stories.tsx` 파일을 생성해야 합니다. 이 때 Redux나 Recoil 등 상태관리를 위한 Provider를 사용하는 컴포넌트가 있다면 해당 Provider를 스토리 파일에 정의합니다. 또한 react-hook-form 같은 라이브러리에서 지원하는 메소드를 사용하는 컴포넌트가 있다면 이 또한 Provider에 설정하는 작업이 필요합니다.

> 디폴트 html 설정

(참고 코드 및 이미지 추가)

root 외에 div 엘리먼트를 사용하는 React Portal이 정의된 컴포넌트라면 스토리북 디폴트 html에 해당 div 엘리먼트를 정의해야 합니다.

> 컴포넌트 폴더 구조와 설명 및 서브 컴포넌트 설정

(참고 이미지 추가)

컴포넌트 폴더 구조는 Atom, Module, Layout 3가지 분류로 나누고, 하위에 해당 분류의 컴포넌트가 속할 수 있도록 이미지처럼 경로를 설정합니다. 컴포넌트 내부에 다른 컴포넌트가 포함된 합성 컴포넌트의 경우 서브 컴포넌트로 표시될 수 있게 설정할 수 있습니다.

> 컴포넌트의 props 설명 추가

(참고 코드 or 이미지 추가)

컴포넌트에서 사용되는 모든 props에 대해서 어떤 선택지가 있고, 어떤 상태를 변형시키게되는 지 설명을 정의할 수 있습니다.

&nbsp;

## 8. 실제 제품에 적용하기

리디자인이 적용된 UI를 제품에 반영하기 위해 사이드바 UI를 스토리북으로 만든 컴포넌트들로 구성해 보도록 하겠습니다. 대표적으로 사이드바 컴포넌트에는 아래의 컴포넌트들이 사용됩니다.

```
1. Profile Group
   1. Profile
   2. Button
2. List Group
3. List Header
4. List Item
5. Dropdown
6. Tooltip
```

이 때 버튼과 특정 리스트 아이템의 onClick 이벤트나 dropdown, tooltip 등을 띄우기 위해 각 컴포넌트에 props로 사용할 컴포넌트와 UI를 넘겨 활용할 수 있도록 구현합니다.

(props로 넘기고 실제 구현되는 코드는 Atom 컴포넌트에서 작성한 코드를 추가)

TBD...

&nbsp;

---

&nbsp;

디자이너가 있는 조직에서 여러 개발자들과 함께 디자인 시스템을 구축해본 경험은 아니었지만 디자인 시스템을 구축할 때 스토리북은 굉장히 유용한 도구임을 경험해 볼 수 있었습니다.

&nbsp;

또 이번에 구축한 디자인 시스템의 컴포넌트들은 실제 제품에 점진적으로 반영하고 있기에 처음부터 모든 컴포넌트를 준비하고 제품에 적용하고 교체하는 방식 보다는 기존 디자인과 신규 디자인을 선택 가능하도록 사용자에게 제공하고, 적응기간을 제공하는 동안 그 다음 단계의 디자인을 적용하고 교체하는 방식을 채택했습니다. 다음 주제로는 '서비스중인 제품에 디자인 리뉴얼을 적용하기'에 대해서 공유해보도록 하겠습니다.

&nbsp;

---

&nbsp;

#### 참고하면 좋은 링크

- TBD : [링크](){:target="_blank"}

&nbsp;

> 태그 Tag : #react #storybook #design-system #atomic-design


