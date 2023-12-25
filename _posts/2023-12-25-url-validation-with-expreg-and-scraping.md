---
title: 정규 표현식과 web scraping을 활용한 URL 검증 기능 구현하기
updated: 2023-12-25 12:00
---

> Elice SW Track 7기 - 16주차 진행 내용입니다.

![Elice Banner](/blog/assets/elice/SW7_top_banner.png)

&nbsp;

이전 포스팅에서 이미지 태깅 컴포넌트를 작성해보았습니다. 이번 글에서는 정보 태깅 시 입력한 URL의 유효성 검증과 URL의 실제 메타 정보를 보여주는 기능을 구현하여 휴먼 에러를 방지해보겠습니다.

> 이전 글 - 이미지 정보 태깅을 위한 TagButtonHandler 훅 구현하기 : [링크](https://ji-hoon.github.io/blog/implement-tag-button-handler)

&nbsp;

## 목차

[1. 유효성 검증 기능 구현하기](#1-유효성-검증-기능-구현하기)

[2. URL 미리보기 기능 구현하기 (서버)](#2-url-미리보기-기능-구현하기-서버)

[3. URL 미리보기 기능 구현하기 (클라이언트)](#2-url-미리보기-기능-구현하기-클라이언트)

&nbsp;

---

&nbsp;

## 1. 유효성 검증 기능 구현하기

입력한 URL이 유효한지 검증하는 로직을 작성하기 위해서 어떤 검증 조건들이 있을 지 정리해보도록 하겠습니다. 

1. http:// 또는 https:// 가 포함되어 있는지?
2. .{domain} 형태의 도메인 주소가 포함되어 있는지?

위 2개의 조건들을 검증하는 코드를 작성해보면 아래와 같이 정규 표현식으로 정의가 가능합니다.

&nbsp;

```javascript
async function fetchUrlInfo(url: string) {
    try {
      // 입력된 URL의 유효성 검사
      const regex = /^(http:\/\/|https:\/\/).*\.[a-zA-Z]{2,}/;

      if (!regex.test(url)) {
        toast.error("유효하지 않은 URL입니다. 다시 한 번 확인해주세요.");
        return;
      }

      ...
    } catch (error) {
      toast.error(error as string);
    }
  }
```

> 참고 - 정규 표현식 : [링크](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Regular_expressions){:target="_blank"}

&nbsp;

## 2. URL 미리보기 기능 구현하기 (서버)

사용자가 입력한 URL이 정말 의도했던 URL이 맞는 지 확인할 수 있는 기능을 제공하려면, 해당 URL의 `og` 정보를 가져와서 보여주는 기능을 구현해야 합니다. **og**란 **open graph**의 약자로, URL을 방문하기 전에 해당 URL이 어떤 정보를 담고 있는 지 미리 확인할 수 있는 정보를 제공하는 프로토콜을 의미합니다. 보통 `og:title`, `og:image`, `og:description`, `og:url` 같은 정보들을 `html`의 `head tag`안에 `meta tag`로 정의합니다.

> 참고 - Open Graph Protocol : [링크](https://ogp.me/){:target="_blank"}

클라이언트에서 URL 입력 시, 미리보기 버튼을 제공하여 입력한 URL이 의도한 것이 맞는지 확인할 수 있는 정보를 제공하기 위해 서버 사이드 코드를 작성해보겠습니다.

> parseRouter.ts

```javascript
import { Router } from "express";
import parseController from "./parse.controller.js";

const parseRouter = Router();

parseRouter.get("/", parseController.getOGdata);

export default parseRouter;
```

> parseController.ts

```javascript
import "dotenv/config";
import asyncHandler from "../middleware/asyncHandler.js";
import express, { Response } from "express";
import { load } from "cheerio";

const parseController = {
  getOGdata: asyncHandler(async (req: express.Request, res: Response) => {
    const url = req.query.url; // 요청 파라미터에서 크롤링할 URL을 가져옴

    const response = await fetch(url as string);
    const html = await response.text();
    const $ = load(html);

    const ogTitle = $('meta[property="og:title"]').attr("content");
    const ogURL = $('meta[property="og:url"]').attr("content");
    const ogImage = $('meta[property="og:image"]').attr("content");
    const ogDescription = $('meta[property="og:description"]').attr("content");

    res.json({ ogTitle, ogURL, ogImage, ogDescription });
  }),
};

export default parseController;
```

&nbsp;

`/api/parse?url={url}` 요청으로 들어온 url의 정보를 파싱하기 위해 `cheerio` 라이브러리를 설치하고, 파싱한 텍스트에서 사용할 정보들을 각각 변수로 가져와 응답으로 반환합니다.

&nbsp;

## 3. URL 미리보기 기능 구현하기 (클라이언트)

이제 클라이언트 단에서 응답을 받아 처리하는 부분을 작성해보겠습니다. 이전 글에서 사용했던 팝업 내 일부를 변경해서, 미리보기 버튼 엘리먼트를 추가합니다. 해당 버튼은 입력한 URL이 있을 경우 해당 URL을 /api/parse?url={입력한 url} 로 get 요청을 보내서 필요한 og 데이터들을 응답받는 역할을 수행합니다.

> ImageAnchorButton.tsx

```javascript
import UrlPreview from "../UrlPreview/UrlPreview";

export default function ImageAnchorButton({ ... }) {
  ...
  async function fetchUrlInfo(url: string) {
    try {
      // 입력된 URL의 유효성 검사
      const regex = /^(http:\/\/|https:\/\/).*\.[a-zA-Z]{2,}/;

      if (!regex.test(url)) {
        toast.error("유효하지 않은 URL입니다. 다시 한 번 확인해주세요.");
        return;
      }

      // URL에 접근하여 og 이미지와 제목 정보 가져오기
      const response = await parseAPI.getURLMetaData(url);

      setMetaData(response.data);
    } catch (error) {
      toast.error(error as string);
    }
  }
  ...
  return (
    ...
      <section>
        <label>링크</label>
        <div className="input-w-button">
          <input
            ref={tagUrlRef}
            name="taguRL"
            type="text"
            placeholder="URL을 입력해주세요"
            onChange={() => {
              if (tagUrlRef.current) {
                setMetaData(undefined);
                setTagUrl(tagUrlRef.current.value);
              }
            }}
            value={tagUrl}
          />
          <button
            disabled={tagUrl.length == 0 ? true : false}
            onClick={() => {
              if (tagUrl.length > 0) {
                fetchUrlInfo(tagUrl);
              }
            }}
          >미리보기
          </button>
        </div>
        <div className="meta-data">
          {metaData && <UrlPreview url={metaData} />}
        </div>
      </section>
    ...
  )
}
```

> UrlPreview.tsx

```javascript
import * as S from "./UrlPreview.styles";

export default function UrlPreview({
  url,
}: {
  url: {
    ogTitle: string;
    ogDescription: string;
    ogImage: string;
    ogURL: string;
  };
}) {
  return (
    <S.PreviewContainer href={url.ogURL} target="_blank">
      <S.MetaImageContainer>
        <img src={url.ogImage} />
      </S.MetaImageContainer>
      <S.MetaDataContainer>
        <h3>{url.ogTitle}</h3>
        <p>{url.ogDescription}</p>
        <span>{url.ogURL}</span>
      </S.MetaDataContainer>
    </S.PreviewContainer>
  );
}
```

&nbsp;

![og 미리보기](/blog/assets/posts/asset-og-preview.gif)

&nbsp;

---
&nbsp;

[![Elice UTM Banner](/blog/assets/elice/SW7_jihoonkim_bottom_banner.png)](https://elice.training/track/sw?utm_source=sw7&utm_medium=blog&utm_campaign=challenge&utm_content=m2gzitm8b)
&nbsp;
> 태그 Tag : #엘리스트랙 #엘리스트랙후기 #리액트네이티브강좌 #온라인코딩부트캠프 #온라인코딩학원 #프론트엔드학원 #개발자국비지원 #개발자부트캠프 #국비지원부트캠프 #프론트엔드국비지원 #React #Styledcomponent #React Router Dom #Redux #Typescript #Javascript