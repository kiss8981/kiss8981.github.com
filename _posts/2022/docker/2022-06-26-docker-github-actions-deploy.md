---
title: "[Docker + Github Actions] 깃허브 액션과 도커를 이용하여 배포 자동화하기"
last_modified_at: '2022-06-26 03:00:00 +0900'
categories:
- Docker
tag:
- NodeJS
- Docker
- Github
---

Next.JS + Docker + Github Actions 자동배포

현재 서비스하고 있는 프로젝트를 기준으로 설명드리겠습니다
Next.JS를 이용한 프로젝트 입니다
- https://github.com/Archive-Discord/archive-front-v2

먼저 도커 파일을 생성합니다.  
`Dockerfile`
```Dockerfile
FROM node:16.14.2

RUN mkdir -p /app
WORKDIR /app
ADD . /app/

RUN rm yarn.lock || true
RUN rm package-lock.json || true
RUN yarn
RUN yarn build

ENV HOST 0.0.0.0
EXPOSE 3000

CMD [ "yarn", "start"]
```


`/.github/workflows/deploy-production.yml`
```yml
name: Deploy (Production)

on:
  push:
    branches: [stable] 
    # stable 브랜치에 push했을때
    # https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows

    # release가 생성되었을때
    # release:
    #   types: [created] 


env:
  DOCKER_IMAGE: ghcr.io/archive-discord/archive-front-v2 # 도커 이미지 주소
  VERSION: ${{ github.sha }}
  NAME: archive_front # 서버에서 사용할 도커 컨테이너 이름
  GITHUB_USER: archive-discord # 깃허브 이름

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up docker build
        id: buildx
        uses: docker/setup-buildx-action@v1
      - name: Cache docker layers
        uses: actions/cache@v2
        with:
          path: /tmp/.buildx-cache
          key: ${{ runner.os }}-buildx-${{ env.VERSION }}
          restore-keys: |
            ${{ runner.os }}-buildx-
      - name: Login to ghcr
        uses: docker/login-action@v1
        with:
          registry: ghcr.io
          username: ${{ env.GITHUB_USER }}
          password: ${{ secrets.GHCR_TOKEN }}
      - name: Build and push
        id: docker_build
        uses: docker/build-push-action@v2
        with:
          builder: ${{ steps.buildx.outputs.name }}
          push: true
          tags: ${{ env.DOCKER_IMAGE }}:latest
  # 배포
  deploy:
    needs: build
    name: Deploy
    runs-on: [ self-hosted, archive ]
    steps:
      - name: Login to ghcr
        uses: docker/login-action@v1
        with:
          registry: ghcr.io
          username: ${{ env.GITHUB_USER }}
          password: ${{ secrets.GHCR_TOKEN }}
      - name: Docker run
        # 기존 도커 이미지 삭제 후 새로운 도커이미지를 받아온 후 실행
        run: |
          docker stop ${{ env.NAME }} && docker rm ${{ env.NAME }} && docker rmi ${{ env.DOCKER_IMAGE }}:latest 
          docker run -d -p 3000:3000 --name ${{ env.NAME }} --restart always ${{ env.DOCKER_IMAGE }}:latest
```

깃허브에 도커 이미지를 올리기 위한 토큰 세팅
- https://github.com/settings/tokens/new
![]({{ 'assets/images/post/docker/github_token_setting.png' | relative_url }})

도커 이미지를 실행할 Actions Runner 생성
![]({{ 'assets/images/post/docker/actions_runner_setting.png' | relative_url }})

`./config.sh`실행중 label 세팅부분에서 `runs-on` 들어갈 label입력 위의경우 `archive`로 되어있습니다
종료되지 않고 백그라운드 실행
```sh
./config.sh --url https://github.com/Archive-Discord/archive-front-v2 --token **********
nohup ./run.sh &
```
리눅스의경우 실행중 `Must not run with sudo` 오류 발생시 아래와 같이 진행해주세요
```sh
RUNNER_ALLOW_RUNASROOT="1" ./config.sh --url https://github.com/Archive-Discord/archive-front-v2 --token **********
RUNNER_ALLOW_RUNASROOT="1" nohup ./run.sh &
```

완료 시 푸시 할 경우 아래와 같이 Actions 탭에 표시됩니다
![]({{ 'assets/images/post/docker/github_actions_complete.png' | relative_url }})

