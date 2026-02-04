---
name: nextjs-frontend
description: Use when working on Next.js frontend with FSD architecture, Zustand, React Query, and shadcn/ui - provides Feature-Sliced Design patterns
---

# Next.js Frontend Stack (FSD Architecture)

## Overview

Next.js 16 + Feature-Sliced Design + Zustand + React Query + shadcn/ui 스택의 best practices입니다.

**핵심 철학:** "레이어로 분리하고, 상위에서 하위로만 의존한다."

**이 스킬은 참조용입니다.** 코드 작성 시 이 패턴을 따르세요.

## Core Design Principles

1. **Minimize noise** - 아이콘으로 소통, 과도한 레이블 지양
2. **No generic AI-UI** - 보라색 그라데이션, 과도한 그림자, 예측 가능한 레이아웃 피하기
3. **Context over decoration** - 모든 요소는 목적이 있어야 함
4. **Theme consistency** - `globals.css` CSS 변수 사용, 색상 하드코딩 금지

## FSD 레이어 구조 (with shadcn)

```
src/
├── app/                    # App layer - Next.js App Router, providers
│   ├── (protected)/       # Auth required routes
│   │   ├── dashboard/
│   │   ├── settings/
│   │   ├── components/    # Route-specific components
│   │   └── lib/           # Route-specific utils/types
│   ├── (public)/          # Public routes
│   │   ├── login/
│   │   └── register/
│   ├── actions/           # Server Actions (global)
│   ├── api/               # API routes
│   ├── providers.tsx      # Global providers
│   ├── layout.tsx
│   ├── globals.css        # Theme tokens
│   └── page.tsx
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
├── shared/                 # Shared layer - 재사용 가능한 코드
│   ├── api/               # API client
│   ├── config/            # Environment config
│   ├── lib/               # Utilities (cn, utils)
│   └── ui/                # shadcn/ui components
├── components/             # Non-FSD shared components
│   ├── ui/                # shadcn primitives (alternative location)
│   ├── layout/            # Layout components (sidebar, nav)
│   ├── backgrounds/       # Grid, Dot, Gradient patterns
│   └── animations/        # FadeIn, ScrollReveal
├── hooks/                  # Custom React hooks
└── data/                   # Database queries ("use cache")
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

## Next.js 16 Features

### Async Params

```tsx
export default async function Page({
  params,
  searchParams,
}: {
  params: Promise<{ id: string }>;
  searchParams: Promise<{ q?: string }>;
}) {
  const { id } = await params;
  const { q } = await searchParams;
}
```

### Data Fetching vs Server Actions

**CRITICAL RULE:**
- **Server Actions** = ONLY for mutations (create, update, delete)
- **Data fetching** = In Server Components or `'use cache'` functions

```tsx
// ❌ WRONG: Server Action for data fetching
"use server"
export async function getUsers() {
  return await db.users.findMany()
}

// ✅ CORRECT: Data function with caching
// data/users.ts
export async function getUsers() {
  "use cache"
  cacheTag("users")
  cacheLife("hours")
  return await db.users.findMany()
}

// ✅ CORRECT: Read cookies in Server Component directly
export default async function Page() {
  const theme = (await cookies()).get("theme")?.value ?? "light"
  return <App theme={theme} />
}
```

### Caching

```tsx
"use cache";

import { cacheTag, cacheLife } from "next/cache";

export async function getProducts() {
  cacheTag("products");
  cacheLife("hours");
  return await db.products.findMany();
}
```

### Server Actions (Mutations Only)

```tsx
"use server";

import { updateTag, revalidateTag } from "next/cache";
import { z } from "zod";

const schema = z.object({
  title: z.string().min(1),
  content: z.string(),
});

export async function createPost(formData: FormData) {
  // Always validate input
  const parsed = schema.parse({
    title: formData.get("title"),
    content: formData.get("content"),
  });

  await db.insert(posts).values(parsed);
  updateTag("posts"); // Read-your-writes
}
```

### Proxy API

Use `proxy.ts` for request interception (replaces middleware). Place at project root:

```tsx
// proxy.ts (project root, same level as app/)
import { NextResponse } from "next/server"
import type { NextRequest } from "next/server"

export function proxy(request: NextRequest) {
  // Auth checks, redirects, etc.
}

export const config = {
  matcher: ['/dashboard/:path*'],
}
```

## Component Patterns

### Client Boundaries

- `"use client"` only at leaf components (smallest boundary)
- Props must be serializable (data or Server Actions, no functions/classes)
- Pass server content via `children`

### className Pattern

Always accept and merge `className`:

```tsx
import { cn } from "@/lib/utils"

interface CardProps extends React.HTMLAttributes<HTMLDivElement> {
  variant?: "default" | "outline"
}

export function Card({ className, variant = "default", ...props }: CardProps) {
  return (
    <div
      className={cn(
        "rounded-lg p-4",
        variant === "outline" && "border",
        className
      )}
      {...props}
    />
  )
}
```

### Server vs Client Decision Tree

```
Need state/effects/browser APIs?
├── Yes → "use client" at smallest boundary
└── No → Server Component (default)

Passing data to client?
├── Functions/classes → ❌ Not serializable
├── Plain objects/arrays → ✅ Props
└── Server logic → ✅ Server Actions
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

## References

- **Architecture**: [references/architecture.md](references/architecture.md) - Server/Client patterns, Suspense, data fetching
- **Styling**: [references/styling.md](references/styling.md) - Themes, fonts, animations, background patterns
- **Sidebar**: [references/sidebar.md](references/sidebar.md) - shadcn sidebar with nested layouts
- **Project Setup**: [references/project-setup.md](references/project-setup.md) - Commands, presets

## 체크리스트

### FSD 아키텍처
- [ ] 상위 레이어에서 하위 레이어로만 의존하는가?
- [ ] 같은 레이어의 다른 slice를 직접 import하지 않는가?
- [ ] 각 slice의 public API(index.ts)만 export하는가?
- [ ] shared/에 비즈니스 로직이 없는가?

### Next.js 16
- [ ] Server Actions는 mutation만 사용하는가?
- [ ] Data fetching은 Server Component 또는 "use cache"인가?
- [ ] async params/searchParams를 await하는가?

### 코드 품질
- [ ] Zustand selector로 필요한 state만 구독하는가?
- [ ] React Query keys factory 패턴을 사용하는가?
- [ ] Server Component 우선, 필요시만 Client Component?
- [ ] className을 cn()으로 merge하는가?

## Red Flags

**Never:**
- 하위 레이어에서 상위 레이어 import
- entities에서 features import
- shared에서 다른 레이어 import
- 전체 Zustand state 구독
- Server Component에서 hooks 사용
- Server Actions으로 data fetching
- API 키 클라이언트에 노출
- 색상 하드코딩 (bg-blue-500 대신 bg-primary)

**Always:**
- slice의 index.ts로만 export
- Query keys factory 패턴 사용
- Form validation은 Zod + React Hook Form
- shadcn/ui는 shared/ui/ 또는 components/ui/에 위치
- CSS 변수로 테마 색상 사용
