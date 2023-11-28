---
layout: post
title: Mongoose CRUD 메소드 제대로 이해하고 사용하기
updated: 2023-11-07 01:00
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

> 이전 글 : [MongoDB에서 Model Schema로 Document 생성하기](https://ji-hoon.github.io/blog/mongo-db-schema){:target="_blank"}

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

도큐먼트를 검색하는 메소드로는 **find(`{조건}`)** 와 **findOne(`{조건}`)** 을 주로 사용합니다. 두 메소드의 차이점은 **find**는 조건과 일치하는 **모든** 도큐먼트를 `객체가 담긴 배열` 형태로 반환하지만, **findOne**은 조건과 일치하는 **첫 번째**의 단일 도큐먼트만 `객체` 형태로 반환한다는 점이 다릅니다.

&nbsp;

아래의 예시 코드는 **find** 메소드를 사용해서 전체 사용자의 정보를 조회하는 코드입니다. 조건이 비어있고, 콤마 다음에 또 다른 객체를 넣은 것을 확인할 수 있습니다. 이는 모든 도큐먼트를 검색하지만, userName, email, age, phone, address, role, orderList 필드의 값만 `객체` 형태로 담아 `배열`로 반환합니다.

```javascript
import { User } from '../db/models/users/user-model.js';

export default class UserService {
    async getAllUsersInfo () {
        const result = await User.find(
                    {}, 
                    { userName: 1, email: 1 ,age:1, phone: 1, address:1, role:1, orderList:1, });
    }
}
```

> 반환된 result 배열 정보

```javascript
[
    {
        "_id": "65431dbce1e666bb3a750fe7",
        "role": "USER",
        "userName": "ㅇㅇㅇ",
        "email": "abc@abc.com",
        "phone": "01049157327",
        "age": 12,
        "address": "인천 연수구 12",
        "orderList": []
    },
    {
        "_id": "6545f8a8e46b79bf62610974",
        "role": "ADMIN",
        "userName": "abc",
        "email": "abcdefg@abc.com",
        "phone": null,
        "age": null,
        "address": null
    },
    ...
]
```
&nbsp;

다음은 **findOne** 메소드를 사용해서 단일 사용자의 정보를 조회하는 코드입니다. 조회 조건으로 함수의 인자로 전달받은 shortId와 일치하는 1개의 도큐먼트를 찾고, userName, email, age, phone, addres 필드의 값만 담긴 `객체`를 반환 받습니다.

```javascript
import { User } from '../db/models/users/user-model.js';

export default class UserService {
    async getUserInfo (shortId) {
        const result = await User.findOne(
                    { shortId: shortId }, 
                    { userName: 1, email: 1 ,age:1, phone: 1, address:1 });
    }
}
```
> 반환된 result 객체 정보

```javascript
{
     "_id": "65431dbce1e666bb3a750fe7",
    "userName": "ㅇㅇㅇ",
    "email": "abc@abc.com",
    "phone": "01049157327",
    "age": 12,
    "address": "인천 연수구 12",
}
```
&nbsp;

## 4. 도큐먼트를 수정하는 메소드

도큐먼트를 수정하는 메소드로는 **update(`{조건}`, `{수정할 정보}`)** 와 **findOneAndUpdate(`{조건}`, `{수정할 정보}`)** 을 주로 사용합니다. 두 메소드의 차이점은 **update**는 조건과 일치하는 **모든** 도큐먼트를 수정하고, **findOneAndUpdate**은 조건과 일치하는 **단일** 도큐먼트만 수정한다는 점이 다릅니다.

&nbsp;

아래 예시는 단일 도큐먼트만 수정하고, 수정된 도큐먼트를 반환받도록 동작하는 코드입니다. 검색 조건으로 shortId와 일치하는 도큐먼트를 찾고, data 인자로 넘어온 객체 중에서 userName, age, phone, address를 업데이트한 뒤 matchedUser객체에 업데이트된 도큐먼트를 반환합니다. 수정된 도큐먼트를 반환 받으려면, `{수정할 정보}` 다음에 `{ new:true}` 옵션을 추가해야 합니다.

```javascript
import { User } from '../db/models/users/user-model.js';

export default class UserService {
    
    async updateUserInfo (shortId, data) {
        const matchedUser = await User.findOneAndUpdate( 
                { shortId: shortId }, 
                { userName: data.userName, age: data.age, phone: data.phone, address:data.address, },
                { new: true } );
        if(!matchedUser) {
            throw { status: 404, message: "NO_MATCHES", };
        }
        return { message: "SUCCESS", user: matchedUser};
    }
}
```
&nbsp;

## 5. 도큐먼트를 삭제하는 메소드

도큐먼트를 삭제하는 메소드로는 **delete(`{조건}`)** 와 **findOneAndDelete(`{조건}`)** 을 주로 사용합니다. 두 메소드의 차이점은 **delete**는 조건과 일치하는 **모든** 도큐먼트를 삭제하고, **findOneAndDelete**은 조건과 일치하는 **단일** 도큐먼트만 삭제한다는 점이 다릅니다.

&nbsp;

아래 예시는 조건을 충족하는 단일 도큐먼트만 삭제하도록 동작하는 코드입니다. 검색 조건으로 shortId와 일치하는 도큐먼트를 찾고, 일치하는 도큐먼트가 있다면 도큐먼트를 삭제합니다. 만약 조건에 일치하는 도큐먼트가 없다면 `null`을 반환합니다. 따라서 이 반환값을 기준으로 에러 처리를 정의할 수 있습니다.

```javascript
import { User } from '../db/models/users/user-model.js';

export default class UserService {
    
    async deleteUserInfo (shortId) {
        const deletedUser = await User.findOneAndDelete( 
            { shortId: shortId });
        //return deletedUser //조건에 맞지 않으면 null을 반환
        
        if(!deletedUser) {
            throw {status: 404, message: "존재하지 않는 계정입니다.",};
        }
        return { message: "SUCCESS", };
    }

}
```


&nbsp;

---
&nbsp;

[![Elice UTM Banner](/blog/assets/elice/SW7_jihoonkim_bottom_banner.png)](https://elice.training/track/sw?utm_source=sw7&utm_medium=blog&utm_campaign=challenge&utm_content=m2gzitm8b)
&nbsp;
> 태그 Tag : #엘리스트랙 #엘리스트랙후기 #리액트네이티브강좌 #온라인코딩부트캠프 #온라인코딩학원 #프론트엔드학원 #개발자국비지원 #개발자부트캠프 #국비지원부트캠프 #프론트엔드국비지원 #React #Styledcomponent #React Router Dom #Redux #Typescript #Javascript