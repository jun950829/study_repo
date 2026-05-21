# QUERY_RULES — 서버 상태 / API 패턴

## agent() — 공통 fetch 래퍼

파일: `src/utils/fetch.ts`

```ts
const data = await agent<ResponseType>('/api/admin/users', {
  method: 'POST',
  body: JSON.stringify(payload),
});
```

**동작:**
- `NEXT_PUBLIC_API_URL` + `url` 조합으로 요청
- `js-cookie`에서 `ACCESS_TOKEN` 읽어 `Authorization: Bearer {token}` 자동 주입
- `body`가 `FormData`이면 `Content-Type` 헤더 생략 (브라우저 자동 처리)
- `response.ok` 실패 시 → `throw { status, ...errorData }` (ErrorData 타입)
- `errorData.code === 4001` → `alert('권한이 없습니다.')` + `/sign-in` 강제 이동

**주의:** `agent()`는 브라우저 전용. Server Component에서 직접 호출하지 않는다.

---

## 응답 타입

```ts
// 단건
interface IApiResponse<T> { code: number; message: string; data: T; }
// 사용 예: agent<{ data: UserDetail }>('/api/admin/users/123') → .data

// 페이지네이션 목록
interface IResponsePaged<T> {
  content: T[];
  totalElements: number;
  totalPages: number;
  number: number;      // 현재 페이지 (0-based)
  size: number;
  first: boolean;
  last: boolean;
}
```

---

## Query Hook 파일 구조

한 파일에 순수 API 함수와 Query 훅을 함께 정의한다.

```ts
// src/hook/user/useGetUserList.ts

// 1. 응답 타입
export interface IUserListResponse { ... }

// 2. 순수 API 함수 (테스트/직접 호출 가능)
export const GetUserList = async (params): Promise<IResponsePaged<IUserListResponse>> => {
  const data = await agent<{ data: IResponsePaged<IUserListResponse> }>(`/api/admin/users?${query}`);
  return data.data;
};

// 3. Query 훅
export const useGetUserList = (params) => {
  return useQuery({
    queryKey: [userListKey, params],
    queryFn: () => GetUserList(params),
  });
};
```

---

## Query Key 관리

파일: `src/constants/queryKeys.ts`

```ts
const userListKey = 'userList';
export { userListKey };
```

**prefix 공유 전략:**
- 목록: `queryKey: [userListKey, params]`
- 상세: `queryKey: [userListKey, uuid]`
- 무효화: `queryClient.invalidateQueries({ queryKey: [userListKey] })`
  → prefix `userListKey`가 포함된 **목록·상세 모두** 한 번에 무효화됨

새 feature 추가 시 `queryKeys.ts`에 상수 추가:
```ts
const memberListKey = 'memberList';
export { memberListKey };
```

---

## Mutation Hook 패턴

```ts
// src/hook/user/usePostUser.ts
export const usePostUser = () => {
  const alert = useAlert();
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: PostUser,
    onSuccess: async () => {
      await queryClient.invalidateQueries({ queryKey: [userListKey] });
      await alert({ title: '사용자가 등록되었습니다.', cancelText: null });
    },
    onError: (error: ErrorData) => {
      alert({ title: error.message ?? '오류가 발생했습니다.', cancelText: null });
    },
  });
};
```

- `onSuccess`: `invalidateQueries` 먼저 → `alert` 순서 유지
- `cancelText: null` → 확인 버튼만 있는 알림 모달
- `error: ErrorData` → `error.message` 우선, 없으면 기본 메시지

---

## QueryClient 설정

```ts
// src/providers/query-provider.tsx
new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60 * 1000,      // 1분 이내 재요청 시 캐시 사용
      gcTime: 5 * 60 * 1000,     // 5분 후 캐시 GC
      retry: 1,                   // 실패 시 1회 재시도
    },
  },
})
```

---

## Pagination Hook

파일: `src/hook/usePagination.ts`

```ts
const { page, size, onPaginationChange } = usePagination({
  defaultPage: 1,
  defaultSize: 10,
  preserveParams: true,   // 다른 searchParam 유지 여부
});
```

- `page`, `size`: URL `?page=1&size=10` 에서 읽음
- `onPaginationChange(newPage, newSize)`: URL 업데이트 (router.push)
- `usePagination`은 `useSearchParams`를 사용하므로 반드시 `'use client'` 컴포넌트에서 호출

**DataTable 연동 패턴:**
```tsx
const { page, size, onPaginationChange } = usePagination();
const { data } = useGetUserList({ page, size });

<DataTable
  columns={columns}
  data={data?.content ?? []}
  isPagination
  totalElements={data?.totalElements}
  pageCount={data?.totalPages}
  currentPage={page}
  pageSize={size}
  onPaginationChange={onPaginationChange}
  manualPagination
/>
```

---

## 전역 Alert / Confirm

파일: `src/providers/alert-provider.tsx`

```ts
const alert = useAlert();

// 확인 전용 알림
await alert({ title: '저장되었습니다.', cancelText: null });

// 확인/취소 confirm
const confirmed = await alert({
  title: '삭제하시겠습니까?',
  description: '삭제 후 복구가 불가능합니다.',
  confirmText: '삭제',
});
if (confirmed) { ... }
```

- Promise 기반. `await`으로 사용자 응답 대기.
- `cancelText: null` → 취소 버튼 숨김 (알림용)
- `confirmText` 기본값: `'확인'`, `cancelText` 기본값: `'취소'`
