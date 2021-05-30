---
title:  "[NodeJS] PM2 리액트 빌드파일 실행하기"
search: true
categories: 
  - NodeJS
last_modified_at: 2021-05-30T22:10:00+09:00
tag:
 - PM2
 - NodeJS
---

Node JS PM2에서 리액트 빌드파일(single-page application) 실행하기


* PM2 설치

-g 옵션을 필수로 넣어주셔야 합니다

```
npm install pm2 -g
```

* PM2 SPA(single-page application) 서비스 실행

```
pm2 serve [폴더명] --spa --name "PM2에 표시될 이름"
```

* 다른 포트에서 실행을 원할 경우

예) 8080 포트에서 실행할경우
```
pm2 serve [폴더명] 8080 --spa --name "PM2에 표시될 이름"
```

* pm2 시작 SPA서비스 기준 (빌드된 파일)

```
pm2 serve [폴더명] 80 --spa --name "이름"
```