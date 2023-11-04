---
title: Node.js에서 JWT를 이용한 토큰 발급 및 검증 로직 구현하기
updated: 2023-11-04 10:00
---

# Node.js에서 JWT를 이용한 토큰 발급 및 검증 로직 구현하기

&nbsp;

![Elice Banner](/blog/assets/elice/SW7_top_banner.png)


인증에는 다양한 방법이 있지만, 이번에는 다양한 클라이언트에서 범용적으로 사용될 수 있는 JWT를 이용해서 로그인 및 토큰 인증 기능을 구현해보았습니다.

&nbsp;

## 목차
1. Web Token을 활용한 인증 관리 방식
2. JWT의 구조
3. JWT를 만들어보자
4. 토큰이 올바른지 확인 하는 방법
5. 로그인 응답에 JWT 담아서 반환하기
6. 인증이 필요한 요청 시 JWT 검증하여 결과 반환하기

&nbsp;
## 1. Web Token을 활용한 인증 관리 방식
로그인 발생 시, 서버(Server)는 사용자를 식별할 수 있는 토큰(Web Token)을 생성하여 클라이언트(Client)에게 발급을 하게 됩니다. (토큰 생성시 토큰에 서명 = **sign**).
&nbsp;

클라이언트는 발급 받은 토큰을 로컬 저장소나 쿠키 (local storage, Cookie) 등 에 저장하여 이후 서버를 향한 HTTP 요청(HTTP request) 시 해당 토큰을 HTTP header등에 담아 보내고,서버는 전달받은 토큰에 대해 검증을 진행하고 유효한 토큰일 경우에만 요청을 처리하게 됩니다.
&nbsp;

 이 때, JSON 포멧을 이용하여 사용자에 대한 속성을 저장하는 Claim 기반의 Web Token(= **JWT**)이 주로 사용됩니다. 

&nbsp;
## 2. JWT의 구조
JWT는 헤더 Header, 페이로드 Payload, 서명 Signature 3가지 형태로 구성되어 있습니다.
&nbsp;

**헤더 Header** 에는 토큰 타입(**typ**)과 알고리즘(**alg**) 정보를 담고 있습니다. 아래의 경우는 토큰의 타입은 JWT이고, HS256 알고리즘을 사용하여 서명을 만들었다는 것을 의미합니다.
```
{
  "alg": "HS256",
  "typ": "JWT"
}
```
**페이로드 Payload** 는 토큰을 누구에게 발급했는지, 유효 기간이 언제 까지인지 등의 정보를 담고 있습니다. 유효기간(**exp**)과 발급 시간(**iat**)를 제외하고는 키 값을 자유롭게 설정해서 담을 수 있습니다.
&nbsp;

아래 예시에서는 user_id와 role, exp를 사용하여 작성된 형태를 보여주고 있습니다.

```
{
  "user_id": "sample_id",
  "role": "USER",
  "exp": 1698991726,
  "iat": 1698991696
}
```
**서명 Signature** 에는 **Secret Key**를 사용한 토큰을 인코딩하거나 유효성 검증을 할 때 사용하는 고유한 암호화 코드가 담겨 있습니다.
```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  {Secret Key}
)
```

&nbsp;
## 3. JWT를 만들어보자

그럼 JWT는 어떻게 만들까요? 먼저 jsonwebtoken을 설치 해야 합니다.
```shell
$ npm install jsonwebtoken
```
여기서는 Node.js에서 토큰을 생성하는 코드 예시입니다. (ESM 방식)
```javascript
import jwt from "jsonwebtoken"; /* jwt를 사용하기 위해 jsonwebtoken import */

const expirationTime = Math.floor(Date.now() / 1000) + 60*60; /* 현재시간 + @, e.g. 60*60 = 1시간 후 만료되도록 설정 */
const payload = { 
    user_id : "sample_id",
    role : "USER",
    exp : expirationTime,
}; /* payload에 담을 객체를 선언합니다. */

const secretKey = "jwt-secret-key"; /* 사용할 secret 키를 선언합니다. */
const token = jwt.sign(payload, secretKey); /* jwt의 sign 메소드를 사용해서 token을 생성합니다. */

```
&nbsp;

위 코드에서 payload, secretKey 값으로 생성된 token의 값은 다음과 같습니다.
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoic2FtcGxlX2lkIiwicm9sZSI6IlVTRVIiLCJleHAiOjE2OTg5OTE2OTYsImlhdCI6MTY5ODk5MTY5Nn0.9JiUW66ABzPvf374S4QtFW05-hhmPyCwL8skx-nzU7k
```
&nbsp;
## 4. 토큰이 올바른지 확인 하는 방법

위에서 생성한 토큰은 [https://jwt.io](https://jwt.io/) 에서 확인해볼 수 있습니다. 이 때 서명 영역의 secret key를 바꿔 보면, Invalid Signature 라고 표시되는 걸 확인할 수 있습니다. (payload에 있는 data를 변경하려면, secret key를 알고 있어야 한다는 의미 이기도 합니다.)
&nbsp;

![jwt example](/blog/assets/posts/asset-jwt-example.png)

&nbsp;
그럼 이제 코드 상에서는 토큰의 유효성 여부를 어떻게 검증할 수 있는지 코드로 살펴 보겠습니다.

&nbsp;
## 5. 로그인 응답에 JWT 담아서 반환하기
앞서 클라이언트의 로그인 요청을 받고, 성공 응답에 JWT를 담아서 반환한다고 했었습니다. 여기서는 유저와 비밀번호에 대한 검증이 완료되었다고 가정하고, API 응답에 생성된 JWT를 담아서 반환하는 예시 코드를 작성해보겠습니다.
```javascript
import jwt from "jsonwebtoken"; 
import { Router } from 'express';
const router = Router();

// 서버를 구동하기 위한 코드는 생략되었습니다.

router.post("/signin", async (req, res) => {
    // ... 유저와 비밀번호를 검증
    const expirationTime = Math.floor(Date.now() / 1000) + 60*60;
    const payload = { 
        user_id : "sample_id",
        role : "USER",
        exp : expirationTime,
    };

    const secretKey = "jwt-secret-key";
    const token = jwt.sign(payload, secretKey);

    res.setHeader('Authorization', `Bearer ${token}`);
    res.status(200)
        .json({message: "로그인에 성공했습니다."});
});
```
&nbsp;

위 라우터 코드가 정상적으로 실행되면 아래 이미지처럼 Response > Header > Authorization 필드 에 토큰이 담겨 반환되는 것을 확인할 수 있습니다.
![로그인 응답](/blog/assets/posts/asset-jwt-response.png)


&nbsp;
## 6. 인증이 필요한 요청 시 JWT 검증하여 결과 반환하기
이제 마지막으로 Node.js에서 JWT를 검증하는 코드를 작성해 보겠습니다.
&nbsp;

응답받은 JWT는 클라이언트가 사용자 인증이 필요한 HTTP 요청을 서버로 보낼 때, HTTP Header의 Authorization 필드에 아래 형태로 담아 전송합니다.

| Key          | Value           |
| -------------|-----------------|
| Authorization| Bearer {jwt}    |

서버에서 토큰을 검증하는 로그인 미들웨어에 대한 예시 코드 입니다.
```javascript
import { Router } from 'express';
const router = Router();
import login-middleware from "../middlewares/login-middleware.js";

/* 라우터에서 실행 코드가 실행되기 전, 아래 처럼 미들웨어를 선언하여 줍니다. 이렇게 미들웨어로 작성하게 되면 다양한 라우터에서 동일하게 사용할 수 있다는 장점이 있습니다. */
router.get("/users/:email",
    login-middleware,
    (req, res, next) => {
        // ...
    }
);
```
login-middleware.js 파일에는 아래와 같이 작성합니다.
```javascript
import jwt from "jsonwebtoken";
import { User } from '../../db/models/users/user-model.js';

async function login-middleware (req, res, next) {
   const { email } = req.params || {};
   const userToken = req.headers["authorization"]?.split(" ")[1] ?? "null";

   if (userToken === "null") {
      res.status(400).json({ message: "토큰이 없습니다."});
      return;
   }

   try {
      const secretKey = "jwt-secret-key";
      const jwtDecoded = jwt.verify(userToken, secretKey);

      const currentUser = await User.findOne(
         {email: email},
         { _id: 1}
      );
      
      if(currentUser._id == jwtDecoded.user_id) {
         next();
         return;
      }
         
      res.status(400).json({ message : "권한이 없습니다."});
         
   } catch (error) {
      if (error.name === "TokenExpiredError") {
         res.status(400).json({ message: "토큰이 만료되었습니다." });
      } else {
         res.status(400).json({ message: "정상적인 토큰이 아닙니다." });
      }
      return;
   }
}
  
export { login-middleware };
```
&nbsp;

위 코드를 통해 GET 요청에 담긴 이메일 주소와 일치하는 사용자를 User 모델에서 찾고, 해당 유저의 id와 토큰의 user_id가 일치하는 지 검증하고, 일치한다면 next();를 통해 미들웨어로 요청을 넘기고, 일치하지 않는다면 400 에러와 함께 권한이 없다는 응답을 반환합니다.
&nbsp;

또한 토큰의 유효기간이 지난 경우와 정상적인 토큰이 아닐 경우에도 그에 맞는 에러 응답을 반환할 수 있습니다.

&nbsp;
&nbsp;

---
&nbsp;
[![Elice UTM Banner](/blog/assets/elice/SW7_jihoonkim_bottom_banner.png)](https://elice.training/track/sw?utm_source=sw7&utm_medium=blog&utm_campaign=challenge&utm_content=m2gzitm8b)
&nbsp;
> 태그 Tag : #엘리스트랙 #엘리스트랙후기 #리액트네이티브강좌 #온라인코딩부트캠프 #온라인코딩학원 #프론트엔드학원 #개발자국비지원 #개발자부트캠프 #국비지원부트캠프 #프론트엔드국비지원 #React #Styledcomponent #React Router Dom #Redux #Typescript #Javascript