---
title: 함수형 컴포넌트에서 React Hook 활용하기
updated: 2023-11-19 12:00
---

> Elice SW Track 7기 - 10주차 진행 내용입니다.


![Elice Banner](/blog/assets/elice/SW7_top_banner.png)

&nbsp;

이전 글에서 `React Hook`을 사용하기 위해서는 `함수형 컴포넌트`에서만 가능하다고 소개 드렸습니다. 이번에는 가장 많이 사용되는 `Hook`에는 어떤 것들이 있는지 다뤄보도록 하겠습니다.

> 참고 - Hook 개요 : [링크](https://ko.legacy.reactjs.org/docs/hooks-intro.html){:target="_blank"}




&nbsp;

## 목차

[1. State hook](#1-state-hook)

[2. Effect hook](#2-effect-hook)

[3. Ref hook](#3-ref-hook)

[4. Custom hook](#4-custom-hook)



&nbsp;

---

&nbsp;
## 1. State hook

먼저 컴포넌트의 상태를 관리하는 `State hook` 입니다. `useState()` 메서드를 사용하며, 정의된 상태 값이 변경되면 컴포넌트가 리렌더링됩니다. 아래 예시 코드는 버튼을 클릭했을 때 카운트 상태값을 증가 시키는 동작을 수행합니다. 

```javascript
import React, { useState } from 'react';

function Button() {
    const [count, setCount] = useState(0);

    return( 
        <div>
            <button onClick={() => setCount(count+1)}>클릭</button>
            <div>클릭한 횟수 : {count}</div>
        </div>
    )
}
```

`State hook`을 사용하면 `count` 변수와 `setCount` 라는 메서드를 사용할 수 있게 됩니다. `State hook` 을 사용해서 정의한 변수들은 set- 메서드를 사용해서 변경해야만 React가 변화를 인지하고 컴포넌트를 리렌더링할 수 있습니다. (직접 변수를 수정할 경우 리렌더링이 일어나지 않습니다.)

&nbsp;
## 2. Effect hook

다음은 컴포넌트 생명주기에 따라 실행할 코드를 관리하는 `Effect hook` 입니다. `useEffect( () => { 실행할 코드 }, [의존성 배열])` 형태로 작성하며, 의존성 배열의 유무와 설정 값에 따라 코드가 실행되는 시점이 달라집니다.

```javascript
import React, { useState, useEffect } from 'react';

function Button() {
    const [count, setCount] = useState(0);
    
    // mount, unmount 시점에 1번 씩 실행되는 Effect hook
    useEffect( () => {
        console.log("didMount");

        return( () => {
            console.log("unMounted!");
        });
    }, []); 

    // count 상태 변경될 때 마다 실행되는 Effect hook
    useEffect( () => {
        console.log(count);
    }, [count,]);

    return( 
        <div>
            <button onClick={() => setCount(count+1)}>클릭</button>
            <div>클릭한 횟수 : {count}</div>
        </div>
    )
}
```
위 코드에서 첫 번째 `Effect hook` 처럼 의존성 배열에 빈 배열을 넣을 경우 `didMount` 시점에 1번,  `useEffect`의 콜백 함수에 정의된 코드가 `unMount` 시점에 **1번 씩**만 실행됩니다.

&nbsp;

두 번째 `Effect hook` 처럼 의존성 배열에 특정 상태 변수를 담을 경우, 해당 **상태 변수가 변경될 때마다** 정의된 코드가 실행됩니다. 만약 아래처럼 의존성 배열이 없는 `Effect hook` 에서 변경되는 상태를 의존성 배열에 담게 되면, **계속해서 화면이 새로고침**되는 상태에 빠지므로 주의해야 합니다. 

```javascript
import React, { useState, useEffect } from 'react';

function Button() {
    const [count, setCount] = useState(0);
    
    useEffect( () => {
        console.log("didMount");

        return( () => {
            console.log("unMounted!");
        });
    }, []); 

    useEffect( () => {
        // 의존성 배열에 담은 상태 변수를 변경하는 코드를 작성하면, 무한으로 코드가 반복됨
        setCount(count+1); 
        console.log(count);
    }, [count, ]);

    return( 
        <div>
            <button onClick={() => setCount(count+1)}>클릭</button>
            <div>클릭한 횟수 : {count}</div>)
        </div>
    )
}
```

&nbsp;
## 3. Ref hook

`State hook` 을 통해 선언된 상태 값이 변경되면 화면을 리렌더링하는 반면, `Ref hook`으로 관리하는 값은 값이 변해도 화면이 리렌더링되지 않습니다. 아래 코드를 실행해보면 `countRef++` 버튼을 클릭해도 변경된 값이 표시되지 않지만, 
`count++` 버튼을 클릭해야 변경된 값이 표시되는 것을 확인할 수 있습니다.

```javascript
import React, { useState, useEffect, useRef } from 'react';

function Button() {
    const [count, setCount] = useState(0);
    const countRef = useRef(count);

    useEffect( () => {
        console.log("didMount");

        return( () => {
            console.log("unMounted!");
        });
    }, []); 

    useEffect( () => {
        console.log(count);
    }, [count, ]);

    return( 
        <div>
            <button onClick={() => {
                setCount(count+1);
                }}>count++</button>
            <em>&nbsp;</em>
            <button onClick={() => {
                countRef.current = countRef.current+1;
                }}>countRef++</button>
            <div>클릭한 횟수 : {count} | countRef : {countRef.current}</div>
        </div>
    )
}
```
추가로 `Ref hook`의 경우 리렌더링이 일어나도 참조하는 값을 항상 유지되는 특성을 활용하여 `DOM 엘리먼트를` 참조할 수도 있습니다. 아래 코드를 실행하면, `count` 상태가 변경되어 리렌더링이 일어나더라도 `inputRef`에 담긴 인풋에 **focus()** 가 계속해서 유지되는 것을 확인할 수 있습니다. (input 엘리먼트의 ref 어트리뷰트로 inputRef를 할당)

```javascript
import React, { useState, useEffect, useRef } from 'react';

function Button() {
    const [count, setCount] = useState(0);
    const countRef = useRef(count);
    const inputRef = useRef(null);

    useEffect( () => {
        console.log("didMount");

        return( () => {
            console.log("unMounted!");
        });
    }, []); 

    useEffect( () => {
        console.log(count);
        inputRef.current.focus(); // count 값이 변경될 때 마다 focus
    }, [count, ]);

    return( 
        <div>
            <button onClick={() => {
                setCount(count+1);
                }}>count++</button>
            <em>&nbsp;</em>
            <button onClick={() => {
                countRef.current = countRef.current+1;
                inputRef.current.focus(); // 버튼 이벤트 발생할 때 마다 focus
                }}>countRef++</button>
            <div>클릭한 횟수 : {count} | countRef : <input ref={inputRef} value={countRef.current} /></div>
        </div>
    )
}
```

&nbsp;
## 4. Custom hook

`Custom hook`은 React에서 지원하는 hook은 아니지만 특정 로직들을 추상화시키고 이를 캡슐화 하여 재사용이 가능하도록 만든 함수를 의미합니다. 아래 코드는 버튼 클릭 시 각 버튼에서 관리하는 변수를 업데이트하는 코드를 `Custom hook` 으로 생성한 예시입니다.

```javascript
import React, { useState, useEffect, useRef } from 'react';

const useUpdateCount = () => {

    const [value, setValue] = useState(0);
    const onClick = (e) => {
        setValue(parseInt(e.target.textContent)+1);
    }

    return {value, onClick};
}

function Button() {
    const countA = useUpdateCount();
    const countB = useUpdateCount();

    return( 
        <div>
            A를 클릭한 횟수 : <em>&nbsp;</em>
            <button {...countA}>{countA.value}</button>
            <br />
            B를 클릭한 횟수 : <em>&nbsp;</em>
            <button {...countB}>{countB.value}</button>
        </div>
    )
}
```

위 코드를 실행하면, `countA`와 `countB` 변수에 `useUpdateCount` 이라는 `Custom hook` 을 연결하고, 리턴되는 값을 `button` 엘리먼트에 연결하였기 때문에 클릭 이벤트 발생 시 각 버튼의 레이블이 업데이트되는 것을 확인할 수 있습니다.

&nbsp;

![custom hook result](/blog/assets/posts/asset-react-custom-hook.gif)



&nbsp;

---
&nbsp;

[![Elice UTM Banner](/blog/assets/elice/SW7_jihoonkim_bottom_banner.png)](https://elice.training/track/sw?utm_source=sw7&utm_medium=blog&utm_campaign=challenge&utm_content=m2gzitm8b)
&nbsp;
> 태그 Tag : #엘리스트랙 #엘리스트랙후기 #리액트네이티브강좌 #온라인코딩부트캠프 #온라인코딩학원 #프론트엔드학원 #개발자국비지원 #개발자부트캠프 #국비지원부트캠프 #프론트엔드국비지원 #React #Styledcomponent #React Router Dom #Redux #Typescript #Javascript