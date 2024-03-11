# 역할

- Microservice로 이벤트를 수신하는 역할

# 구현

- main에 Microservice를 추가

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.TCP,
    },
  );
  app.listen();
}
bootstrap();
```

- contorller에 사용용도에 맞는 pattern 추가

```typescript
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';
import { EventPattern } from '@nestjs/microservices';
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
}
```

# 이해

- 해당 프로젝트는 emit 받은 EventPattern을 처리
- 생성된 user의 email로 관련 메일을 보내는 로직을 추가할 수 있음
