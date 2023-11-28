---
title: 포스트맨 Postman 으로 API 명세서 만들기
updated: 2023-11-05 16:01
---

> Elice SW Track 7기 - 8주차 진행 내용입니다.


![Elice Banner](/blog/assets/elice/SW7_top_banner.png)

&nbsp;

**API**는 Application Programming Interface의 약자로, 서버와 클라이언트간 통신을 하며 데이터를 주고 받는 규약을 의미합니다. 일반적으로 request(요청)와 response(응답) 2가지로 구성되며, 클라이언트는 API를 통해 서버에서 정보를 받아오거나 추가 또는 삭제하는 요청을 하게됩니다. 서버는 이 요청을 받아서 비즈니스 로직에 맞게 구현된 기능을 수행한 뒤 다시 클라이언트로 응답을 보내는 일련의 사이클을 돌게됩니다.

&nbsp;

이 API는 통상 서버 사이드에서 작성을 하게 되는데, 협업을 하게되면 클라이언트 개발자들에게 작성된 API를 ① 어떤 방식(method)으로 ② 어떤 데이터를 보내서 ③ 어떠한 응답을 받게 되는지 가이드(명세서)를 제공하는 것이 중요합니다. 오늘은 그러한 가이드를 쉽게 작성할 수 있게 도와주는 Postman을 통해서 명세서를 작성해보겠습니다.


&nbsp;

## 목차
[1. Postman이란?](#1-postman이란)

[2. 콜렉션 Collection과 리퀘스트 Request](#2-콜렉션-collection과-리퀘스트-request)

[3. 리퀘스트 요청 샘플 저장하기](#3-리퀘스트-요청-샘플-저장하기)

[4. 리퀘스트 도큐멘테이션 작성하기](#4-리퀘스트-도큐멘테이션-작성하기)

[5. 콜렉션 도큐먼트 퍼블리싱하기](#5-콜렉션-도큐먼트-퍼블리싱하기)


&nbsp;

---

&nbsp;
## 1. Postman이란?

**Postman**은 API 테스트와 문서화 할 수 있는 플랫폼 입니다. API 리퀘스트를 콜렉션 단위로 모아두고 관리할 수 있으며, 별도로 문서를 작성하지 않아도 도큐멘테이션(명세서)을 작성할 수 있는 기능도 제공합니다.
> Postman : [링크](https://www.postman.com/){:target="_blank"}

&nbsp;

## 2. 콜렉션 Collection과 리퀘스트 Request

Postman에 가입하면 기본적으로 1개의 **Workspace**에 참여하게 됩니다. (그룹 워크스페이스를 생성하여 여러 사람들과 협업할 수도 있지만 여기서는 개인 워크스페이스를 사용합니다) 아래 화면처럼 여러개의 콜렉션 (사용자, 상품, 주문, 카테고리)을 생성하여 그룹별로 API를 관리할 수 있습니다.

&nbsp;

![Collections](/blog/assets/posts/asset-postman-00-collection.png)

&nbsp;

콜렉션에 새로운 API 리퀘스트를 추가하려면, 콜렉션 우측 더보기 버튼을 클릭하여 Add request 메뉴를 실행하여 새로운 리퀘스트를 생성합니다. (Default HTTP Method는 "GET") 

&nbsp;

![New collection](/blog/assets/posts/asset-postman-01-new-request.png)

&nbsp;

새로운 리퀘스트를 만들게 되면 HTTP Method, 리퀘스트의 이름, params, request body, request header 등을 설정할 수 있습니다. 아래는 미리 생성해둔 회원가입 API 리퀘스트 화면입니다. POST 요청이기 때문에 params 필드는 사용하지 않고 request body에만 API로 요청할 값을 JSON 형태로 작성한 것을 확인할 수 있습니다.

&nbsp;

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

## 3. 리퀘스트 요청 샘플 저장하기

이렇게 리퀘스트를 만들고 나면 다양한 테스트 케이스에서 정상 동작을 하는 지 확인하기 위해 다양한 값들을 넣어 보면서 코드의 안정성을 검증하게 되는데, Postman에서는 이 때 API 요청과 응답 결과를 하나의 템플릿 형태로 저장할 수 있는 Save as example 기능을 제공합니다.

&nbsp;

![example request](/blog/assets/posts/asset-postman-03-save-example.png)

&nbsp;

이 기능을 사용해 저장하면, 현재 요청과 응답 값을 그대로 저장하게 됩니다. 아래 화면은 201 응답 코드를 받은 응답 예시를 저장한 경우입니다. 기존에 만들어둔 리퀘스트 하위에 여러 개의 예시가 존재하는 것을 확인하실 수 있습니다.

&nbsp;

![example 201](/blog/assets/posts/asset-postman-04-201.png)


&nbsp;

## 4. 리퀘스트 도큐멘테이션 작성하기

응답 예시까지 저장했다면, 이제 API를 설명해주는 도큐멘테이션 documentation 을 작성합니다. 리퀘스트 우측의 **Documentation** 버튼을 클릭하면 설명을 입력할 수 있는 화면이 제공되고, 여기에 간략한 설명과 발생 가능한 응답 코드 그리고 해당 코드가 발생하는 조건 등을 기재하여 이해하기 쉽게 기술합니다.

&nbsp;

![documentation](/blog/assets/posts/asset-postman-05-documentation.png)


&nbsp;

## 5. 콜렉션 도큐먼트 퍼블리싱하기

이제 리퀘스트 별로 도큐멘테이션을 작성하고, 응답 예시도 저장했다면 본인 계정 뿐만아니라 다른 협업하는 사람들에게도 확인할 수 있도록 콜렉션 문서를 **Publish**를 해 보도록 하겠습니다.

&nbsp;

사용자 콜렉션을 클릭해서 사용자 콜렉션 상세 화면으로 진입합니다. 이어서 화면 하단에 **View complete documentation** 버튼을 클릭합니다. 

&nbsp;

![complete documentation](/blog/assets/posts/asset-postman-06-complete-doc.png)

&nbsp;

화면 우측 상단의 **Publish**를 클릭하여 다음 화면으로 진입합니다.

&nbsp;

![publish documentation](/blog/assets/posts/asset-postman-07-publish.png)

&nbsp;

화면 하단 **Publish**를 클릭하면 이제 출판된 URL을 통해 API 콜렉션 도큐먼트에 접근할 수 있습니다.

&nbsp;

![publish documentation 2](/blog/assets/posts/asset-postman-08-publish-confirm.png)

> 사용자 API 콜렉션 : [링크](https://documenter.getpostman.com/view/30577749/2s9YXb9kQJ){:target="_blank"}


&nbsp;

---
&nbsp;

[![Elice UTM Banner](/blog/assets/elice/SW7_jihoonkim_bottom_banner.png)](https://elice.training/track/sw?utm_source=sw7&utm_medium=blog&utm_campaign=challenge&utm_content=m2gzitm8b)
&nbsp;
> 태그 Tag : #엘리스트랙 #엘리스트랙후기 #리액트네이티브강좌 #온라인코딩부트캠프 #온라인코딩학원 #프론트엔드학원 #개발자국비지원 #개발자부트캠프 #국비지원부트캠프 #프론트엔드국비지원 #React #Styledcomponent #React Router Dom #Redux #Typescript #Javascript