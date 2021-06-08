---
title:  "[React] 리액트 구글 로그인"
search: true
categories: 
  - React
last_modified_at: 2021-06-08T13:01:57+09:00
tag:
 - React
 - NodeJS
 - Google
---

react-google-login 기능을 이용하여 구글 로그인 기능 만들기


react-google-login을 설치합니다
```
npm install react-google-login
```


로그인성공시 responseGoogle로 응답을 전달합니다
```js
import { GoogleLogin } from 'react-google-login';

const responseGoogle = (response) => {
  console.log(response);
}

const Login = () => {
    return (
        <>
            <GoogleLogin
            clientId='구글 OAuth 클라이언트 ID'
            buttonText="로그인"
            onSuccess={responseGoogle}
            onFailure={responseGoogle}
            cookiePolicy={'single_host_origin'}
            />
        </>
    )
}

export default Login;
```

로그인성공시 localStorage에 유저 ID, 유저 Email, 유저 Name 을추가합니다

![]({{ 'assets/images/post/react/localStorage.PNG' | relative_url }}){: .align-center}

```js
import { GoogleLogin } from 'react-google-login';

const responseGoogle = (response) => {
  window.localStorage.setItem("user_id", response.googleId);
  window.localStorage.setItem("user_email", response.Ft.pu);
  window.localStorage.setItem("user_name", response.Ft.Ue);
}

const Login = () => {
    return (
        <>
            <GoogleLogin
            clientId='구글 OAuth 클라이언트 ID'
            buttonText="로그인"
            onSuccess={responseGoogle}
            onFailure={responseGoogle}
            cookiePolicy={'single_host_origin'}
            />
        </>
    )
}

export default Login;
```


로그인성공시 localStorage에 유저 ID, 유저 Email, 유저 Name 을추가하고<br>
로그아웃시 localStorage에있는 유저 ID, 유저 Email, 유저 Name 을삭제합니다

```js
import React from 'react'
import { GoogleLogin, GoogleLogout } from 'react-google-login'

const responseGoogle = (response) => {
    window.localStorage.setItem("user_id", response.googleId);
    window.localStorage.setItem("user_email", response.Ft.pu);
    window.localStorage.setItem("user_name", response.Ft.Ue);
  }

const logout = () => {
    window.localStorage.removeItem("user_id");
    window.localStorage.removeItem("user_email");
    window.localStorage.removeItem("user_name");
  }

const Login = () => {
    return (
        <>
          <GoogleLogin
          clientId='구글 OAuth 클라이언트 ID'
          buttonText="로그인"
          onSuccess={responseGoogle}
          onFailure={responseGoogle}
          cookiePolicy={'single_host_origin'}
          />
          <GoogleLogout
          clientId="구글 OAuth 클라이언트 ID"
          buttonText="로그아웃"
          onLogoutSuccess={logout}
          />
        </>
    )
}

export default Login;
```

`localStorage.getItem("user_id")`향목이 null 일 경우 로그인을 표시하고<br>
아닐경우 로그아웃 버튼을 표시합니다

```js
{localStorage.getItem("user_id") === null ? (
  <GoogleLogin
    clientId='구글 OAuth 클라이언트 ID'
    buttonText="로그인"
    onSuccess={responseGoogle}
    onFailure={responseGoogle}
    cookiePolicy={'single_host_origin'}
  />
) : (
  <GoogleLogout
    clientId="구글 OAuth 클라이언트 ID"
    buttonText="로그아웃"
    onLogoutSuccess={logout}
  />
)}
```

