---
title:  "[AWS] PM2 80번 포트에서 실행하기"
search: true
categories: 
  - AWS
last_modified_at: 2021-05-28T08:20:00+09:00

tag:
 - AWS
 - PM2
 - NodeJS
 - Linux
---

AWS Nodejs Error: listen EACCES 0.0.0.0:80 에러 해결하기

**루드 폴더에서 실행:** 이 방식은 루트 폴더에서 실행하여 해결하는 방식입니다.
{: .notice--info}

* root 비밀번호 설정

```
sudo passwd root
```

* root 계정으로 로그인

```
su –
```

* 루트 폴에더에서 git clone

```
git clone git링크
```

* pm2 시작 SPA(single-page application)서비스 기준 (빌드된 파일)

```
pm2 serve [폴더명] 80 --spa --name "이름"
```