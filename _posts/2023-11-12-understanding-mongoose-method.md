---
title: Mongoose 메소드 제대로 이해하고 사용하기
updated: 2023-11-12 07:00
---

> Elice SW Track 7기 - 9주차 진행 내용입니다.


![Elice Banner](/blog/assets/elice/SW7_top_banner.png)

&nbsp;

MongoDB를 활용하여 비즈니스 로직을 구현할 때, DB에 접근하여 데이터를 수정하거나 추가하는 작업, 또는 삭제하는 로직을 구현해야하는 경우가 존재합니다. 기본적으로 Node.js 는 JavaScript 문법을 사용하기 때문에 로직을 짜는 건 어렵지 않겠지만 Mongoose에서 제공하는 메소드의 동작과 반환값을 이해하는 것이 선행되어야 오류 없는 코드를 작성할 수 있습니다.


&nbsp;

## 목차
[1. Mongoose란?](#1-mongoose란)

[2. 도큐먼트를 생성하는 메소드](#2-도큐먼트를-생성하는-메소드)

[3. 도큐먼트를 검색하는 메소드](#3-도큐먼트를-검색하는-메소드)

[4. 도큐먼트를 수정하는 메소드](#4-도큐먼트를-수정하는-메소드)

[5. 도큐먼트를 삭제하는 메소드](#5-도큐먼트를-삭제하는-메소드)


&nbsp;

---

&nbsp;
## 1. Mongoose란?

**Mongoose**는 MongoDB 객체 모델링 도구로서, Express.js에서 사용 가능한 모듈입니다. 아래 명령어를 통해 설치합니다.

```shell
$ npm install mongoose
```

이전에 작성했던 User 모델을 기준으로 주로 사용되는 메소드들을 알아보겠습니다.

> 이전 글 : [MongoDB에서 Model Schema로 Document 생성하기](https://ji-hoon.github.io/blog/mongo-db-schema)

&nbsp;

## 2. 도큐먼트를 생성하는 메소드

도큐먼트를 생성하는 메소드로는 **create({생성할 객체 정보})** 를 사용합니다. 아래처럼 메소드의 인자로 생성할 도큐먼트의 정보를 객체 형태로 전달하면, 생성된 도큐먼트 정보를 `객체` 형태로 반환합니다.

```javascript
import { User } from '../db/models/users/user-model.js';

export default class UserService {

    async Signup({userName, email, password, role = "USER"}) {
        // ... 비밀번호 hash 및 중복 가입 검증 로직 
        const user = {userName, email, password, role};
        const newUser = await User.create(user);
    }
}
```

> 반환된 newUser 객체 정보

```javascript
 {
    "role": "USER",
    "userName": "abc",
    "email": "abcdef@abc.com",
    "password": "$2a$10$XtJC5AvgovydGIJZ9mOOH.469hdVzmI3nauqz4LU9/oYvSsBChEve",
    "orderList": [],
    "_id": "65426f00635ce27005861636",
    "shortId": "FJxpmLpYNYyq-yaCuseMj",
    "__v": 0
}
```

&nbsp;

## 3. 도큐먼트를 검색하는 메소드

도큐먼트를 검색하는 메소드로는 **find({조건})** 와 **findOne({조건})** 을 주로 사용합니다. 두 메소드의 큰 차이점은 **find**는 조건과 일치하는 **모든** 도큐먼트를 `객체가 담긴 배열` 형태로 반환하지만, **findOne**은 조건과 일치하는 **1개**의 도큐먼트만 `객체` 형태로 반환한다는 점이 다릅니다.



&nbsp;

---
&nbsp;

[![Elice UTM Banner](/blog/assets/elice/SW7_jihoonkim_bottom_banner.png)](https://elice.training/track/sw?utm_source=sw7&utm_medium=blog&utm_campaign=challenge&utm_content=m2gzitm8b)
&nbsp;
> 태그 Tag : #엘리스트랙 #엘리스트랙후기 #리액트네이티브강좌 #온라인코딩부트캠프 #온라인코딩학원 #프론트엔드학원 #개발자국비지원 #개발자부트캠프 #국비지원부트캠프 #프론트엔드국비지원 #React #Styledcomponent #React Router Dom #Redux #Typescript #Javascript