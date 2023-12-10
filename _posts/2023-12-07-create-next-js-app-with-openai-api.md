---
title: OpenAI API를 활용한 Next.js App 프로젝트를 Vercel에 배포하기
updated: 2023-12-07 12:00
---

> Elice SW Track 7기 - 13주차 진행 내용입니다.


![Elice Banner](/blog/assets/elice/SW7_top_banner.png)

&nbsp;

이전 글에서 GCP VM 환경에서 React 프로젝트를 배포하는 작업을 진행 했었습니다. 이번에는 [OpenAI API](https://platform.openai.com/){:target="_blank"} 를 활용한 [Next.js](https://nextjs.org/){:target="_blank"} 프로젝트를 만들어보고 [Vercel](https://vercel.com/){:target="_blank"} 에 배포하는 작업을 진행해보겠습니다. 

> 참고 - VM(가상머신) 환경에서 프로젝트 배포하기
 : [링크](https://ji-hoon.github.io/blog/project-release-on-vm){:target="_blank"}




&nbsp;

## 목차

[1. Next.js 프로젝트 생성하기](#1-nextjs-프로젝트-생성하기)

[2. OpenAI API 플랫폼 가입하고 키 발급받기](#2-openai-api-플랫폼-가입하고-키-발급받기)

[3. Chat API 작성하기](#3-chat-api-작성하기)

[4. UI 구성하기](#4-ui-구성하기)

[5. Vercel에 리포지토리 등록 후 배포하기](#5-vercel에-리포지토리-등록-후-배포하기)



&nbsp;

---

&nbsp;
## 1. Next.js 프로젝트 생성하기

**Next.js**는 CSR(Clent-side Rendering)의 단점을 극복하기 위해 개발된 React 기반의 오픈소스 렌더링 프레임워크입니다. React 문법을 그대로 사용하기 때문에, React에 익숙한 사람들이 쉽게 적응할 수 있다는 장점이 있습니다. 

**Next.js**는 SSR(Server-side Rendering, 서버 사이드 렌더링)과 SSG(Server-side Generation, 정적 사이트 생성)를 지원하여 기존 CSR의 단점이었던 느린 로딩속도, SEO 친화적인 기능을 제공합니다.

> 참고 - Next.js의 특장점과 빠르게 시작하는 법 : [링크](https://hanbit.co.kr/channel/category/category_view.html?cms_code=CMS7641364152){:target="_blank"}

아래 명령어를 통해 프로젝트를 생성합니다.

```shell
$ npm create-next-app@latest
```

명령어를 실행하면 프로젝트 이름, 디렉토리, 그리고 템플릿과 CSS 프레임워크를 선택하고, TypeScript와 ESLint, Prettier 사용 여부를 선택합니다. 이 프로젝트에서는 CSS 프레임워크로 **Tailwind**를, **TypeScript**를 사용해보겠습니다.


&nbsp;
## 2. OpenAI API 플랫폼 가입하고 키 발급받기

다음은 OpenAI의 API를 사용하기 위해 [홈페이지](https://platform.openai.com/){:target="_blank"}에서 가입을 한 뒤, API 키를 발급 받습니다. API key는 생성 시 제공된 팝업내에서 한 번만 키를 복사할 수 있기 때문에, 발급 받았다면 잘 저장해 둡니다.

&nbsp;

![api keys](/blog/assets/posts/asset-openai-api-key.png)


&nbsp;
## 3. Chat API 작성하기

발급 받은 API 키를 `환경 변수`로 만들어 등록한뒤, 질문 작성 후 요청할 API를 작성합니다. `/src/app/api/chat/` 경로에 `route.ts` 라는 파일을 생성합니다. 파일 안에는 아래와 같이 POST 메소드에 응답할 API를 작성해줍니다.

```javascript
import { OpenAIStream, StreamingTextResponse } from "ai";
import { NextRequest } from "next/server";
import OpenAI from "openai";

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function GET() {
  return new Response(`Hello OPEN AI! your key is: ${openai.apiKey}`);
}

export async function POST(req: NextRequest) {

  const { messages } = await req.json();
  const response = await openai.chat.completions.create({
    model: "gpt-3.5-turbo",
    stream: true, // true 설정 시, 응답을 나누어서 받습니다.
    temperature: 0.7,
    messages,
  });
  const stream = OpenAIStream(response);
  return new StreamingTextResponse(stream);
}
```

`Stream` 옵션을 `True`로 설정할 경우, Chat API 응답을 한 번에 모든 데이터를 받아서 넘겨 주는 것이 아니라, **Stream** 방식을 사용해서 데이터를 나누어서 전달 받게 됩니다.


&nbsp;
## 4. UI 구성하기

마지막으로 질문을 입력하고 응답받은 메시지를 표시하는 간단한 기능을 구현해 보겠습니다. `page.tsx` 파일에 `question`과 `messages` 상태를 `state hook`으로 정의하고, `input` 필드에 질문을 입력한 뒤 **질문하기 버튼** 클릭 시 사전에 정의한 API를 호출하는 로직입니다.

> page.tsx

```typescript
'use client'

import { useState } from "react";

type Chat = {
    role: "user" | "assistant";
    content: string;
}

export default function Home() {
    const [ question, setQuestion ] = useState("");
    const [ messages, setMessages ] = useState<Chat[]>([]);

    function handleQuestion(e:React.ChangeEvent<HTMLInputElement>) {
        setQuestion(e.target.value);
    }

    function postChatAPI() {

        (async () => {
            const response = await fetch("/api/chat", {
              method: "POST",
              body : JSON.stringify({
                // 보낼 때 message 히스토리를 포함하기 위해 기존 배열을 유지
                "messages" : [
                    ...messages,
                    {
                        role: 'user',
                        content: question,
                    }
                ],
              }),
            });
    
            const reader = response.body?.getReader();
            const decoder = new TextDecoder();
            let content ="";
            if(!reader) return;
    
            while (true) {
              
              const { done, value } = await reader.read();
    
              if(done) {
                console.log("done!"); 
                break;
              }
              const decodedValue = decoder.decode(value);
              content += decodedValue;

              setMessages([
                ...messages,
                {role : 'user', content : question},
                {role : 'assistant', content : content}
              ]);
            }
    
        })()
    }

    return (
        <div className="py-3 px-5">
            <h3 className="py-3 text-2xl" >GPT에게 질문해보세요.
            </h3>
            <input className="px-3 py-2 text-sm shadow-sm rounded-md w-1/2 ring-gray-300 dark:ring-gray-900 ring-1 ring-inset"
                    placeholder="질문을 입력하고 질문하기 버튼을 클릭하세요."
                    onChange={(e) => handleQuestion(e)} value={question} />
            <button className="mx-1 bg-indigo-600 px-3 py-2 text-sm font-semibold text-white shadow-sm rounded-md disabled:bg-gray-300 dark:disabled:bg-gray-700 disabled:cursor-not-allowed"
                    onClick={postChatAPI} 
                    disabled={!question}>질문하기</button>
            <p className="my-3">
            {!messages &&  "질문을 입력해주세요."}
            {messages.map( (message, index) => (
                <div key={index} className="flex border-t border-gray-200 dark:border-gray-700 py-3">
                    <p className="w-1/6 text-indigo-600 py-2 px-3">{message.role}</p>
                    <p className="w-5/6 px-3 py-2 bg-gray-100 dark:bg-gray-600 rounded-md">
                        <span key={index}>{message.content}</span>
                    </p>
                </div>)
            )}
            </p>
        </div>
    )
}
```
작성된 코드를 github 리포지토리를 연결한 후 push 합니다.

&nbsp;
## 5. Vercel에 리포지토리 등록 후 배포하기

이제 마지막으로 push한 리포지토리를 **Vercel**에 등록한 뒤 배포해보도록 하겠습니다. 먼저 https://vercel.com/signup 페이지에서 Signup(회원 가입)을 합니다. (타입은 hobby로 설정)
여기서는 Github 계정을 통해 가입합니다.

&nbsp;

![vercel signup](/blog/assets/posts/asset-vercel-signup-github.png)

&nbsp;

회원 가입이 완료되면, 아래 화면의 **install** 버튼을 클릭하여 불러올 Github 리포지토리를 선택합니다.

&nbsp;

![vercel install repo](/blog/assets/posts/asset-vercel-install-repository.png)

&nbsp;

install이 완료되면 선택한 리포지토리가 보여집니다. 아래 화면의 **import** 버튼을 클릭하여 프로젝트 배포 화면으로 넘어갑니다.

&nbsp;

![vercel import project](/blog/assets/posts/asset-vercel-import-project.png)

&nbsp;

마지막으로 API 키를 설정한 환경 변수 정보를 추가하고 **Deploy** (배포) 버튼을 클릭합니다.

&nbsp;

![vercel deploy](/blog/assets/posts/asset-vercel-deploy-project.png)

&nbsp;

배포가 완료되면 url을 할당받아 접속할 수 있게 됩니다.

&nbsp;

![vercel app preview](/blog/assets/posts/asset-vercel-app-preview.gif)

&nbsp;

---
&nbsp;

[![Elice UTM Banner](/blog/assets/elice/SW7_jihoonkim_bottom_banner.png)](https://elice.training/track/sw?utm_source=sw7&utm_medium=blog&utm_campaign=challenge&utm_content=m2gzitm8b)
&nbsp;
> 태그 Tag : #엘리스트랙 #엘리스트랙후기 #리액트네이티브강좌 #온라인코딩부트캠프 #온라인코딩학원 #프론트엔드학원 #개발자국비지원 #개발자부트캠프 #국비지원부트캠프 #프론트엔드국비지원 #React #Styledcomponent #React Router Dom #Redux #Typescript #Javascript