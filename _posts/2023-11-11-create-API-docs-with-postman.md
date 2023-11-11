---
title: 포스트맨 Postman 으로 API 명세서 만들기
updated: 2023-11-11 16:01
---

> Elice SW Track 7기 - 8주차 진행 내용입니다.


![Elice Banner](/blog/assets/elice/SW7_top_banner.png)

&nbsp;

**데이터베이스**(**DB**, 또는 **DBMS**)의 타입에는 관계형 데이터베이스 (RDB, Relational DB)와 NoSQL 데이터베이스 (NoSQL DB) 가 존재합니다. 각각의 데이터베이스 타입들은 사용하는 용도와 장단점이 명확합니다. 보통 RDB는 데이터간의 관계가 중요한 서비스에서 사용되며, NoSQL DB는 데이터 간의 관계가 크게 중요하지 않거나 간단한 CRUD 기반의 서비스에 적합합니다. 이번에는 NoSQL DB 중 하나인 **MongoDB**에서 사용되는 모델의 스키마를 작성해보도록 하겠습니다.

&nbsp;

## 목차
[1. Postman이란?](#1-postmand이란)

[2. 콜렉션 Collection 생성하고 관리하기](#2-콜렉션-collection-생성하고-관리하기)

[3. HTTP Method 별로 리퀘스트 Request 생성하고 요청 샘플 저장하기](#3-http-method-별로-리퀘스트-request-생성하고-요청-샘플-저장하기)

[4. 도큐멘테이션 작성하기](#4-도큐멘테이션-작성하기)

[5. 도큐먼트 퍼블리싱하기](#5-도큐먼트-퍼블리싱하기)


&nbsp;

---

&nbsp;
## 1. Postman이란?

**Postman**은 API 테스트와 문서화, 협업을 할 수 있는 플랫폼 입니다. API 리퀘스트를 콜렉션 단위로 모아두고 관리할 수 있으며, 별도로 문서를 작성하지 않아도 도큐멘테이션을 작성할 수 있는 기능도 제공합니다.
> Postman : [링크](https://www.postman.com/)

&nbsp;

## 2. 콜렉션 Collection 생성하고 관리하기

Postman에 가입하면 기본적으로 1개의 **Workspace**에 참여하게 됩니다. (그룹 워크스페이스를 생성하여 여러 사람들과 협업할 수도 있지만 여기서는 개인 워크스페이스를 사용합니다) 아래 화면처럼 여러개의 콜렉션 (사용자, 상품, 주문, 카테고리)을 생성하여 그룹별로 API를 관리할 수 있습니다.

&nbsp;

![Collections](/blog/assets/posts/asset-postman-00-collection.png)

&nbsp;

콜렉션에 새로운 API 리퀘스트를 추가하려면, 콜렉션 우측 더보기 버튼을 클릭하여 Add request 메뉴를 실행하여 새로운 리퀘스트를 생성합니다. (Default HTTP Method는 "GET") 

&nbsp;

![New collection](/blog/assets/asset-postman-01-new-request.png)

&nbsp;

새로운 리퀘스트를 만들고 HTTP Method, 리퀘스트의 이름, params, request body, request header 등을 설정할 수 있습니다. 아래는 미리 생성해둔 회원가입 API 리퀘스트 화면입니다. POST 요청이기 때문에 params 필드는 사용하지 않고 request body에만 API로 요청할 값을 JSON 형태로 작성한 것을 확인할 수 있습니다.


&nbsp;
화면 우측의 **Send** 버튼을 클릭하여 설정된 주소로 API 요청을 보낼 수 있으며, 화면 하단에 요청에 대한 응답 결과가 나타납니다. 아래 화면은 서버 측으로 보낸 요청이 성공하여 status code를 201로 받았으며, 요청에 대한 응답으로 성공 메시지와 db에 생성된 user 객체의 정보를 반환 받았다는 의미입니다. 

&nbsp;

![signup request](/blog/assets/posts/asset-postman-02-signup-post-request.png)

&nbsp;

그럼 이제 서버측 코드에는 어떻게 작성되어 있는지 확인해보겠습니다.

&nbsp;

> router.js

router에서는 req.body에 담긴 값을 userService의 Signup 메소드의 인자로 넘겨 리턴값을 받아와서, 성공 또는 실패 응답을 response 객체에 담아 보냅니다.

```javascript
;import UserService from '../../services/user-service.js';
import { validator_signup } from "../../middlewares/validator/validator-signup.js";
import { Router } from 'express';
const router = Router();

const userService = new UserService;

/*
 * 회원가입 요청
 */
router.post("/",
    validator_signup,
    async (req, res, next) => {
                
        try {
            const result = await userService.Signup(req.body);
            if(result.message === "SUCCESS") {
                res.status(201)
                .json({
                        message: "회원가입에 성공했습니다.",
                        user: result.user,
                      });
                return;
            } else {
                throw {status: 404, message: 'unknown error'};
            }
            
        } catch(err) {
            res.status(err.status)
            .json({ message: err.message,})
        }
    }
);

export default router;
```
&nbsp;
> user-service.js

user-service.js에서는 인자로 전달받은 req.body객체의 userName, email, password, role 정보를 받아와서 db에서 해당 이메일로 가입된 도큐먼트가 존재하는 지 검증하고, 존재하지 않다면 새로운 user 도큐먼트를 생성하여 저정하고, SUCCESS 메시지와 함께 생성된 user 객체를 리턴합니다.

```javascript
import bcrypt from 'bcryptjs';
import { User } from '../db/models/users/user-model.js';

export default class UserService {

    async Signup({userName, email, password, role = "USER"}) {
        const user = {userName, email, password, role};

        const existUser = await User.findOne( {email: user.email} );
        if(existUser !== null) {
            throw {status: 400, message: '이미 가입한 이메일입니다.'};
        } 
        
        const salt = await bcrypt.genSalt(10);
        const hash = await bcrypt.hash(user.password, salt);
        
        user.password = hash;
        const newUser = await User.create(user);
        return { message : "SUCCESS", user: newUser};
    }
}
```


&nbsp;

---
&nbsp;

[![Elice UTM Banner](/blog/assets/elice/SW7_jihoonkim_bottom_banner.png)](https://elice.training/track/sw?utm_source=sw7&utm_medium=blog&utm_campaign=challenge&utm_content=m2gzitm8b)
&nbsp;
> 태그 Tag : #엘리스트랙 #엘리스트랙후기 #리액트네이티브강좌 #온라인코딩부트캠프 #온라인코딩학원 #프론트엔드학원 #개발자국비지원 #개발자부트캠프 #국비지원부트캠프 #프론트엔드국비지원 #React #Styledcomponent #React Router Dom #Redux #Typescript #Javascript