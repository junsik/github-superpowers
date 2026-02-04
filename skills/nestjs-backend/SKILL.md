---
name: nestjs-backend
description: Use when working on NestJS backend with Hexagonal Architecture, UseCase pattern, and BullMQ - provides patterns for Clean Architecture API development
---

# NestJS Backend Stack (Hexagonal Architecture)

## Overview

NestJS + Hexagonal Architecture + UseCase 패턴의 best practices입니다.

**핵심 철학:** "Service는 만들지 말고, 행동은 UseCase로, 기술은 Adapter로 분리한다."

**이 스킬은 참조용입니다.** 코드 작성 시 이 패턴을 따르세요.

## 핵심 원칙

1. **Vertical Slice (Feature-first)**: 기능 단위로만 개발, 다른 feature 침범 금지
2. **UseCase First**: 모든 비즈니스 로직은 `usecases/`에만. Service 비대화 금지
3. **Hexagonal Architecture**: 외부 의존성은 Port 인터페이스로 추상화
4. **Contract-First**: DTO 먼저 정의, DTO 없는 Controller 금지

## 프로젝트 구조

```
src/
├─ features/{feature-name}/
│   ├─ api/                 # Controller + DTO
│   ├─ usecases/            # 비즈니스 로직 (1 파일 = 1 행동)
│   ├─ domain/              # 엔티티, 정책, 이벤트
│   ├─ ports/               # 인터페이스 정의
│   ├─ infra/               # 기술 구현체 (@Injectable)
│   └─ {feature}.module.ts
├─ workers/
│   └─ bullmq/              # Queue processors
└─ shared/                  # 횡단 관심사만
    ├─ auth/                # JWT 인증
    ├─ prisma/              # Prisma 클라이언트
    └─ ...
```

## 아키텍처 의존성

```
Controller → UseCase → Port (인터페이스)
                         ↑
                      Adapter (구현)
```

**의존성 규칙:**
- UseCase → Port (인터페이스만 의존)
- Adapter → Port (구현)
- ❌ UseCase → Adapter (직접 의존 금지)
- ❌ UseCase → NestJS 데코레이터 금지

## 네이밍 컨벤션

| 구분 | 패턴 | 예시 |
|------|------|------|
| UseCase | `{Action}{Entity}UseCase` | `CreateUserUseCase` |
| Port | `{Entity}RepositoryPort` | `UserRepositoryPort` |
| Adapter | `{Entity}Repository` | `UserRepository` |
| DTO | `{Action}{Entity}Dto` | `CreateUserDto` |

## UseCase 패턴

```typescript
// features/users/usecases/create-user.usecase.ts
import { UserRepositoryPort } from '../ports/user-repository.port'
import { CreateUserDto } from '../api/dto/create-user.dto'

// ❌ @Injectable() 사용 금지 - UseCase는 순수한 비즈니스 로직
export class CreateUserUseCase {
  constructor(private readonly userRepository: UserRepositoryPort) {}

  async execute(dto: CreateUserDto): Promise<User> {
    const existing = await this.userRepository.findByEmail(dto.email)
    if (existing) {
      throw new ConflictException('Email already exists')
    }

    return this.userRepository.create({
      email: dto.email,
      name: dto.name,
      hashedPassword: await hashPassword(dto.password),
    })
  }
}
```

## Port 인터페이스

```typescript
// features/users/ports/user-repository.port.ts
export interface UserRepositoryPort {
  findById(id: string): Promise<User | null>
  findByEmail(email: string): Promise<User | null>
  create(data: CreateUserData): Promise<User>
  update(id: string, data: UpdateUserData): Promise<User>
  delete(id: string): Promise<void>
}

export const USER_REPOSITORY_PORT = Symbol('UserRepositoryPort')
```

## Adapter (Infra) 구현

```typescript
// features/users/infra/user.repository.ts
import { Injectable } from '@nestjs/common'
import { PrismaService } from '@/shared/prisma/prisma.service'
import { UserRepositoryPort } from '../ports/user-repository.port'

@Injectable() // Adapter에만 @Injectable
export class UserRepository implements UserRepositoryPort {
  constructor(private readonly prisma: PrismaService) {}

  async findById(id: string): Promise<User | null> {
    return this.prisma.user.findUnique({ where: { id } })
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.prisma.user.findUnique({ where: { email } })
  }

  async create(data: CreateUserData): Promise<User> {
    return this.prisma.user.create({ data })
  }

  async update(id: string, data: UpdateUserData): Promise<User> {
    return this.prisma.user.update({ where: { id }, data })
  }

  async delete(id: string): Promise<void> {
    await this.prisma.user.delete({ where: { id } })
  }
}
```

## Controller 패턴

```typescript
// features/users/api/users.controller.ts
import { Controller, Post, Body, Get, Param, ParseUUIDPipe } from '@nestjs/common'
import { ApiTags, ApiOperation } from '@nestjs/swagger'
import { CreateUserUseCase } from '../usecases/create-user.usecase'
import { CreateUserDto } from './dto/create-user.dto'

@ApiTags('users')
@Controller('users')
export class UsersController {
  constructor(private readonly createUserUseCase: CreateUserUseCase) {}

  @Post()
  @ApiOperation({ summary: 'Create user' })
  create(@Body() dto: CreateUserDto) {
    return this.createUserUseCase.execute(dto)
  }
}
```

## Module 구성

```typescript
// features/users/users.module.ts
import { Module } from '@nestjs/common'
import { UsersController } from './api/users.controller'
import { CreateUserUseCase } from './usecases/create-user.usecase'
import { UserRepository } from './infra/user.repository'
import { USER_REPOSITORY_PORT } from './ports/user-repository.port'

@Module({
  controllers: [UsersController],
  providers: [
    // Adapter 등록
    {
      provide: USER_REPOSITORY_PORT,
      useClass: UserRepository,
    },
    // UseCase 등록 (Port 주입)
    {
      provide: CreateUserUseCase,
      useFactory: (repo) => new CreateUserUseCase(repo),
      inject: [USER_REPOSITORY_PORT],
    },
  ],
  exports: [CreateUserUseCase],
})
export class UsersModule {}
```

## DTO 패턴

```typescript
// features/users/api/dto/create-user.dto.ts
import { ApiProperty } from '@nestjs/swagger'
import { IsEmail, IsString, MinLength } from 'class-validator'

export class CreateUserDto {
  @ApiProperty({ example: 'john@example.com' })
  @IsEmail()
  email: string

  @ApiProperty({ example: 'John Doe' })
  @IsString()
  @MinLength(2)
  name: string

  @ApiProperty({ minLength: 8 })
  @IsString()
  @MinLength(8)
  password: string
}
```

## BullMQ 패턴

```typescript
// workers/bullmq/processors/email.processor.ts
import { Processor, WorkerHost } from '@nestjs/bullmq'
import { Job } from 'bullmq'

@Processor('email')
export class EmailProcessor extends WorkerHost {
  async process(job: Job<EmailJobData>) {
    switch (job.name) {
      case 'send-welcome':
        return this.sendWelcome(job.data)
      default:
        throw new Error(`Unknown job: ${job.name}`)
    }
  }

  private async sendWelcome(data: EmailJobData) {
    // 이메일 전송 로직
  }
}
```

```typescript
// features/users/infra/email-queue.adapter.ts
@Injectable()
export class EmailQueueAdapter implements EmailQueuePort {
  constructor(@InjectQueue('email') private queue: Queue) {}

  async sendWelcome(userId: string, email: string) {
    return this.queue.add('send-welcome', { userId, email }, {
      attempts: 3,
      backoff: { type: 'exponential', delay: 1000 },
    })
  }
}
```

## 에러 처리

**UseCase에서 비즈니스 오류는 NestJS HTTP 예외 사용:**
- `NotFoundException` (404)
- `BadRequestException` (400)
- `ConflictException` (409)
- `UnauthorizedException` (401)
- `ForbiddenException` (403)

❌ 일반 `Error` throw 금지 → 500 에러로 처리됨

## 품질 체크리스트

### 아키텍처
- [ ] `*.service.ts` 파일이 없는가?
- [ ] UseCase에 `@Injectable()` 데코레이터가 없는가?
- [ ] 외부 의존성이 Port 인터페이스로 추상화되어 있는가?
- [ ] Feature 간 직접 참조가 없는가?

### 코드 품질
- [ ] Controller에 비즈니스 로직이 없는가?
- [ ] DB 모델을 API 응답으로 직접 반환하지 않는가?
- [ ] DTO에 유효성 검증 데코레이터가 있는가?

## Red Flags

**Never:**
- `*.service.ts` 파일 생성
- UseCase에 `@Injectable()` 사용
- UseCase에서 Adapter 직접 의존
- Feature 간 직접 import
- Controller에 비즈니스 로직

**Always:**
- UseCase = 1 파일 = 1 행동
- Port 인터페이스로 추상화
- DTO로 입력 검증
- Swagger 데코레이터로 문서화
