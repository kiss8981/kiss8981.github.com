---
title:  "[React] 리액트 useEffect() 사용방법"
search: true
categories: 
  - React
last_modified_at: 2021-05-31T20:19:00+09:00
tag:
 - React
 - NodeJS
---

리액트에서 useEffect()를 사용하는 자세한방법


**첫 번째:** 변동 사항이 있을 때마다 실행하는 방식
{: .notice--info}

```
import React, { useEffect } from 'react';

function main() {
  useEffect(() => {
    console.log('실행!')
  });
}

```

**두 번째:** 페이지가 로딩될 때만 실행
{: .notice--info}

```
import React, { useEffect } from 'react';

function main() {
  useEffect(() => {
    console.log('실행!')
  }, []);
}

```


**세 번째:** count 변수에 변동이 생길 시 수정
{: .notice--info}

```
import React, { useState, useEffect } from 'react';

function main() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log(`실행! ${count}번째`)
  }, [count]);

  return (
    <div>
      <p>실행! {count}번째</p>
      <button onClick={() => setCount(count + 1)}>
        Button
      </button>
    </div>
  );
}

```