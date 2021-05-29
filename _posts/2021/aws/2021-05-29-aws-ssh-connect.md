---
title: "[AWS] 윈도우에서 AWS 접속하기"
last_modified_at: '2021-05-29 21:57:32 +0900'
categories:
- AWS
tag:
- AWS
- Linux
---

윈도우에서 SSH방식으로 AWS EC2 서버에 접속하는 방법

윈도우에서 CMD창을 열어줍니다
![]({{ 'assets/images/post/aws/cmd.PNG' | relative_url }})


CMD창에서 접속을 해봅시다!

```
ssh ec2-user@(퍼블릭 IPv4 주소) -i (AWS 인스턴스 생성시 사용된 키 페어 경로)
```

![]({{ 'assets/images/post/aws/cmd2.PNG' | relative_url }})

**예시:**  ssh ec2-user@127.0.0.1 -i C:\Users\Administrator\ec2.pem
{: .notice--info}

접속정공!

![]({{ 'assets/images/post/aws/cmd3.PNG' | relative_url }})
