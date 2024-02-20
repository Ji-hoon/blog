---
title: Google Cloud에 프로젝트 배포하기
updated: 2024-02-07 12:00
---

> '가계부를 부탁해' 프로젝트를 진행하며 작성한 내용입니다.

[![project-logo](https://github.com/Ji-hoon/Home-accountant/raw/master/client/public/img-logo.png)](https://github.com/Ji-hoon/Home-accountant){:target="_blank"}

&nbsp;

새롭게 개인 프로젝트를 진행하면서 다뤘던 기술들에 대해서 기록을 남겨보려 합니다. 오늘은 네 번째 주제로, **구글 클라우드 Google Cloud**에 프로젝트를 배포하는 방법에 대해서 다뤄보려 합니다.

> 프로젝트 보러가기 👉 : [링크](http://35.231.16.39/)

&nbsp;

## 목차

[1. 구글 클라우드 가입하고 가상 머신 생성하기](#1-구글-클라우드-가입하고-가상-머신-생성하기)

[2. 고정 IP 설정하고 터미널에서 VM 접근하기](#2-고정-ip-설정하고-터미널에서-vm-접근하기)

[3. NginX와 환경 변수 설정하기](#3-nginx와-환경-변수-설정하기)

[4. 방화벽 설정하기](#4-방화벽-설정하기)

&nbsp;

---

&nbsp;

## 1. 구글 클라우드 가입하고 가상 머신 생성하기

먼저 구글 계정으로 [구글 클라우드](https://cloud.google.com/?hl=ko){:target="_blank"}에 가입 합니다. 가입 후 **콘솔** 화면으로 이동하면, (신규 가입의 경우 **90일동안 300달러의 무료 크레딧**을 사용할 수 있습니다)

가입을 완료하면 디폴트 프로젝트가 자동으로 생성됩니다. 우측 상단의 메뉴 버튼을 눌러 **Compute engine > 가상 머신 > VM 인스턴스** 메뉴를 클릭하고, **인스턴스 만들기 버튼**을 클릭합니다.

&nbsp;

![create vm](/blog/assets/posts/asset-deploy-on-gcp-screenshot00.png)

&nbsp;

이 때 결제 계정을 등록하라고 하는데, 비용이 결제될 신용 카드 정보를 등록하고 이후 절차를 진행하면 됩니다. VM 인스턴스 설정 옵션에 따라 과금 내용이 달라지게 되므로, 여기서는 무료의 가까운 비용으로 사용할 수 있는 옵션을 선택합니다. 

1. **리전**과 **영역**은 다음 [링크](https://cloud.google.com/free/docs/free-cloud-features?hl=ko#compute){:target="_blank"}를 참조하여 선택 합니다. (저는 `us-central1(아이오와)` - `us-central1-a`를 선택)
2. **머신 구성**은 `E2`를 선택하고, **머신 유형**은 `e2-micro` 옵션을 선택해줍니다.


&nbsp;

![instance option](/blog/assets/posts/asset-deploy-on-gcp-screenshot01.png)

&nbsp;

부팅 디스크의 경우 익숙한 OS로 선택을 하시면 되는데, 저는 `Ubuntu`와 `20.04 LTS`버전을 선택했습니다. 디스크 타입은 `표준 영구 디스크`로, 용량은 `30GB`로 잡아주고 **변경**을 클릭합니다.

마지막으로 방화벽 설정은 `http` 와 `https` 옵션이 있는데, 저는 `http`만 사용할 예정이기 때문에 `http 트래픽 허용` 옵션 만 선택했습니다. 설정을 완료했다면 화면 하단의 **만들기** 버튼을 클릭하여 가상 머신 인스턴스를 생성합니다. (완료되기 까지 시간이 조금 걸립니다)


&nbsp;

## 2. 고정 IP 설정하고 터미널에서 VM 접근하기

VM 생성이 완료되었다면, **Compute engine > 가상 머신** 화면 진입 시 생성한 VM 정보가 표시됩니다. 다음은 VM에 접근하기 위한 외부 고정 IP 주소를 설정합니다. 고정 IP가 설정되어 있지 않다면 인스턴스 재부팅 시나 기타 오류로 인해 외부 IP가 바뀌어 버릴 수 있으므로, **네트워킹 섹션의 VPC 네트워크 > IP 주소** 메뉴로 진입하여 **외부 고정 IP 주소 예약** 버튼을 클릭합니다.

&nbsp;

![ip option](/blog/assets/posts/asset-deploy-on-gcp-screenshot02.png)

&nbsp;

설정 화면에서 사용할 **이름**을 입력하고, **버전**을 `IPv4`로 설정한 다음 **예약** 버튼을 클릭해줍니다.

> 참고 : 고정 IP 사용에 대한 비용이 발생할 수는 있으나, 결제 보고서를 보면 비용 할인이 들어가는 걸 확인했습니다.

이제 터미널에서 VM에 접근하여 git 과 프로젝트 설정을 해주어야 합니다. VM에 접근하는 방법과 git 프로젝트 설정에 대해서는 이전에 작성한 내용을 참고했습니다.

> 이전 글 - VM(가상머신) 환경에서 프로젝트 배포하기 : [링크](https://ji-hoon.github.io/blog/project-release-on-vm){:target="_blank"}

&nbsp;

## 3. NginX와 환경 변수 설정하기

이전 글에서 NginX를 설정했을 때는 아래처럼 `proxy_pass` 값을 수정하는 작업만 진행했었는데 이번 프로젝트의 경우에는 클라이언트와 백엔드 프로젝트가 같이 있는 레포지토리다보니, NginX 설정이 좀 더 필요했습니다.

```shell
(기존) try_files $uri $uri/ =404; 
(변경) proxy_pass http://127.0.0.1:3000; 
```

먼저 ```sudo nano /etc/nginx/sites-available/default``` 명령어로 nginx 설정 파일을 엽니다.

```shell
root /home/{계정이름}/home_accountant/Home-accountant/server/dist;

# Add index.php to the list if you are using PHP
index index.html index.htm index.nginx-debian.html;

server_name _;

location / {
    # First attempt to serve request as file, then
    # as directory, then fall back to displaying a 404.
    try_files $uri $uri/ /index.html;
    proxy_pass http://127.0.0.1:5001;

}

location /api {
    proxy_pass http://127.0.0.1:5001;
}
```

1. 먼저 클라이언트 프로젝트를 빌드한 파일을 serve 할 서버 프로젝트의 빌드 폴더 경로를 root 디렉토리로 잡아줍니다. (여기서는 dist)
2. location / 경로로 접근했을 때, 모든 route가 index.html로 접근할 수 있도록 try_files 경로를 수정해줍니다.
3. proxy_pass 값은 이전과 동일하게 수정해줍니다. (포트만 5001로 변경, 이 포트 번호는 방화벽 설정 시 필요하니 잘 기억해둡니다.)
4. 마지막으로 index.html에서 api로의 접근을 허용해주어야 하므로, /api 경로도 proxy_pass 값에 추가해줍니다.

그 다음은 환경 변수를 설정합니다. 클라이언트와 백엔드 프로젝트를 빌드하기 전에 아래처럼 환경 변수를 설정해줍니다.

> 클라이언트 환경 변수

```shell
(로컬)
VITE_BACKEND_URL="http://localhost:5001"
VITE_FRONTEND_URL="http://localhost:5173"

(VM)
VITE_BACKEND_URL="http://35.231.16.39"
VITE_FRONTEND_URL="http://35.231.16.39"
```

로컬 환경에서 개발할 때는 각각의 포트를 다르게 설정했지만 클라이언트 빌드 후 server 프로젝트의 dist 폴더로 합쳐지게 되므로, 각 URL에 동일한 주소를 할당합니다. 이 주소에는 2번 항목에서 생성했던 **VM 인스턴스의 외부 고정 IP**를 `/` **없이** 입력해줍니다. 다음은 서버 프로젝트의 환경 변수를 설정해야 합니다.

> 서버 환경 변수

```shell
(로컬)
PORT=5001
IP=localhost
FRONTEND_URL="http://localhost:5173"
BACKEND_URL="http://localhost:5001"

(VM)
PORT=5001
IP=127.0.0.1
FRONTEND_URL="http://35.231.16.39"
BACKEND_URL="http://35.231.16.39"
```

포트와 IP는 NginX 환경 설정 대로 입력해주고, URL의 경우 클라이언트 환경 변수와 동일하게 **VM 인스턴스의 외부 고정 IP**를 `/` **없이** 입력해줍니다. (IP 값에 ip 형태로 지정하지 않을 경우, `tcp`로 설정되지 않을 수 있기 때문에 `localhost`가 아닌 `127.0.0.1`로 입력해줍니다.) 

&nbsp;

`tcp`로 동작하는지 여부는 터미널에서 ```netstat -tuln``` 명령어로 확인할 수 있습니다.

```shell
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State    
...  
tcp        0      0 127.0.0.1:5001          0.0.0.0:*               LISTEN     
...
```


&nbsp;

## 4. 방화벽 설정하기

이제 마지막으로 방화벽 설정을 해야 합니다. **네트워킹 섹션의 VPC 네트워크 > 방화벽** 메뉴로 진입하여 **방화벽 규칙 만들기** 버튼을 클릭합니다.

&nbsp;

![firewall](/blog/assets/posts/asset-deploy-on-gcp-screenshot04.png)

&nbsp;

이름과 설명을 입력해줍니다. 트래픽 방향은 `인그레스(수신)`으로, 소스 필터는 `ipv4`, 범위는 `0.0.0.0/0` 으로 입력하여 모든 ip에서의 요청을 허용합니다. 마지막으로 프로토콜 및 포트 옵션은 `TCP - 5001`로 입력해준 뒤, 만들기 버튼으로 규칙을 생성합니다. (**여기서 입력한 포트 번호는 서버 프로젝트 환경 변수에서 설정한 포트 번호와 동일해야 합니다.**)

&nbsp;

이제 설정이 모두 완료 되었습니다. 터미널로 돌아가서 서버를 실행 시키면 외부 고정 IP를 통해 프로젝트에 접속할 수 있습니다.

&nbsp;

![firewall](/blog/assets/posts/asset-deploy-on-gcp-screenshot05.png)

&nbsp;


> 참고하면 좋은 링크 - GCP에 웹 서버 배포하기 : [링크](https://uzzam.dev/38?category=1026425)


> 태그 Tag : #react #typescript #gcp #vm