---
title: "[Next.js] Server Side Rendering"
last_modified_at: '2022-07-03 07:00:00 +0900'
categories:
- Next.js
tag:
- NodeJS
- Docker
- Github
---

Next.JS의 getServerSideProps와 getStaticProps

현재 서비스하고 있는 프로젝트를 기준으로 설명드리겠습니다
Next.JS를 이용한 프로젝트 입니다
- https://github.com/Archive-Discord/archive-front-v2

> getStaticProps
--
`처음 빌드시 한번만 실행됨`  
호출 시 마다 매번 data를 불러오지 않아 성능향상에 도움이 됩니다
```js
export const getStaticProps: GetStaticProps = async() => {
  let server = (await fetch(`${process.env.API_DOMAIN}/servers`).then(res => res.json())) as any;
  let bots = (await fetch(`${process.env.API_DOMAIN}/bots`).then(res => res.json())) as any;
  return {
    props: {
      server: server.data,
      bot: bots.data
    },
    revalidate: 600 // revalidate를 줄경우 빌드이후에도 정해진시간마다 재생성
  };
};
```

> getServerSideProps (Server Side Rendering)
--
`페이지 요청시 마다 실행됨`   
호출 시 마다 매번 data를불러와 서버에서 랜더링해  
SEO(검색 엔진 최적화) 최적화에 도움이 됩니다
```js
export const getServerSideProps: GetServerSideProps<ServerProps> = async context => {
  const server = (await fetch(`${process.env.API_DOMAIN}/servers/${encodeURI(context.params.id as string)}`)
    .then(res => res.json())) as any;
  if (server.status !== 200) {
    return {
      props: {
        server: null,
        error: true,
        message: server.message
      },
    };
  }
  return {
    props: {
      server: server.data,
      error: false,
      message: server.message
    },
  };
};
```

> 결과
![]({{ 'assets/images/post/nextjs/serversiderenderpreview.png' | relative_url }})
