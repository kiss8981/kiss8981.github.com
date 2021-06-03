---
title:  "[React] useState() 데이터가 안 나오는 경우"
search: true
categories: 
  - React
last_modified_at: 2021-06-03T22:42:00+09:00
tag:
 - React
 - NodeJS
---

useState() 를 사용하여 API 데이터를 저장하려고 했지만 console.log에서는 API 데이터가 나오는데 State에 저장된 값은 안 나오는 경우

```
const [confirmData, setConfirmData] = useState();

useEffect(() => {
 const data = axios.get('http://127.0.0.1:8080/getdata');
 console.log(data);
 setConfirmData(data);
 console.log(confirmData)
}, []);
```
	
위와 같이 실행했을 때에는 불러온 데이터가 `console.log(data);`는 출력되지만 `console.log(confirmData)`는 출력되지 않는다
자세한 사항은 [stackoverflow 답변](https://stackoverflow.com/a/58877875/15785590)을 참고해보자

이것을 해결하는 방법은 아래와 같이 처음 접속했을 때 `useEffect`를 실행하고
`confirmData`에 변동이 생길 경우 변수를 실행하는 방식으로 하면 된다
이것에 대한 자세한 사용법은 [여기](https://kiss8981.github.io/react/react-useeffect-useage)를 읽어보자

```
const [confirmData, setConfirmData] = useState();

useEffect(() => {
 const data = axios.get('http://127.0.0.1:8080/getdata');
 console.log(data);
 setConfirmData(data);
}, [])

useEffect(() => { 
 console.log(confirmData);
}, [confirmData])
```