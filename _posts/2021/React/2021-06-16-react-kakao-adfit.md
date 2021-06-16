---
title:  "[React] 리액트 구글 로그인"
search: true
categories: 
  - React
last_modified_at: 2021-06-16T18:49:57+09:00
tag:
 - React
 - NodeJS
 - AdFit
---

리액트에서 카카오 AdFit 광고를 넣어보기

컴포넌트에 아래 코드를 추가해 줍니다

```js
componentDidMount() {
    let ins = document.createElement('ins');
    let scr = document.createElement('script');
  
    ins.className = 'kakao_ad_area';
    ins.style = "display:none;";
    scr.async = 'true';
    scr.type = "text/javascript";
    scr.src = "//t1.daumcdn.net/kas/static/ba.min.js";
    ins.setAttribute('data-ad-width', '320');
    ins.setAttribute('data-ad-height', '100');
    ins.setAttribute('data-ad-unit', 'unit 아이디');
  
    document.querySelector('.adfit').appendChild(ins);
    document.querySelector('.adfit').appendChild(scr);
  }
```


return 안쪽에 className을 adfit으로 하여 DIV를 생성해 줍니다

```js
render() {
    return (
        <div className="adfit"/>
    );
  }
```




