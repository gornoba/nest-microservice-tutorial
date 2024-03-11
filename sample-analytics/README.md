# 역할

- Rest API와 Microservice 역할을 모두 하는 hybrid 형태

# 구현

- main에 Microservice를 추가

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { Transport } from '@nestjs/microservices';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.connectMicroservice({
    transport: Transport.TCP,
    options: {
      port: 3001,
    },
  });
  await app.startAllMicroservices();
  await app.listen(3001);
}
bootstrap();
```

- controller에 역할에 맞는 Microservice pattern을 사용

```typescript
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';
import { EventPattern, MessagePattern } from '@nestjs/microservices';
import { CreateUserEvent } from './create-user.event';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }

  @EventPattern('user_created')
  handleUserCreated(data: CreateUserEvent) {
    this.appService.handleUserCreated(data);
  }

  @MessagePattern({ cmd: 'get_analytics' })
  getAnalytics() {
    return this.appService.getAnalytics();
  }
}
```

# 이해

- EventPattern은 응답을 기다리지 않는 fire-and-forget 방식으로 독립적으로 작업
- MassagePattern은 request-response 방식으로 응답을 기대할 수 있으며 비동기를 지원
