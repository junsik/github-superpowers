---
name: nextjs-frontend
description: Use when working on Next.js frontend with FSD architecture, Zustand, React Query, and shadcn/ui - provides Feature-Sliced Design patterns
---

# Next.js Frontend Stack (FSD Architecture)

## Overview

Next.js + Feature-Sliced Design + Zustand + React Query + shadcn/ui 스택의 best practices입니다.

**핵심 철학:** "레이어로 분리하고, 상위에서 하위로만 의존한다."

**이 스킬은 참조용입니다.** 코드 작성 시 이 패턴을 따르세요.

## FSD 레이어 구조

```
src/
├── app/                    # App layer - Next.js App Router, providers
│   ├── (auth)/            # Route groups
│   ├── api/               # API routes
│   ├── providers.tsx      # Global providers
│   ├── layout.tsx
│   └── page.tsx
├── pages/                  # Pages layer (if using pages router)
├── widgets/                # Widget layer - 독립적인 UI 블록
│   └── header/
│       ├── ui/
│       └── index.ts
├── features/               # Features layer - 사용자 시나리오
│   └── auth/
│       ├── api/           # Feature API hooks
│       ├── model/         # Feature state (Zustand)
│       ├── ui/            # Feature components
│       └── index.ts       # Public API
├── entities/               # Entities layer - 비즈니스 엔티티
│   └── user/
│       ├── api/           # Entity API hooks
│       ├── model/         # Entity types & stores
│       ├── ui/            # Entity UI components
│       └── index.ts
└── shared/                 # Shared layer - 재사용 가능한 코드
    ├── api/               # API client
    ├── config/            # Environment config
    ├── lib/               # Utilities
    └── ui/                # shadcn/ui components
```

## 레이어 의존성 규칙

```
app → pages → widgets → features → entities → shared
                    ↓           ↓          ↓        ↓
                    └───────────┴──────────┴────────┘
                          (하위 레이어만 의존 가능)
```

**의존성 규칙:**
- ✅ features → entities (가능)
- ✅ features → shared (가능)
- ❌ entities → features (금지)
- ❌ shared → 어떤 레이어도 의존 금지

## Slice 구조

각 레이어의 슬라이스는 동일한 구조:

```
{slice}/
├── api/           # API 호출, React Query hooks
├── model/         # 상태 관리, Zustand stores, types
├── ui/            # UI 컴포넌트
├── lib/           # 유틸리티 (해당 slice 전용)
└── index.ts       # Public API (re-exports)
```

## Zustand Store 패턴 (FSD)

```typescript
// features/auth/model/auth.store.ts
import { create } from 'zustand'
import { persist } from 'zustand/middleware'

interface AuthState {
  user: User | null
  token: string | null
  setUser: (user: User | null) => void
  setToken: (token: string | null) => void
  logout: () => void
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      token: null,
      setUser: (user) => set({ user }),
      setToken: (token) => set({ token }),
      logout: () => set({ user: null, token: null }),
    }),
    { name: 'auth-storage' }
  )
)
```

**Selector 패턴 (리렌더링 최적화):**

```typescript
// ✅ 필요한 state만 구독
const user = useAuthStore((state) => state.user)
const { setUser, logout } = useAuthStore()

// ❌ 전체 state 구독 (불필요한 리렌더링)
const { user, token, setUser } = useAuthStore()
```

## React Query 패턴 (FSD)

```typescript
// entities/user/api/user.queries.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'
import { api } from '@/shared/api'

// Query Keys Factory
export const userKeys = {
  all: ['users'] as const,
  lists: () => [...userKeys.all, 'list'] as const,
  list: (filters: string) => [...userKeys.lists(), { filters }] as const,
  details: () => [...userKeys.all, 'detail'] as const,
  detail: (id: string) => [...userKeys.details(), id] as const,
}

// Queries
export function useUsers(filters?: string) {
  return useQuery({
    queryKey: userKeys.list(filters ?? ''),
    queryFn: () => api.users.list(filters),
  })
}

export function useUser(id: string) {
  return useQuery({
    queryKey: userKeys.detail(id),
    queryFn: () => api.users.get(id),
    enabled: !!id,
  })
}

// Mutations
export function useCreateUser() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: api.users.create,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: userKeys.lists() })
    },
  })
}
```

## Feature Public API

```typescript
// features/auth/index.ts
// UI
export { LoginForm } from './ui/login-form'
export { LogoutButton } from './ui/logout-button'

// Model
export { useAuthStore } from './model/auth.store'
export type { AuthState } from './model/auth.store'

// API
export { useLogin, useLogout } from './api/auth.mutations'
```

**다른 레이어에서 사용:**

```typescript
// widgets/header/ui/header.tsx
import { LogoutButton, useAuthStore } from '@/features/auth'
import { UserAvatar } from '@/entities/user'

export function Header() {
  const user = useAuthStore((s) => s.user)

  return (
    <header>
      {user && <UserAvatar user={user} />}
      <LogoutButton />
    </header>
  )
}
```

## Entity UI 컴포넌트

```typescript
// entities/user/ui/user-card.tsx
import { Card, CardContent, CardHeader, CardTitle } from '@/shared/ui/card'
import { Avatar, AvatarFallback, AvatarImage } from '@/shared/ui/avatar'
import { User } from '../model/user.types'

interface UserCardProps {
  user: User
  actions?: React.ReactNode  // slot for feature-level actions
}

export function UserCard({ user, actions }: UserCardProps) {
  return (
    <Card>
      <CardHeader>
        <CardTitle className="flex items-center gap-2">
          <Avatar>
            <AvatarImage src={user.avatar} />
            <AvatarFallback>{user.name[0]}</AvatarFallback>
          </Avatar>
          {user.name}
        </CardTitle>
      </CardHeader>
      <CardContent>
        <p className="text-muted-foreground">{user.email}</p>
        {actions}
      </CardContent>
    </Card>
  )
}
```

## Form with Zod + React Hook Form

```typescript
// features/user/ui/user-form.tsx
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from '@/shared/ui/form'
import { Input } from '@/shared/ui/input'
import { Button } from '@/shared/ui/button'

const userSchema = z.object({
  name: z.string().min(2, 'Name must be at least 2 characters'),
  email: z.string().email('Invalid email'),
})

type UserFormValues = z.infer<typeof userSchema>

export function UserForm({ onSubmit, defaultValues }) {
  const form = useForm<UserFormValues>({
    resolver: zodResolver(userSchema),
    defaultValues,
  })

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
        <FormField
          control={form.control}
          name="name"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Name</FormLabel>
              <FormControl>
                <Input {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        <Button type="submit">Submit</Button>
      </form>
    </Form>
  )
}
```

## API Client (Shared)

```typescript
// shared/api/client.ts
const BASE_URL = process.env.NEXT_PUBLIC_API_URL

async function fetchAPI<T>(
  endpoint: string,
  options?: RequestInit
): Promise<T> {
  const { useAuthStore } = await import('@/features/auth')
  const token = useAuthStore.getState().token

  const res = await fetch(`${BASE_URL}${endpoint}`, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      ...(token && { Authorization: `Bearer ${token}` }),
      ...options?.headers,
    },
  })

  if (!res.ok) {
    const error = await res.json()
    throw new Error(error.message || 'API Error')
  }

  return res.json()
}

export const api = {
  users: {
    list: (filters?: string) => fetchAPI<User[]>(`/users?${filters}`),
    get: (id: string) => fetchAPI<User>(`/users/${id}`),
    create: (data: CreateUser) => fetchAPI<User>('/users', {
      method: 'POST',
      body: JSON.stringify(data),
    }),
  },
}
```

## Server vs Client Components

```typescript
// app/users/page.tsx (Server Component)
import { UserList } from '@/widgets/user-list'
import { AddUserButton } from '@/features/user'

export default async function UsersPage() {
  // 서버에서 초기 데이터 fetch
  const initialUsers = await fetch('/api/users').then(r => r.json())

  return (
    <div>
      <UserList initialUsers={initialUsers} />
      <AddUserButton /> {/* Client Component */}
    </div>
  )
}

// features/user/ui/add-user-button.tsx
'use client'

import { useCreateUser } from '../api/user.mutations'

export function AddUserButton() {
  const { mutate, isPending } = useCreateUser()
  // ...
}
```

## Import Aliases

```typescript
// tsconfig.json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"],
      "@/app/*": ["./src/app/*"],
      "@/widgets/*": ["./src/widgets/*"],
      "@/features/*": ["./src/features/*"],
      "@/entities/*": ["./src/entities/*"],
      "@/shared/*": ["./src/shared/*"]
    }
  }
}
```

## 체크리스트

### FSD 아키텍처
- [ ] 상위 레이어에서 하위 레이어로만 의존하는가?
- [ ] 같은 레이어의 다른 slice를 직접 import하지 않는가?
- [ ] 각 slice의 public API(index.ts)만 export하는가?
- [ ] shared/에 비즈니스 로직이 없는가?

### 코드 품질
- [ ] Zustand selector로 필요한 state만 구독하는가?
- [ ] React Query keys factory 패턴을 사용하는가?
- [ ] Server Component 우선, 필요시만 Client Component?

## Red Flags

**Never:**
- 하위 레이어에서 상위 레이어 import
- entities에서 features import
- shared에서 다른 레이어 import
- 전체 Zustand state 구독
- Server Component에서 hooks 사용
- API 키 클라이언트에 노출

**Always:**
- slice의 index.ts로만 export
- Query keys factory 패턴 사용
- Form validation은 Zod + React Hook Form
- shadcn/ui는 shared/ui/에 위치
