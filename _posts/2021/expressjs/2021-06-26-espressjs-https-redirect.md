---
title:  "[ExpressJS] ExpressJS http없이 접속 https로 리다이렉트(Redirect) 처리"
search: true
categories: 
  - ExpressJS
last_modified_at: 2021-06-26T19:53:00+09:00
tag:
 - ExpressJS
 - NodeJS
---

ExpressJS에서 http없이 접속할 경우 https가 있는 페이지로 리다이렉트

http는 80번 포트이므로 80번 포트로 접속하는 모든 유저를 https로 리다이렉트 합니다

```js
const express = require('express');
const app = express();
const server = http.createServer(app);

app.get("*", (req, res) => {
    let to = "https://" +  req.headers.host + req.url;
    res.redirect(to)
})

const listener = server.listen(80, function() {
    console.log("app is listening on port " + listener.address().port);})
```

