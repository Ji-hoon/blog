---
title: Node.js Multer로 파일 업로드 기능 구현하기
updated: 2023-11-09 01:00
---

> Elice SW Track 7기 - 9주차 진행 내용입니다.


![Elice Banner](/blog/assets/elice/SW7_top_banner.png)

&nbsp;

클라이언트 측에서 서버에 json 뿐만 아니라 file을 요청하여 저장하는 기능을 구현해야 할 때가 있습니다. 이 때 다양한 Node.js 미들웨어가 있겠지만, 가장 많이 사용되는 **Multer** 미들웨어를 활용하여 파일 업로드 기능을 구현해 보도록 하겠습니다.


&nbsp;

## 목차
[1. Multer란?](#1-multer란)

[2. Multipart / form-data 이해하기](#2-multipart--form-data-이해하기)

[3. 클라이언트 코드 작성하기](#3-클라이언트-코드-작성하기)

[4. 서버 코드 작성하기](#4-서버-코드-작성하기)


&nbsp;

---

&nbsp;
## 1. Multer란?

**Multer**는 파일 업로드를 위해 사용되는 multipart/form-data 를 다루기 위한 node.js 의 미들웨어 입니다. 아래 명령어를 통해 설치합니다. Multer는 multipart (multipart/form-data)가 아닌 form에서는 동작하지 않습니다.

```shell
$ npm install --save multer
```
> Multer : [링크](https://www.npmjs.com/package/multer)

&nbsp;
## 2. Multipart / form-data 이해하기

Multipart/form-data는 일반적으로 `<form>` 요소를 사용하여 파일 업로드와 함께 데이터를 전송할 때 사용됩니다. form 엘리먼트의 `encType` 어트리뷰트를 `multipart/form-data`로 정의하여 사용하며, 모든 문자를 인코딩하지 않음을 의미합니다. 주로 **이미지**나 **파일**을 서버로 전송할 때 사용합니다.

&nbsp;

```html
<form encType="multipart/form-data">
    <input type="file" name="image">
</form>
```

&nbsp;
## 3. 클라이언트 코드 작성하기

위에서 정의한 form을 이용하여 서버로 API 요청을 보내는 코드를 작성해 보겠습니다. form 내 input에 파일이 선택되면 `formData` 객체에 파일을 `'image'` 키로 담아 사전 정의된 `uploadUrl`에 `POST` method로 `formData`를 담아 보냅니다.

```javascript
const fileInput = document.querySelector("form input");
const formData = new FormData();
fileInput.addEventListener("input", async () => {
    formData.append('image', fileInput.files[0]);
    fetch(uploadUrl,
        {
            method: "POST",
            body: formData,
        })
        .then(res => {
            if (res.ok) { // 수정: 응답이 정상적으로 도착한 경우에만 처리합니다.
                return res.json(); // JSON 형식으로 응답을 파싱합니다.
            } else {
                throw new Error('파일 업로드 실패'); // 응답이 실패한 경우 오류를 throw합니다.
            }
        })
        .then(data => {
            console.log(data); // 파일 업로드 성공 시 서버 응답을 확인합니다.
        })
        .catch(err => {
            console.log(err); // 오류 처리
        });
    }
);
```

&nbsp;
## 4. 서버 코드 작성하기

서버 단에서는 POST method로 들어온 API 요청을 Multer 미들웨어에서 처리하여 서버의 스토리지 정의한 form을 이용하여 서버로 API 요청을 보내는 코드를 작성해 보겠습니다. ESM 형식을 사용하기 떄문에 path 모듈을 추가로 설치합니다.

```shell
$ npm install path
```

&nbsp;
express와 multer, path를 import하고 업로드된 파일이 저장될 경로를 지정하고, 원본 파일의 확장자를 가져와서 저장할 파일명을 저장합니다.
&nbsp;

```javascript
import { Router } from 'express';
const router = Router();

import multer from 'multer';
import path from "path";

const storage = multer.diskStorage({
    destination: function(req, file, cb) {
        cb(null, 'public/assets/cars/'); // 업로드된 파일이 저장될 경로를 지정합니다.
    },
    filename: function(req, file, cb) {
        const extname = path.extname(file.originalname); // 원본 파일의 확장자를 가져옵니다.
       const filename = file.originalname.replace(extname, '').toLowerCase(); // 파일명을 설정합니다. 여기서는 확장자를 제외하고 소문자로 변환합니다.
        cb(null, filename + '-' + Date.now() + extname); // 저장할 파일명을 설정합니다. 이 예제에서는 파일명에 타임스탬프를 추가합니다.
    }
});
const upload = multer({ storage: storage });
```
&nbsp;
다음은 POST 요청을 처리할 router를 정의하는 부분입니다. upload가 성공했다면, req에 담긴 file 객체의 filename을 클라이언트에 반환하여 클라이언트에서 활용할 수 있도록 합니다.
&nbsp;

```javascript
router.post("/",
    upload.single('image'),
    async (req,res,next) => {
        try {
            const result = `/images/${req.file.filename}`;
            if(!result) {
                throw {status: "400", message: "업로드 중에 오류가 발생했습니다."}
            }
            res.status(201).json({message: "업로드에 성공했습니다.", img: result});
        } catch (err) {
            res.status(err.status).json({message:err.message});
        }
    }
);

export default router;
```



&nbsp;

---
&nbsp;

[![Elice UTM Banner](/blog/assets/elice/SW7_jihoonkim_bottom_banner.png)](https://elice.training/track/sw?utm_source=sw7&utm_medium=blog&utm_campaign=challenge&utm_content=m2gzitm8b)
&nbsp;
> 태그 Tag : #엘리스트랙 #엘리스트랙후기 #리액트네이티브강좌 #온라인코딩부트캠프 #온라인코딩학원 #프론트엔드학원 #개발자국비지원 #개발자부트캠프 #국비지원부트캠프 #프론트엔드국비지원 #React #Styledcomponent #React Router Dom #Redux #Typescript #Javascript