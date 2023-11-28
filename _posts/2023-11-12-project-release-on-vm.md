---
title: VM(가상머신) 환경에서 프로젝트 배포하기 
updated: 2023-11-12 12:00
---

> Elice SW Track 7기 - 9주차 진행 내용입니다.


![Elice Banner](/blog/assets/elice/SW7_top_banner.png)

&nbsp;

VM(Virtual Machine)이란, 물리적 하드웨어 시스템(오프프레미스 또는 온프레미스에 위치)에 구축되어 자체 CPU, 메모리, 네트워크 인터페이스 및 스토리지를 갖추고 가상 컴퓨터 시스템으로 작동하는 가상 환경입니다.

> 참고 - RedHat 홈페이지 : [링크](https://www.redhat.com/ko/topics/virtualization/what-is-a-virtual-machine){:target="_blank"}

 이번에는 GCP (Google Cloud Platform)에서 제공하는 VM 환경에서 프로젝트를 배포해보았습니다.

&nbsp;

## 목차
[1. VM 접속하기](#1-vm-접속하기)

[2. node 설치 및 환경 변수 설정하기](#2-node-설치-및-환경-변수-설정하기)

[3. pm2와 nginx 설치하기](#3-pm2와-nginx-설치하기)

[4. React 프로젝트 빌드하여 배포하기](#4-react-프로젝트-빌드하여-배포하기)


&nbsp;

---

&nbsp;
## 1. VM 접속하기

VM에는 터미널에서 **SSH** 프로토콜을 통해 접속합니다. **SSH (Secure Shell)**은 원격 컴퓨터와 네트워크 사이의 안전한 통신을 제공하는 프로토콜로, 주로 원격으로 다른 컴퓨터에 로그인하고 파일을 전송하거나 명령을 실행하는 데 사용됩니다.

```shell
$ ssh {접속 주소}
```

접속 주소를 통해 ssh 접속을 시도하면, 비밀번호를 입력하면 vm에 접속할 수 있습니다.

&nbsp;
## 2. node 설치 및 환경 변수 설정하기

VM에 접속하면, 프로젝트 설치와 배포를 위한 도구들을 설치해야 합니다. 먼저 아래 명령어를 통해 업데이트를 진행합니다.

```shell
$ sudo apt update -y && sudo apt upgrade -y
```
설치가 종료되면 nvm을 설치합니다.

```shell
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.37.2/install.sh | bash
$ source ~/.bashrc
$ nvm install 16.14.0
$ nvm use 16.14.0
```
Node까지 설치가 완료되었다면, 프로젝트 리포지토리를 클론합니다. VM의 ssh 키를 발급받아서 Gitlab or Github 계정에 등록하는 과정이 필요한데, git의 user.name과 user.email을 본인의 정보로 등록 후 VM에서 발급한 ssh키를 사용중인 Gitlab or Github 계정에 등록해서 사용합니다.
```shell
$ git clone {프로젝트 리포지토리 주소}
```
클론이 완료 되었다면, 이제 프로젝트 폴더에 node module을 설치합니다. (cd 명령어를 사용해서 프로젝트 폴더로 이동 후 설치)
```shell
$ cd elice-motors-server
/elice-motors-server$ npm install
```
패키지 설치가 완료되었다면 마지막으로 환경 변수를 생성하여 필요한 변수들을 작성합니다. 내용 입력 후에는 :wq를 눌러 저장 후 exit를 수행합니다.
```shell
/elice-motors-server$ vi .env
```
서버를 돌릴 준비가 되었습니다. 아래 명령어를 시작하면 외부에서도 VM의 주소를 통해 접근이 가능합니다.(port는 3000번) 단, npm 명령어로는 VM에 접속한 상태에서만 서버가 정상적으로 동작하게 됩니다.
```shell
/elice-motors-server$ npm start 
```

&nbsp;
## 3. pm2와 nginx 설치하기

VM에 접속한 상태가 아니더라도 서버를 계속해서 운용하려면, pm2 모듈을 설치해야 합니다. 다음 명령어를 입력하여 설치합니다.

```shell
/elice-motors-server$ npm install --global pm2
```
이제 아래 명령어를 입력하면, 서버에 접속해있지 않은 상태라도 서버가 돌아가게 됩니다. (반드시 프로젝트 폴더 위치에서 명령어를 입력해야 합니다.)
```shell
/elice-motors-server$ pm2 --name express start npm -- run start 
```
돌아가는 서버의 리스트를 보려면 `pm2 list`를, 0번 서버를 종료하려면 `pm2 delete 0`를 입력합니다.
```shell
/elice-motors-server$ pm2 list
/elice-motors-server$ pm2 delete 0
```
다음은 nginx를 설치하는 단계만 남았습니다. 역시 프로젝트 폴더 내에 설치합니다.
```shell
/elice-motors-server$ sudo apt install nginx 
```
설치가 완료되면, 설정 파일을 열어 파일을 수정해야 합니다.
```shell
/elice-motors-server$ sudo nano /etc/nginx/sites-available/default
```
파일 내부에서 아래 문구를 찾아 수정합니다. (포트 번호는 3000) 수정 후 저장하려면 (컨트롤 x -> 컨트롤 y -> 엔터를 입력합니다.)
```shell
(기존) try_files $uri $uri/ =404; 
(변경) proxy_pass http://127.0.0.1:3000; 
```
설정 파일에 문제가 없는지 다음 명령어를 통해 검증하고, 문제가 없다면 nginx를 재실행하여 설정이 정상적으로 반영되도록 합니다. (2번째 명령어가 동작하지 않는다면 3번째 명령어를 입력합니다.)
```shell
/elice-motors-server$ sudo nginx -t 
/elice-motors-server$ sudo systemctl reload nginx
/elice-motors-server$ sudo service nginx restart
```
이제 모든 설정이 완료되었습니다! 아래 명령어를 수행하면 포트 번호를 입력하지 않아도 접속이 가능하게 됩니다.
```shell
/elice-motors-server$ pm2 --name express start npm -- run start 
```

&nbsp;
## 4. React 프로젝트 빌드하여 배포하기

방금 배포한 프로젝트 리포지토리에는 서버단 소스 코드만 포함되어 있었습니다. React로 작성된 프로젝트를 같은 URL에 배포하려면, 먼저 React 프로젝트 폴더에서 아래 명령어를 실행하여 React 프로젝트의 배포 파일을 생성합니다.

```shell
/elice-motors-front$ npm run build 
```
배포 파일 생성이 완료되면, 프론트엔드 프로젝트 폴더 하위에 `build` 폴더가 신규 생성되는 것을 확인할 수 있습니다. 이 폴더 하위의 모든 파일을 복사하여 서버 프로젝트 폴더 하위에도 동일한 `build` 폴더를 생성하여 붙여넣습니다.

&nbsp;

다시 서버 프로젝트 폴더로 돌아가서, 빌드 파일을 포함한 git commit을 생성하고 push 합니다. 
&nbsp;

push가 완료되면, VM으로 돌아가서 동일한 브랜치의 최신 커밋을 받아옵니다. 아래 명령어를 통해 서버를 다시 실행하면, React 프로젝트의 빌드 파일이 적용된 화면을 확인하실 수 있습니다.

```shell
/elice-motors-server$ pm2 delete 0
/elice-motors-server$ pm2 --name express start npm -- run start 
```




&nbsp;

---
&nbsp;

[![Elice UTM Banner](/blog/assets/elice/SW7_jihoonkim_bottom_banner.png)](https://elice.training/track/sw?utm_source=sw7&utm_medium=blog&utm_campaign=challenge&utm_content=m2gzitm8b)
&nbsp;
> 태그 Tag : #엘리스트랙 #엘리스트랙후기 #리액트네이티브강좌 #온라인코딩부트캠프 #온라인코딩학원 #프론트엔드학원 #개발자국비지원 #개발자부트캠프 #국비지원부트캠프 #프론트엔드국비지원 #React #Styledcomponent #React Router Dom #Redux #Typescript #Javascript