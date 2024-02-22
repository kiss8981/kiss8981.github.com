---
title: "[iOS, Apple] 앱 심사하면서 있었던 일들"
last_modified_at: "2024-02-22 07:00:00 +0900"
categories:
  - iOS
tag:
  - Apple
  - iOS
---

### 애플 리젝만 20번넘게 받고 깨달은 것들

![]({{ 'assets/images/post/2024/apple-reject-list/rejectlist.png' | relative_url }}){: width="400"}

#### Guideline 1.2 - Safety - User Generated Content

가장 애를 먹었던 부분인데 토이 프로젝트로 커뮤니티 기능이 있는 앱을 개발했다.
여기에서 처음에는 단순 글 삭제 기능만 넣었지만 `Guideline 1.2 - Safety - User Generated Content` 사유로 항상 리젝 당했다.

- 유저가 사용자를 신고하고 해당 게시물을 보지 않도록 하는 기능을 추가하자
- 유저가 신고한 게시물이 전달되는 어드민 페이지를 캡처하여 회신해 주자

#### Guideline 2.1 - Information Needed

애플 측에서 앱을 심사할 때 최소한의 콘텐츠가 필요하다.

- 커뮤니티 기능의 경우 글 3개정도를 추가하고 심사를 요청하자

#### Guideline 2.1 - Information Needed - 2

`Your app uses the AppTrackingTransparency framework, but we are unable to locate the App Tracking Transparency permission request when reviewed on iOS 17.3.1.`
앱 추적 권한을 사용할 때 사용 알림이 뜨지 않을 때 리젝 되는 사유이다.

개발환경에서는 잘 작동하던 `AppTrackingTransparency` 권한이 배포시에는 뜨지 않는 오류가 있다.
찾아보니 왜 인지는 모르겠으나 권한 요청이 늦게뜨는 경우가 있다고 한다.

- React Native의 경우는 Splash Screen 이 사라진 후에 조금 있다가 요청하는 식으로 바꾸자.
- Swift의 경우 찾아보니 `asyncAfter` 를 사용해서 요청하면 된다는 것을 찾았다.

#### 나머지 유용한 사항들

- 심사 시 리젝 된 경우 앱 심사를 다시 제출하는 것이나 메시지를 남기는 것이나 거의 동일하게 재 심사가 시작된다.
- 애플 빠른 심사 요청의 경우는 제출당 한번만 해주면 계속 유지된다.
- 애플 심사 소요시간은 빠르면 심사 요청하고 1시간도 안되어 심사를 시작하는 경우도 있고, 길면 하루가 지나서 시작되는 경우가 있다.
- 애플 신규 앱 심사부터 배포까지 2024년 2월 기준으로 심사 요청 후 2일 만에 배포까지 완료된 경우도 있다.
