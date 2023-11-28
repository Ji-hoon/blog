---
layout: post
title: MongoDB에서 Model Schema로 Document 생성하기
updated: 2023-10-30 00:01
---

> Elice SW Track 7기 - 8주차 진행 내용입니다.


![Elice Banner](/blog/assets/elice/SW7_top_banner.png)

&nbsp;

**데이터베이스**(**DB**, 또는 **DBMS**)의 타입에는 관계형 데이터베이스 (RDB, Relational DB)와 NoSQL 데이터베이스 (NoSQL DB) 가 존재합니다. 각각의 데이터베이스 타입들은 사용하는 용도와 장단점이 명확합니다. 보통 RDB는 데이터간의 관계가 중요한 서비스에서 사용되며, NoSQL DB는 데이터 간의 관계가 크게 중요하지 않거나 간단한 CRUD 기반의 서비스에 적합합니다. 이번에는 NoSQL DB 중 하나인 **MongoDB**에서 사용되는 모델의 스키마를 작성해보도록 하겠습니다.

&nbsp;

## 목차
[1. MongoDB란?](#1-mongodb란)

[2. Atlas 프로젝트 환경 구축하기](#2-atlas-프로젝트-환경-구축하기)

[3. 개발 환경에서 MongoDB에 접근하기](#3-개발-환경에서-mongodb에-접근하기)

[4. 스키마 Schema 정의하고 도큐먼트 추가하기](#4-스키마-schema-정의하고-도큐먼트-추가하기)


&nbsp;

---

&nbsp;
## 1. MongoDB란?

MongoDB는 NoSQL DBMS중 하나로, JSON과 유사한 BSON 타입을 지원하기 때문에 웹 생태계에서 인기있는 DBMS 입니다. RDB 대비 비교적 설계가 단순하기 때문에 진입 장벽이 낮아 인기가 많습니다. 아래 링크에서 더 많이 알아보실 수 있으며 이번에는 MongoDB Atlas를 활용해서 모델(도큐먼트)를 만드는 것 까지 해보겠습니다.
> MongoDB : [링크](https://www.mongodb.com/ko-kr){:target="_blank"}

&nbsp;

## 2. Atlas 프로젝트 환경 구축하기

MongoDB에 가입하면 **Cloud SaaS** 형태로 제공되는 **Atlas** 또는 머신에 직접 설치하는 **on-premise** 버전이 제공되는데, 여기서는 Cloud 형태의 Atlas를 사용합니다.

&nbsp;

![New Project](/blog/assets/posts/asset-mongodb-atlas-new-project.png)

&nbsp;

[MongoDB Atlas 등록](https://www.mongodb.com/ko-kr/cloud/atlas/register) 화면에서 가입 후 로그인을 하게 되면, 새로운 조직 Organization이 자동으로 생성됩니다. 위 화면 처럼 조직 > 프로젝트 목록 화면에서 **새로운 프로젝트**를 생성합니다.

&nbsp;

![New Database](/blog/assets/posts/asset-mongodb-atlas-new-database.png)

&nbsp;

새로운 프로젝트를 만들면 이제 위 화면의 **Create** 버튼을 클릭하여 클러스터 생성 화면으로 넘어갑니다. 테스트 용도이기 때문에 무료 Free 버전으로, Provider는 AWS, Region은 Seoul을 선택해주고 클러스터 이름을 입력한다음 **Create**를 해줍니다.

&nbsp;

![Quick start - User](/blog/assets/posts/asset-mongodb-atlas-quickstart-user.png)

&nbsp;

클러스터를 생성 했다면, 위 화면처럼 Security Quickstart 화면으로 넘어가게 됩니다. (이 화면은 좌측 메뉴의 Security > Quickstart 메뉴로도 진입할 수 있습니다)

&nbsp;

여기서는 Username과 password, 접근할 개발환경을 설정하게 됩니다. 여기서 설정한 값들은 개발 환경에서 MongoDB에 접근할 때 사용하게 됩니다. **Create User** 버튼을 클릭하여 접속 계정을 생성합니다.


&nbsp;

![Quick start - IP](/blog/assets/posts/asset-mongodb-atlas-quickstart-ip.png)

&nbsp;

그 다음으로는 DB에 접근할 환경을 설정하게 되는데, 로컬 개발 환경에서 접근할 예정이므로 Local Environment로 설정하고, **Add my currnnt IP** 버튼을 클릭하여 본인의 IP를 추가하고 저장합니다.

&nbsp;

![Quick start - Overview](/blog/assets/posts/asset-mongodb-atlas-step3-overview.png)

&nbsp;

![Quick start - New Collection](/blog/assets/posts/asset-mongodb-atlas-step3-new-collection.png)

&nbsp;

그럼 이제 Database 메뉴에서 Browse Collection 버튼을 눌러 데이터베이스와 콜렉션을 생성합니다. 클러스터 하위에 여러 개의 데이터베이스가 존재하고, 데이터베이스 하위에 여러개의 콜렉션이, 그리고 마지막으로 콜렉션 하위에는 여러 개의 도큐먼트가 존재하게 됩니다.

&nbsp;

![Quick start - Created Collection](/blog/assets/posts/asset-mongodb-atlas-step3-created-collection.png)

&nbsp;

데이터베이스와 콜렉션을 만들었다면 위 화면 처럼 **user-model** 데이터베이스 하위에 **users** 콜렉션이 존재하게 됩니다.

&nbsp;

![Quick start - Connect](/blog/assets/posts/asset-mongodb-atlas-step4-connect.png)

&nbsp;

![Quick start - Drivers ](/blog/assets/posts/asset-mongodb-atlas-step4-drivers.png)

&nbsp;

![Quick start - Copy Connection Code](/blog/assets/posts/asset-mongodb-atlas-step4-connection-code.png)

&nbsp;

다시 데이터베이스 화면으로 돌아가서, db에 연결할 수 있는 주소를 얻기 위해 Connect 버튼을 클릭해서 모달로 진입하고, Driver 메뉴를 클릭하면 표시되는 화면에서 복사 버튼을 클릭해서 접근 주소를 복사할 수 있습니다.

&nbsp;

접근 주소는 아래 처럼 필요한 정보를 넣어 수정합니다.


```javascript
// 변수 적용 전
const connectionURL = "mongodb+srv://{username}:{password}@cluster0.6fpsvld.mongodb.net/{databse name}?retryWrites=true&w=majority"

// username이 test, password가 testpassword, 그리고 접근할 database name이 user-model 일 때
const connectionURL = "mongodb+srv://test:testpassword@cluster0.6fpsvld.mongodb.net/user-model?retryWrites=true&w=majority"
```

&nbsp;

## 3. 개발 환경에서 MongoDB에 접근하기

Atlas에서 Database 환경을 구축하고 접근 주소를 생성했다면, 이제 로컬 개발환경에서 MongoDB에 접근해보도록 하겠습니다.

&nbsp;

먼저 터미널에서 MongoDB와 Mongoose를 설치합니다.
```shell
$ npm install mongodb
$ npm install mongoose
```
&nbsp;

index.js 파일 안에 mongoose를 import 하여 사용합니다.

> 아래 예시에서는 connectionURL 변수에 username과 password를 그대로 작성했지만, 환경 변수를 만들어서 따로 사용하는 것이 좋습니다.

```javascript
import express from "express";
import mongoose from 'mongoose';

const port = 3000;
const app = express();

const connectionURL = "mongodb+srv://testuser:testpassword@cluster0.6fpsvld.mongodb.net/user-model?retryWrites=true&w=majority";

mongoose
  .connect(connectionURL, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  })
  .then(() => console.log("Connected to MongoDB"))
  .catch((err) => console.log(err));

app.listen(port);
```

이제 터미널에서 작성한 index.js를 실행합니다. MongoDB와 연결되면 터미널에 `Connected to MongoDB` 문구가 표시되는 것을 확인할 수 있습니다. 
```shell
$ node index.js
```


&nbsp;

## 4. 스키마 Schema 정의하고 도큐먼트 추가하기

마지막으로 스키마를 작성해서, 연결된 콜렉션에 도큐먼트 Document 를 추가하는 작업을 해보겠습니다.

&nbsp;

스키마는 데이터베이스에 데이터가 저장되는 형태 or 구조를 의미합니다. NoSQL DB는 엄격한 설계를 따르지는 않지만 MongoDB에서는 스키마를 정의해두면 도큐먼트 생성 또는 수정 시 데이터 타입을 체크해주는 기능을 제공합니다. 

&nbsp;
아래의 코드는 User 라는 사용자 모델에 대한 스키마 입니다.
```javascript
import mongoose from 'mongoose';
const { Schema } = mongoose;
import shortId from '../types/short-id.js';

const userSchema = new Schema({
	shortId,
	role: {
		type: String,
		default: "USER",
	},
	userName: { 
		type: String,
		required: true, 
	},
	email: {
		type: String,
		required: true,
	},
	password: { 
		type: String,
		required: true 
	},
	phone: {
		type: String,
		default : null,
	},
	age: {
		type: Number,
		default : null,
	},
	address: {
		type: String,
		default : null,
	},
});

const User = mongoose.model('User', userSchema);
export { User };
```
위 코드를 통해 User라는 MongoDB 모델(도큐먼트) 생성 시 스키마를 따라 생성이 되게됩니다. 다음과 같은 코드를 통해 MongoDB에 도큐먼트를 생성할 수 있습니다.
&nbsp;

```javascript
import { User } from '../db/models/users/user-model.js';

const newUser = {
    "userName" : "mike",
    "email" : "mike@abc.com",
    "password" : "abc",
    "role" : "ADMIN"
}

User.create(newUser);
```
&nbsp;
create라는 메소드를 사용해서 newUser 객체에 담긴 정보로 User 도큐먼트를 생성하게 됩니다. 코드를 실행하여 실제 DB에 생성되는 도큐먼트 형태는 아래와 같습니다. (user-model > users 콜렉션 하위에 생성)
&nbsp;

![새롭게 생성된 도큐먼트](/blog/assets/posts/asset-mongodb-model-example.png)

&nbsp;
스키마에서 필수값으로 설정된 userName(이름), email(이메일 주소),password(비밀번호)를 제외한 값이 없어도 User 도큐먼트를 생성할 수 있습니다.

&nbsp;

---
&nbsp;

[![Elice UTM Banner](/blog/assets/elice/SW7_jihoonkim_bottom_banner.png)](https://elice.training/track/sw?utm_source=sw7&utm_medium=blog&utm_campaign=challenge&utm_content=m2gzitm8b)
&nbsp;
> 태그 Tag : #엘리스트랙 #엘리스트랙후기 #리액트네이티브강좌 #온라인코딩부트캠프 #온라인코딩학원 #프론트엔드학원 #개발자국비지원 #개발자부트캠프 #국비지원부트캠프 #프론트엔드국비지원 #React #Styledcomponent #React Router Dom #Redux #Typescript #Javascript