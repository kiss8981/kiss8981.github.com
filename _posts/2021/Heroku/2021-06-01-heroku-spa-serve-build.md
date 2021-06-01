---
title: "[Heroku] Heroku에 배포시 SPA를 자동으로 빌드 하는 방법"
categories:
- Heroku
last_modified_at: '2021-06-01 22:25:10 +0900'
tag:
- Heroku
- NodeJS
- React
---

Heroku에서 React를 자동으로 빌드하고 배포하는 방법


**기본설정:** Procfile 생성 및 package.json 스크립트 추가
{: .notice--info}

root 폴더에 Procfile 생성 후 아래 내용 입력
```
web: serve -s build
```
![]({{ 'assets/images/post/heroku/Procfile.PNG' | relative_url }}){: .align-center}

package.json 파일에  아래 스크립트 추가
```json
"scripts": {
    "postinstall": "npm install -g serv",
    "build": "react-scripts build"
  }
```

![]({{ 'assets/images/post/heroku/Package.PNG' | relative_url }}){: .align-center}

**오류처리:** Heroku에서 빌드도중 ```code: 'MODULE_NOT_FOUND'``` 에러 발생
{: .notice--danger}

package.json 파일에 postinstall 부분에 아래 스크립트 추가

```json
"scripts": {
    "postinstall": "npm install -g serv && npm install (미설치된 모듈 이름)"
  }
```

EX: chart.js 모듈 미설치 오류 발생 시

```json
"scripts": {
    "postinstall": "npm install -g serv && npm install chart.js"
  }
```


**빌드하기:**  해로쿠에 접속하여 깃허브 연동후 파일을 불러옵니다
{: .notice--info}

![]({{ 'assets/images/post/heroku/Heroku.PNG' | relative_url }}){: .align-center}


Enable Automatic deploys 활성화 시 깃허브 커밋 시 자동으로 빌드가 진행됩니다

Deploy Branch를 누를경우 바로 빌드가 실행됩니다

![]({{ 'assets/images/post/heroku/Builded.PNG' | relative_url }}){: .align-center}

빌드성공
