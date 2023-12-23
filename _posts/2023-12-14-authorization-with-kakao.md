---
title: 카카오 소셜 로그인을 활용한 인증 서비스 구현하기 (1)
updated: 2023-12-14 12:00
---

> Elice SW Track 7기 - 14주차 진행 내용입니다.


![Elice Banner](/blog/assets/elice/SW7_top_banner.png)

&nbsp;

이전 글에서 **JWT를 이용한 토큰 발급 및 검증 로직을 구현**하는 작업을 진행 했었습니다. 이번에는 [카카오 소셜 로그인 API](https://developers.kakao.com/docs/latest/ko/kakaologin/common){:target="_blank"}를 활용한 인증 서비스 로직을 구현해보도록 하겠습니다.

> 참고 - Node.js에서 JWT를 이용한 토큰 발급 및 검증 로직 구현하기 : [링크](https://ji-hoon.github.io/blog/JWT-and-authorization){:target="_blank"}




&nbsp;

## 목차

[1. 카카오 소셜 로그인](#1-카카오-소셜-로그인)

[2. 카카오 로그인 구현 방식](#2-카카오-로그인-구현-방식)

[3. 카카오 앱 설정하기](#3-카카오-앱-설정하기)

[4. API 통신 구조 설계하기](#4-api-통신-구조-설계하기)

[5. Oauth API 구현하기](#5-oauth-api-구현하기)

[6. 로그아웃과 회원탈퇴 API 구현하기](#6-로그아웃과-회원탈퇴-api-구현하기)



&nbsp;

---

&nbsp;
## 1. 카카오 소셜 로그인

**소셜 로그인**은 이용하고자 하는 서비스마다 이름, 이메일, 비밀번호 등 여러 정보들을 입력 받아 계정을 생성하는게 아닌 특정 SNS 서비스 계정에 로그인 한 뒤, 이용하고자 하는 서비스에 필요한 정보 제공에 동의하고 해당 서비스를 손쉽게 이용할 수 있도록 합니다.

주로 카카오나 네이버, 구글, 페이스북 등 여러 SNS 계정을  통해서 이용하는데, 이번에는 그 중에서 일반적으로 사용되는 카카오 계정을 활용한 소셜 로그인 기능을 구현해보겠습니다.

> 참고 - 소셜 로그인 사용 비율 : [링크](http://www.mknews.kr/view?no=30407){:target="_blank"}

&nbsp;

## 2. 카카오 로그인 구현 방식

**카카오 로그인**을 구현하기 위해서는 **SDK 설치 방식**과 **REST API** 2가지 방식을 사용할 수 있습니다. SDK (Software Development Kit) 방식의 경우 SDK를 활용해서 인증 로직을 별도로 구현할 필요없이, 제공되는 메소드들로 손쉽게 구현할 수 있다는 장점이 있습니다. 

> 참고 - SDK를 활용한 카카오 로그인 데모 : [링크](https://developers.kakao.com/tool/demo/login/login){:target="_blank"}

반면 REST API 방식의 경우에는 직접 HTTP 요청을 보내서 카카오 로그인 API와 통신하는 형태로 소셜 로그인을 구현합니다. SDK 방식과 달리 카카오에서 제공하는 API 들에 요청을 보내고 응답을 처리하는 로직을 직접 구현해야 합니다. 이 글에서는 **REST API** 방식을 통해 구현해보겠습니다.

> 참고 - REST API를 활용한 카카오 로그인 개발 문서 : [링크](https://developers.kakao.com/docs/latest/ko/kakaologin/rest-api){:target="_blank"}


&nbsp;

## 3. 카카오 앱 설정하기

카카오 로그인을 구현하려면 가장 먼저, [Kakao Developers](https://developers.kakao.com/){:target="_blank"}에 가입한 후 앱을 생성해야 합니다. (카카오 계정만 있으면 가입할 수 있습니다.)

가입을 완료했다면, `내 애플리케이션 > 애플리케이션 추가하기` 메뉴로 진입하여 필요한 정보들을 입력하고 생성을 완료합니다.

&nbsp;

![kakao create app](/blog/assets/posts/asset-kakao-dev-new-app.png)*애플리케이션 생성하기 창*{: .caption}

&nbsp;

앱 생성이 완료되었다면, `앱 설정 > 요약 정보`에서 `REST API` 키 를 복사해서  환경 변수에 설정 후 사용합니다.

그 다음은 `앱 설정 > 플랫폼` 화면 하단의 Web 영역에서 `사이트 도메인` 경로를 설정합니다. 여기에는 **클라이언트 프로젝트의 URL**을 입력합니다. (카카오 로그인 인가 코드를 요청할 URL)

마지막으로 사이트 도메인 경로 하단의 **텍스트 링크**(`https://developers.kakao.com/console/app/{app-id}/product/login`)를 클릭해서 **카카오 로그인**을 활성화 시키고, **동의 항목**을 설정해줍니다. 마지막으로 **Redirect URI**를 설정해줍니다. 이 글에서는 `http://{serverurl}/api/auth/oauth` 로 API를 개발할 것이기 때문에 동일한 URL을 입력해줍니다.


&nbsp;

## 4. API 통신 구조 설계하기

클라이언트 프로젝트에서는 `Vite`와 `React`를 사용하고, 서버 프로젝트에서는 `Node.js`와 `Express`를 사용했습니다. 또한 각 프로젝트 마다 사용되는 **URL**이 다르기 때문에 서버에서 처리할 부분과 클라이언트에서 처리할 부분을 잘 구분해서 설계를 해야 합니다. 아래 이미지는 카카오 개발문서에서 제공하는 로그인 과정입니다.

&nbsp;

![kakao login process](https://developers.kakao.com/docs/latest/ko/assets/style/images/kakaologin/kakaologin_sequence.png)*카카오 로그인 과정 ([출처](https://developers.kakao.com/docs/latest/ko/kakaologin/rest-api){:target="_blank"})*{: .caption}

&nbsp;

복잡해 보이지만 하나 하나 순서대로 살펴보면 :

1. 클라이언트 : 로그인 페이지와 인가 코드를 받기 위한 카카오 서버측 URL로 진입할 수 있는 경로를 제공합니다.
2. 카카오 : 카카오로 `/oauth/authorize` API 요청을 보내면, 카카오 로그인 여부에 따라 로그인 화면 또는 동의 화면을 제공해줍니다. 이때 동의하고 계속하기를 하면 카카오 서버 측에서는 사전에 설정한 `Redirect URI`에 `search query`의 `code` 값에 **인가 코드**를 담아서 전달해줍니다. 
3. 서버 & 카카오 : Redirect URI 라우터를 구현합니다. 전달된 인가 코드로 카카오 측 `/oauth/token` API로 요청을 보내서 액세스 토큰과 리프레시 토큰을 응답 받습니다. 이 토큰으로 다시 `https://kapi.kakao.com/v2/user/me` API로 요청을 보내서 카카오 계정 정보와 정보 제공에 동의한 정보를 받아올 수 있습니다.
4. 서버 : 이 정보를 바탕으로 서비스 내 DB에서 가입 여부를 판단하고 가입되어 있다면 로그인을, 가입되어 있지 않다면 회원가입을 시키고 토큰이 담긴 쿠키를 클라이언트로 응답을 반환합니다.
   
&nbsp;

위 통신 구조를 잘 이해하기 위해 플로우차트로 시각화해보았습니다. (좌측이 클라이언트, 중간부분이 카카오, 우측이 서버 영역입니다.) 이제 아래 이미지를 바탕으로 기능을 구현해보겠습니다.

&nbsp;

![kakao oauth process](/blog/assets/posts/Elice_2ndProject_UI_Specification_SocialLogin_Flowchart_compressed.png)*카카오 로그인 플로우 차트 ([크게 보기](https://ji-hoon.github.io/blog/assets/posts/Elice_2ndProject_UI_Specification_SocialLogin_Flowchart_compressed.png){:target="_blank"})*{: .caption}

&nbsp;

## 5. Oauth API 구현하기

서버 사이드에서 카카오 로그인에 필요한 **Oauth API**는 **1개**로 작성하고, API 내부에서 카카오 측으로 API 요청을 보내고 정보를 받아 활용하는 형태로 구현합니다. 아래 코드는 `/api/oauth`의 `router`에 연결된 `handleKakaoOAuthProcess` 함수 코드 입니다. (서비스 로직은 별도 파일에 작성해서 사용했습니다.)

```typescript
async function handleKakaoOAuthProcess (req: express.Request, res: Response) {
  // URL로 넘어온 인가 코드가 존재하는지 검증하고 code에 할당합니다.
  const code = await authService.validateKakaoOAuthCode(
    req.query.code as string,
  );
  // 인가 코드로 카카오 API를 호출하고 액세스 토큰을 받아와서 data에 할당합니다.
  const data = await authService.getKakaoAccessToken(code);

  // 액세스 토큰으로 카카오 API를 호출하고, 카카오 계정 정보를 받아와서 userInfo에 할당합니다.
  const userInfo = await authService.getUserInfo(data.accessToken);

  // 반환된 userInfo 객체의 id(카카오 id)로 서비스에 가입된 계정을 찾아 isNewUser에 할당합니다.
  const isNewUser = await userService.searchUsers(userInfo.id);

  // mongoose에서 find 메소드로 찾았기 때문에 가입 이력이 없다면 "join"을, 있다면 "login"을 action에 할당합니다.
  const action = isNewUser.length === 0 ? "join" : "login";

  // 카카오 계정 정보와 action을 인자로 전달하여 회원가입 또는 로그인 완료된 서비스 계정 정보를 result에 할당합니다.
  const result = await authService.handleAuthUser(userInfo, action);
  
  // 서비스 계정의 id와, 랜덤하게 생성된 nickname 그리고 exp(만료일 정보)를 payload에 담아 JWT를 생성하고 token에 할당합니다.
  const token = authService.generateJWT(
    result.user._id,
    result.user.nickname,
  );
  
  // 응답 값에 쿠키를 생성하고, "service_token" 값으로 생성한 token을 할당합니다. 이 때 httpOnly옵션을 true로 설정하여 브라우저에서 쿠키에 접근하지 못하도록 제한합니다.
  res.cookie("service_token", token, { path: "/", httpOnly: true });

  // 마지막으로 설정된 프론트엔드 URL의 /login 페이지로 리다이렉트 시키고, 생성된 계정의 id와 nickname을 search query로 넘겨주어서 클라이언트에서 활용할 수 있도록 제공합니다.
  res.redirect(
    `${process.env.FRONTEND_URL}/login?id=${result.user._id}&nickname=${result.user.nickname}`,
  );
}
```

&nbsp;

## 6. 로그아웃과 회원탈퇴 api 구현하기

인증 성공 시 로그인과 토큰 발급 기능을 구현했으니, 이제 로그아웃과 탈퇴 시 처리를 구현해보겠습니다. 각각 `/api/auth/logout`과 `/api/auth/withdraw`로 API를 구현합니다. 

> 로그아웃

```javascript
function handleLogout (req: express.Request, res: Response) {
  res.clearCookie("service_token", { path: "/", maxAge: 0 });
  res.status(204).send();
}
```
&nbsp;

먼저 로그아웃의 경우에는 http요청에 담긴 쿠키를 초기화하여 204 상태코드와 함께 반환합니다. 이렇게 하면 클라이언트 측 쿠키를 삭제할 수 있습니다.

> 회원 탈퇴

```javascript
function handleLogout (req: express.Request, res: Response) {
  const userToken = req.cookies.service_token;
  const userId = authService.extractDataFromToken(userToken, "user_id");
  const result = await userService.withdrawUser(userId);
  if (!result) {
    throw new CustomError({
      status: 400,
      message: "회원 탈퇴에 실패했습니다.",
    });
  }
  res.clearCookie("service_token", { path: "/", maxAge: 0 });
  res.status(200).json({
    message: "회원 탈퇴에 성공했습니다.",
    user: result,
  });
}
```
&nbsp;

회원 탈퇴의 경우에는 먼저 `request cookie`에 담긴 토큰에서 `user_id`를 가져와서, 사용자 모델의 `deletedAt` 필드에 `date` 값을 할당한 결과를 `result`에 할당합니다. 이 때 `findAndUpdate` 메소드를 사용하는데, `{ new : true }` 옵션을 사용할 경우 업데이트 결과를 반환합니다.

```javascript
async withdrawUser(userId: string) {
  return await UserModel.findOneAndUpdate(
    { _id: userId },
    { deletedAt: new Date() }, // 2023-12-17T06:06:10.987+00:00
    { new: true },
  );
},
```
&nbsp;

위 서비스 로직을 통해 필드가 업데이트된 사용자가 반환된 경우, 로그아웃과 마찬가지로 쿠키의 토큰을 삭제 후 200 상태코드와 함께 응답을 반환하면 회원 탈퇴 로직 구현이 완료되었습니다.

&nbsp;

이어지는 글에서는 클라이언트 사이드에서 어떻게 구현해야 하는지 다뤄보도록 하겠습니다. 🥰


&nbsp;

---
&nbsp;

[![Elice UTM Banner](/blog/assets/elice/SW7_jihoonkim_bottom_banner.png)](https://elice.training/track/sw?utm_source=sw7&utm_medium=blog&utm_campaign=challenge&utm_content=m2gzitm8b)
&nbsp;
> 태그 Tag : #엘리스트랙 #엘리스트랙후기 #리액트네이티브강좌 #온라인코딩부트캠프 #온라인코딩학원 #프론트엔드학원 #개발자국비지원 #개발자부트캠프 #국비지원부트캠프 #프론트엔드국비지원 #React #Styledcomponent #React Router Dom #Redux #Typescript #Javascript