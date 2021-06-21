---
title: "[React] 리액트에 채널톡(channel.io) 설치하기"
last_modified_at: '2021-06-21 07:32:00 +0900'
categories:
- React
tag:
- NodeJS
- React
- 채널톡
---

리액트에서 채널톡 추가하기


```js
class ChannelService {
    constructor() {
      this.loadScript();
    }
  
    loadScript() {
      var w = window;
      if (w.ChannelIO) {
        return (window.console.error || window.console.log || function(){})('ChannelIO script included twice.');
      }
      var ch = function() {
        ch.c(arguments);
      };
      ch.q = [];
      ch.c = function(args) {
        ch.q.push(args);
      };
      w.ChannelIO = ch;
      function l() {
        if (w.ChannelIOInitialized) {
          return;
        }
        w.ChannelIOInitialized = true;
        var s = document.createElement('script');
        s.type = 'text/javascript';
        s.async = true;
        s.src = 'https://cdn.channel.io/plugin/ch-plugin-web.js';
        s.charset = 'UTF-8';
        var x = document.getElementsByTagName('script')[0];
        x.parentNode.insertBefore(s, x);
      }
      if (document.readyState === 'complete') {
        l();
      } else if (window.attachEvent) {
        window.attachEvent('onload', l);
      } else {
        window.addEventListener('DOMContentLoaded', l, false);
        window.addEventListener('load', l, false);
      }
    }
  
    boot(settings) {
      window.ChannelIO('boot', settings);
    }
  
    shutdown() {
      window.ChannelIO('shutdown');
    }
  }
  
  export default new ChannelService();
```

위 코드를 복사하여 컴포넌트를 하나 생성합니다


```js
ChannelService.boot({
    "pluginKey": "pluginKey", //please fill with your plugin key
    "memberId": "유저ID",
    "profile": {
      "name": "유저Name",
      "email": "유저Email", 
      "id": "유저ID"
    }
  });
```

위 코드를 라우터에 추가해 줍니다

플러그인키는 여기에서 확인이 가능합니다

![]({{ 'assets/images/post/react/channel_io.PNG' | relative_url }})
