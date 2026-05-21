# RENDERING_RULES — Next.js 렌더링 패턴

## App Router 구조

```
app/
├── layout.tsx          # 루트 레이아웃 — RootProvider + Pretendard 폰트 (Server Component)
├── page.tsx            # / → redirect('/dashboard') (Server Component)
├── (auth)/
│   ├── layout.tsx      # AuthLayout — SideMenu + Header + <main> (Server Component)
│   └── ...page.tsx     # 각 페이지 (Server Component, View를 import)
└── (no-auth)/
    └── sign-in/page.tsx
```

---

## 라우트 그룹

| 그룹 | 경로 | 레이아웃 |
|---|---|---|
| `(auth)` | /dashboard, /users/*, /members/*, ... | SideMenu + Header + `<main className="flex-1 overflow-y-auto bg-admin-gray-1 p-6">` |
| `(no-auth)` | /sign-in | 레이아웃 없음 (중첩 layout.tsx 없음) |

`(auth)/layout.tsx` 구조.
```tsx
<div className="flex h-screen overflow-hidden">
  <SideMenu />
  <div className="flex flex-1 flex-col overflow-hidden">
    <Header />
    <main className="flex-1 overflow-y-auto bg-admin-gray-1 p-6">{children}</main>
  </div>
</div>
```

---

## page.tsx → View 패턴

`page.tsx`는 Server Component. 클라이언트 상태/훅이 필요한 모든 로직은 View 컴포넌트에 위임한다.

```tsx
// app/(auth)/users/page.tsx — Server Component
import UserListView from '@/components/views/user';
export default function UsersPage() {
  return <UserListView />;
}

// app/(auth)/users/[id]/page.tsx
import UserCrudView from '@/components/views/user/crud';
export default function UserEditPage({ params }: { params: { id: string } }) {
  return <UserCrudView id={params.id} />;
}

// app/(auth)/users/create/page.tsx
import UserCrudView from '@/components/views/user/crud';
export default function UserCreatePage() {
  return <UserCrudView />;
}
```

---

## 'use client' 경계

| 파일 유형 | 'use client' |
|---|---|
| `app/**/page.tsx` | 없음 (Server Component) |
| `app/**/layout.tsx` | 없음 (Server Component) |
| `src/components/views/**` | 있음 — 훅/상태 사용 |
| `src/components/shared/**` | 필요한 경우만 있음 |
| `src/components/ui/**` | 파일마다 다름 — `select.tsx` 등 Radix 인터랙티브 래퍼는 `'use client'` 보유, `button.tsx` 등 정적 래퍼는 없음 |
| `src/components/table-columns/**` | 있음 — DropdownMenu 등 이벤트 핸들러 포함 |

---

## RootProvider 구성

파일: `src/providers/root-provider.tsx`

```tsx
// QueryProvider → AlertProvider → Suspense 순서
<QueryProvider>
  <AlertProvider>
    <Suspense>{children}</Suspense>
  </AlertProvider>
</QueryProvider>
```

- `RootProvider` 자체가 `'use client'` — 루트 layout.tsx에서 import하면 자동으로 클라이언트 트리 시작점.
- `Suspense`는 `useSearchParams()`를 사용하는 컴포넌트(usePagination 등)에 필요.

---

## 루트 레이아웃

파일: `src/app/layout.tsx`

- Pretendard 가변 폰트를 `next/font/local`로 로드 후 CSS 변수(`--font-pretendard`)로 주입.
- `<html lang="ko">`, `<body className={pretendard.variable}>` 설정.
- `RootProvider`로 children 감쌈.

---

## 미들웨어

파일: `src/middleware.ts`

- 현재 인증 가드 로직이 **주석 처리** 상태. API 연결 후 활성화 예정.
- `matcher` 설정만 남아 있음.

---

## 페이지 추가 시 체크리스트

1. `src/app/(auth)/{feature}/page.tsx` — Server Component, View import만.
2. `src/components/views/{feature}/index.tsx` — `'use client'`, 실제 로직 구현.
3. 목록이면 `src/components/table-columns/{feature}/index.tsx` — ColumnDef 정의.
4. CRUD면 `crud.tsx` 별도 파일로 분리.
5. `src/constants/routes.ts`의 `ROUTES` / `MENU` 상수에 경로 추가.
