---
title: 카카오 소셜 로그인을 활용한 인증 서비스 구현하기 (2)
updated: 2023-12-17 12:00
---

> Elice SW Track 7기 - 14주차 진행 내용입니다.


![Elice Banner](/blog/assets/elice/SW7_top_banner.png)

&nbsp;

이전 글에서 **카카오 소셜 로그인을 활용한 인증 서비스 구현**하는 작업 중 서버 사이드 부분을 진행 했었습니다. 이번에는 클라이언트 사이드에서 필요한 로직을 구현해보도록 하겠습니다.

> 참고 - 카카오 소셜 로그인을 활용한 인증 서비스 구현하기 (1) : [링크](https://ji-hoon.github.io/blog/authorization-with-kakao){:target="_blank"}



&nbsp;

## 목차

[1. 토큰과 로컬 스토리지, 쿠키 이해하기](#1-토큰과-로컬-스토리지-쿠키-이해하기)

[2. 인증 정보 관리하기](#2-인증-정보-관리하기)

[3. 클라이언트 사이드에서 카카오 로그인 구현하기](#3-클라이언트-사이드에서-카카오-로그인-구현하기)

[4. 로그아웃, 회원탈퇴 구현하기](#4-로그아웃-회원탈퇴-구현하기)



&nbsp;

---

&nbsp;

## 1. 토큰과 로컬 스토리지, 쿠키 이해하기

실제 구현에 앞서, **토큰**과 **로컬 스토리지** 그리고 **쿠키**의 개념에 대해서 짚어 보고 넘어가보겠습니다.

&nbsp;

**토큰**은 인증 관리 방식 중 하나이며, **세션**과 다르게 웹 환경이 아닌 환경에서도 범용적으로 사용될 수 있는 방식입니다. 하지만 HTTP 통신은 stateless 하기에 서버가 발급한 **토큰**을 클라이언트에서 저장하고 관리할 공간이 필요한데, 이 때 사용될 수 있는 방식이 **로컬 스토리지**와 **쿠키** 입니다.

&nbsp;

로컬 스토리지, 쿠키 모두 접속 주소마다 별도의 독립적인 스토리지(or 쿠키)을 가질 수 있지만 로컬 스토리지는 보안 측면에서 injected된 코드에서 접근이 가능하다는 측면에서 토큰 같은 인증 정보를 담기에는 위험합니다. 

&nbsp;

반면 **쿠키**의 경우에는 서버에서 토큰 발급 후 응답을 보낼 때 아래 코드처럼 `httpOnly` 옵션을 설정한 경우 클라이언트에서는 접근할 수 없기에 보안 측면에서 적절한 방식이라고 할 수 있습니다. 

&nbsp;

```javascript
res.cookie("service_token", token, { path: "/", httpOnly: true });
```

&nbsp;

## 2. 인증 정보 관리하기

앞서 설명한 대로, 쿠키를 통해 인증 토큰을 생성하고 저장한 경우 클라이언트에서는 응답 토큰에 담긴 사용자의 정보를 알 수 없습니다. 그렇기 때문에 서버에서 로그인 성공 응답 시 사용자의 userId와 nickname을 search qurey로 포함시켜 반환하면, 클라이언트에서는 해당 정보를 parsing해서 로컬 스토리지에 저장하고 활용하는 방식으로 인증 정보를 관리하는 코드를 작성하였습니다.

> auth.controller.ts

```javascript
// /api/auth/kakao 라우트 핸들러
...
const result = await authService.handleAuthUser(userInfo, action);
const token = authService.generateJWT(
    result.user._id,
    result.user.nickname,
);
res.cookie("service_token", token, { path: "/", httpOnly: true });
res.redirect(
    `${process.env.FRONTEND_URL}/login?id=${result.user._id}&nickname=${result.user.nickname}`,
);
```

> /login.tsx

```javascript
export default function Login() {
  const [searchParams] = useSearchParams();
  const location = useLocation();
  const id = searchParams.get("id");
  const nickname = searchParams.get("nickname");

  if (id && nickname) {
    storage.set(storageKeys.currentUser, {
      userId: id,
      nickname: nickname,
    });
    return <Navigate to={location?.state?.from ?? PATH.root} />;
  }
  ...
}
```
&nbsp;

클라이언트에서는 `redirect url`에 `id`와 `nickname`이 담겨있다면, 로컬 스토리지에 `currentUser`라는 객체에 `usreId`와 `nickname` 정보를 저장하고, root 또는 직전 path로 리다이렉트 시키고 이후 해당 정보를 활용할 수 있게됩니다.


&nbsp;

## 3. 클라이언트 사이드에서 카카오 로그인 구현하기

이제 클라이언트 사이드에서의 로직을 작성해보겠습니다. `/login` 경로에 접근 시 **카카오 계정으로 로그인 버튼**을 제공합니다. REST API 방식을 사용할 때는 **axios**나 **fetch** 같은 형태로 API를 호출하여 응답 받는 방식이 아니라 URL로 이동시키는 처리를 해야 하기 때문에 `Anchor` 태그를 버튼 형태로 작성했습니다.

> 참고 - 카카오 Dev talk : [링크](https://devtalk.kakao.com/t/cors-error/121958/3){:target="_blank"}

아래 코드에서는 `AnchorButton`이라는 `a` 태그를 활용하여 `styled 컴포넌트`를 만들고 활용하고 있습니다.

&nbsp;

```javascript
// 로그인 페이지에서 버튼 제공
export default function Login() {
  ...
  return (
    <S.Container>
      <AnchorButton
        bgcolor="#fde433"
        textcolor="#333"
        href={kakaoAuthorizeURL} // http://{HOST}/api/auth/kakao
        onClick={() => {}}
        children="카카오 계정으로 로그인"
      />
    </S.Container>
  );
}
```
&nbsp;

```javascript
// AnchorButton 컴포넌트
import * as S from "./AnchorButton.styles.ts";
import { AnchorHTMLAttributes } from "react";

type ButtonProps = AnchorHTMLAttributes<HTMLButtonElement> & {
  children?: string;
  href?: string | undefined;
  bgcolor: string;
  textcolor: string;
  onClick: () => void;
};
export default function AnchorButton({
  children,
  href,
  bgcolor,
  textcolor,
  onClick,
}: ButtonProps) {
  return (
    <>
      <S.AnchorButton
        href={href}
        onClick={onClick}
        bgcolor={bgcolor}
        textcolor={textcolor}
      >
        {children}
      </S.AnchorButton>
    </>
  );
}

// Anchor Button Style
export const AnchorButton = styled.a<ButtonProps>`
  background-color: ${(props) => props.bgcolor};
  color: ${(props) => props.textcolor};
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
  text-align: center;
  width: 70%;
  max-width: ${({ theme }) => theme.size.maxWidth}px;
  font-size: ${({ theme }) => theme.size.lg}px;
  padding: ${({ theme }) => theme.size.md}px ${({ theme }) => theme.size.lg}px;
`;
```

&nbsp;

![kakao login preview](/blog/assets/posts/asset-kakao-login-client.gif)

&nbsp;

## 4. 로그아웃, 회원탈퇴 구현하기

마지막으로 로그인 이후 로그아웃과 탈퇴에 대한 클라이언트 사이드에서 구현을 해보겠습니다.

&nbsp;

서버 사이드에서는 로그아웃과 탈퇴 요청 시 계정 정보를 처리한 뒤, 쿠키를 삭제 후 응답을 반환하게 되는데 이 때 클라이언트에서는 http요청에 쿠키를 담아서 요청을 보내야 합니다. 요청에 쿠키가 담겨있지 않다면, 서버 사이드 로직에서 쿠키를 삭제한 뒤 반환하더라도 클라이언트의 쿠키는 업데이트되지 않습니다. 

&nbsp;

또한 프로젝트 구성이 프론트엔드와 서버의 URL이 다르게 설정되었다면, **CORS(Cross-origin Resource Sharing)** 요청이 되기 때문에 `withCredentials` 옵션을 `true`로 설정해주어야만 CORS 요청에 인증 정보(쿠키 포함)가 담겨서 서버로 보낼 수 있습니다.

> Profile.Setting.hooks.ts

```javascript
export const useProfileSetting = () => {
  const navigate = useNavigate();

  const logout = useMutation({
    mutationFn: profileSettingAPI.logout,
    onSuccess: (response) => {
      const message =
        response.status === 204
          ? "로그아웃 되었습니다."
          : response.data.message;

      toast.success(message);
      storage.remove(storageKeys.currentUser);
      navigate(PATH.root);
    },
    onError: (error) => {
      toast.error(error.message);
    },
  }).mutateAsync;

  const withdraw = useMutation({
    mutationFn: profileSettingAPI.withdraw,
    onSuccess: (response) => {
      const message = !response.data.message
        ? "회원 탈퇴가 처리되었습니다."
        : response.data.message;

      toast.success(message);
      storage.remove(storageKeys.currentUser);
      navigate(PATH.login);
    },
    onError: (error) => {
      toast.error(error.message);
    },
  }).mutateAsync;

  return { logout, withdraw };
};
```
> Profile.Setting.api.ts

```javascript
import { axiosInstance } from "../../global/axiosInstance";

const profileSettingAPI = {
  logout: async () => {
    const data = {};
    const response = await axiosInstance.post("/auth/logout", data, {
      withCredentials: true,
    });
    return response;
  },
  withdraw: async () => {
    const data = {};
    const response = await axiosInstance.post("/auth/withdraw", data, {
      withCredentials: true,
    });
    return response;
  },
};
export default profileSettingAPI;
```

&nbsp;

![logout and withdraw preview](/blog/assets/posts/asset-kakao-logout-client.gif)

&nbsp;

---
&nbsp;

[![Elice UTM Banner](/blog/assets/elice/SW7_jihoonkim_bottom_banner.png)](https://elice.training/track/sw?utm_source=sw7&utm_medium=blog&utm_campaign=challenge&utm_content=m2gzitm8b)
&nbsp;
> 태그 Tag : #엘리스트랙 #엘리스트랙후기 #리액트네이티브강좌 #온라인코딩부트캠프 #온라인코딩학원 #프론트엔드학원 #개발자국비지원 #개발자부트캠프 #국비지원부트캠프 #프론트엔드국비지원 #React #Styledcomponent #React Router Dom #Redux #Typescript #Javascript