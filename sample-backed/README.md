# 역할

- Rest API의 역할
- 마이크로서비스의 Cient의 역할

# 구현

- module의 import에 clientModule을 추가한다.

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'COMMUICATION',
        transport: Transport.TCP,
      },
      {
        name: 'ANALYTICS',
        transport: Transport.TCP,
        options: {
          port: 3001,
        },
      },
    ]),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

- Controller에서 API를 수신하고 Service에서 처리한다.

```typescript
import { Inject, Injectable } from '@nestjs/common';
import { CreateUserRequest } from './create-user-request.dto';
import { ClientProxy } from '@nestjs/microservices';
import { CreateUserEvent } from './create-user.event';

@Injectable()
export class AppService {
  private readonly users: any[] = [];

  constructor(
    @Inject('COMMUICATION') private readonly communicationClient: ClientProxy,
    @Inject('ANALYTICS') private readonly analyticsClient: ClientProxy,
  ) {}

  getHello(): string {
    return 'Hello World!';
  }

  createUser(createUserRequest: CreateUserRequest) {
    this.users.push(createUserRequest);
    this.communicationClient.emit(
      'user_created',
      new CreateUserEvent(createUserRequest.email),
    );
    this.analyticsClient.emit(
      'user_created',
      new CreateUserEvent(createUserRequest.email),
    );
  }

  getAnalytics() {
    return this.analyticsClient.send({ cmd: 'get_analytics' }, {});
  }
}
```

# 이해

- sample-backend 프로젝트는 전체 프로젝트에서 중앙 컨트롤 역할
- API를 수신하면 각 역할에 맞는 생산자에게 emit하여 역할을 분담
