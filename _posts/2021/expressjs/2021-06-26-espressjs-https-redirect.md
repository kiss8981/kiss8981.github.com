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


```js
var express = require("express");
var app = express.createServer();

app.all(/.*/, function(req, res, next) {
  var host = req.header("host");
  if (host.match(/^www\..*/i)) {
    next();
  } else {
    res.redirect(301, "http://www." + host + req.url);
  }
});
```

