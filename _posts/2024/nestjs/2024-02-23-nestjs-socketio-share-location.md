---
title: "[Nestjs + SocketIO] 실시간으로 위치 정보를 공유하는 서비스 만들기 (1)"
last_modified_at: '2024-02-23 08:20:00 +0900'
tag:
- socketio
- nestjs
- redis
categories:
- NestJS
---

### 실시간으로 위치 정보를 지도에 표시하는 앱 개발기

![]({{ 'assets/images/post/2024/nestjs-socketio-share-location/preview.gif' | relative_url }}){: width="500"}


##### 프로젝트 만들기
NestJS를 사용할 예정이므로 기본적으로 프로젝트를 만든다
```shell
npm i -g @nestjs/cli
nest new project-name
```

##### 필요 패키지 설치하기
socketio 관련 패키지
```shell
yarn add @nestjs/platform-socket.io @nestjs/websockets
```

DB 관련 패키지
```shell
yarn add prisma @prisma/client redis ioredis @liaoliaots/nestjs-redis
```


##### Database Service 파일 만들기
Prisma Service 파일 생성
```shell
nest g s prisma
```

```ts
// Path: src/prisma/prisma.service.ts
import { Injectable, OnModuleDestroy, OnModuleInit } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService
  extends PrismaClient
  implements OnModuleInit, OnModuleDestroy
{
  async onModuleInit() {
    await this.$connect();
  }

  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

RedisModule Import
```ts
// Path: app.module.ts

@Module({
  imports: [
    ConfigModule.forRoot(),
    RedisModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: async (configService: ConfigService) => ({
        config: {
          host: configService.get('REDIS_HOST'),
          port: configService.get('REDIS_PORT'),
          password: configService.get('REDIS_PASSWORD'),
        },
      }),
      inject: [ConfigService],
    })
  ]
})
```

##### Bus Gateway 파일 만들기 (위치공유 기능)
```shell
nest g mo bus
nest g ga bus
nest g s bus
```

```ts
// Path: src/bus/bus.gateway.ts

@WebSocketGateway(3001, {
  namespace: 'bus',
  cors: { origin: ['*:*'] },
})
export class BusGateway
  implements OnGatewayInit, OnGatewayConnection, OnGatewayDisconnect
{
  constructor(private readonly busService: BusService) {}
}
```

```ts
// Path: src/bus/bus.service.ts

@Injectable()
export class BusService {
  constructor(
    @InjectRedis() private readonly client: Redis,
    private readonly prisma: PrismaService,
    private readonly authService: AuthService,
  ) {}
}
```


##### SocketIO SubscribeMessage 추가
nestjs에서 socketio gateway를 사용하려면 클라이언트에서 보내는 이벤트를 `@SubscribeMessage`를 통하여 구독할 수 있다.

###### 인증 로직 (Authenticate)
`@MessageBody`에는 `socket.emit`을 통하여 보낸 데이터가 들어가있다.
`@ConnectedSocket`에는 연결중인 소캣의 정보가 들어가있다.

`@MessageBody`를 통하여 로그인 관련 정보를 받아 서버에서 소켓 연결에 대한 검증 비즈니스 로직을 처리한다

```ts
@SubscribeMessage('authenticate')
async handleAuthenticateEvent(
  @MessageBody() data: BusAuthenticateDto,
  @ConnectedSocket() client: Socket,
): Promise<void> {
  try {
    const isAthenticate = await this.busService.authenticate(
      client,
      data.provider,
      data.token,
    );
    createSocketResponse(client, 'authenticate', isAthenticate);
  } catch (error: any) {
    this.logger.error(error);
    createSocketResponse(client, 'authenticate', false, error.message);
  }
}
```

서버에서 client의 authenticate를 통하여 전달된 토큰의 검증을 하고 Redis 서버에 해당 클라이언트의 인증 정보를 저장한다.
client의 room을 해당 정보를 제공해야 하는 provider에 입장 시킨다

```ts
// Path: src/bus/bus.service.ts
 async authenticate(
   client: Socket,
   providerId: string,
   token: string,
 ): Promise<boolean> {
   const provider = await this.prisma.provider.findUnique({
     where: { id: providerId },
   });

   if (!provider) throw new Error('찾을 수 없는 클라이언트입니다.');

   try {
     const checkToken = await this.authService.authenticate({
       token: token,
       tokenType: 'access',
       provider: providerId,
     });

     if (!checkToken) throw new Error('인증에 실패했습니다.');

     client.join(providerId);

     await this.client.set(`bus:${client.id}`, providerId, 'EX', 60 * 60 * 3);

      return true;
   } catch (error) {
      throw new Error('인증에 실패했습니다.');
   }
 }
```

###### 위치 이동 로직  (Locationupdate)
`@MessageBody`를 통해 들어오는 운행중인 버스의 `busId`, `location`정보를 이용하여
Redis에 저장된 클라이언트의 인증여부를 확인하고 해당 클라이언트가 소속된 provider에  `busId`, `location`정보를 전달한다.

버스 위치를 보는 입장에서 처음 접속하였을 때 마지막 버스 위치와 운행 여부를 확인하기 위해 버스의 마지막 위치와 아이디를 저장한다.

```ts
  @SubscribeMessage('locationupdate')
  async handleLocationUpdateEvent(
    @MessageBody() data: BusLocationUpdateDto,
    @ConnectedSocket() client: Socket,
  ): Promise<void> {
    try {
      await this.busService.locationUpdate(this.server, client.id, data);
    } catch (error: any) {
      this.logger.error(error);
      createSocketResponse(client, 'locationupdate', null, error.message);
    }
  }
```

```ts
// Path: src/bus/bus.service.ts
async locationUpdate(
    server: Server,
    clientId: string,
    data: BusLocationUpdateDto,
  ): Promise<void> {
    const providerId = await this.client.get(`bus:${clientId}`);
    if (!providerId) throw new Error('인증되지 않은 클라이언트입니다.');
    await this.client.set(
      `bus:${providerId}:${data.busId}:lastlocation`,
      JSON.stringify(data),
      'EX',
      60 * 60 * 3,
    );
    server.to(providerId).emit('locationupdate', data);
  }
```

##### 마무리
벡엔드에서 필요로 하는 대표적인 기본 비즈니스 로직만 넣은 거라 상세한 정보는 [깃허브](https://github.com/kiss8981/busapp-backend)에서 확인하도록 하자
다음 편에서는 실시간으로 위치 정보를 전달하는 클라이언트를 만들어보자
