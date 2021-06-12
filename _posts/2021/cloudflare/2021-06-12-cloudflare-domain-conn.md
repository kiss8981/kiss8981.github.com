---
title:  "[Cloudflare] 클라우드 플레어 도메인 연결하기"
search: true
categories: 
  - Cloudflare
last_modified_at: 2021-06-12T23:01:00+09:00
tag:
 - Cloudflare
 - DNS
---

클라우드 플레어에 도메인 연결하고 무료로 SSL 인증서까지 자동 설정하기

클라우드 플레어에 도메인 추가하기
![]({{ 'assets/images/post/cloudflare/adddomain.PNG' | relative_url }}){: .align-center}

요금제 선택하기(무료)
![]({{ 'assets/images/post/cloudflare/selfree.PNG' | relative_url }}){: .align-center}

클라우드 플레어에 나오는 NS주소를 복사하고
![]({{ 'assets/images/post/cloudflare/nameservercl.PNG' | relative_url }}){: .align-center}
DNS서버 연결하기 (hosting.kr 기준)<br> 전화번호 인증 후 네임서버를 설정합니다
![]({{ 'assets/images/post/cloudflare/setnameserver.PNG' | relative_url }}){: .align-center}



DNS 레코드를 추가해 줍니다
![]({{ 'assets/images/post/cloudflare/addrecord.PNG' | relative_url }}){: .align-center}

https를 설정을 해줍니다
![]({{ 'assets/images/post/cloudflare/httpsset.PNG' | relative_url }}){: .align-center}

**DNS 설정:** DNS 설정 완료 후 짧으면 10분에서 하루까지도 설정하는 시간이 걸립니다
{: .notice--info}

