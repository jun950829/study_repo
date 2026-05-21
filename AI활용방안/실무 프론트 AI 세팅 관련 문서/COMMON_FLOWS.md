# COMMON_FLOWS — 관리자 화면 공통 흐름

새 도메인 화면을 만들 때 반복되는 목록, 등록, 상세, 수정, 모달, 테이블 상태 흐름을 정의한다.
Claude는 새 화면 구현 전에 이 문서를 읽고 기존 흐름을 재사용한다.

---

## 1. App Router 페이지 흐름

`page.tsx`는 Server Component로 유지하고 View만 import한다.

```tsx
import SomeView from '@/components/views/some-feature';

export default function Page() {
  return <SomeView />;
}
```

클라이언트 상태, form, query, mutation, router 이동은 `src/components/views/{feature}` 내부의 `'use client'` 컴포넌트에서 처리한다.

---

## 2. 목록 화면 흐름

기본 파일 구조.

- `src/app/(auth)/{route}/page.tsx`
- `src/components/views/{feature}/index.tsx`
- `src/components/table-columns/{feature}/index.tsx`
- `src/hook/{feature}/useGet{Feature}List.ts`

기본 구성.

1. page title
2. BreadCrumb
3. 검색/필터 card
4. list section title
5. DataTable
6. pagination
7. 등록 버튼 또는 row action

규칙.

- `DataTable`을 우선 사용한다.
- 데이터가 없어도 table header는 유지한다.
- `isLoading`, `isError`, `data.length === 0`은 table body 내부 상태로 처리한다.
- row action은 `Button variant="icon-action"`과 `Icons` registry를 우선 사용한다.
- column 정의는 `src/components/table-columns/{feature}`에 둔다.

---

## 3. 등록 화면 흐름

기본 파일 구조.

- `src/app/(auth)/{route}/create/page.tsx`
- `src/components/views/{feature}/create.tsx` 또는 `crud.tsx`
- `src/hook/{feature}/usePost{Feature}.ts`
- `src/utils/validations.ts`

기본 구성.

1. page title
2. BreadCrumb
3. form card
4. section title
5. form rows
6. bottom buttons

규칙.

- react-hook-form + zodResolver + Zod를 사용한다.
- 신규 schema는 `src/utils/validations.ts`에 추가한다.
- API가 없으면 mock submit 또는 TODO 1개로 제한한다.
- 취소, 목록, 뒤로가기 버튼은 dirty 상태에서 `useLeaveGuard`를 적용한다.
- 저장 성공/실패 안내는 `useAlert()`로 처리한다.

---

## 4. 상세 화면 흐름

기본 파일 구조.

- `src/app/(auth)/{route}/[id]/page.tsx`
- `src/components/views/{feature}/detail.tsx`
- `src/hook/{feature}/useGet{Feature}Detail.ts`

기본 구성.

1. page title
2. BreadCrumb
3. detail card
4. read-only rows
5. bottom buttons

규칙.

- 정적 row는 `DetailPanel` 패턴을 우선 사용한다.
- editor HTML은 read-only preview로 렌더링한다.
- 목록 버튼은 list route로 이동한다.
- 수정 버튼은 edit route로 이동한다.
- 삭제/탈퇴/비활성 등 확인이 필요한 동작은 `useAlert()` 또는 공통 Dialog를 사용한다.

---

## 5. 수정 화면 흐름

기본 파일 구조.

- `src/app/(auth)/{route}/[id]/edit/page.tsx`
- `src/components/views/{feature}/edit.tsx` 또는 `crud.tsx`
- `src/hook/{feature}/useGet{Feature}Detail.ts`
- `src/hook/{feature}/usePatch{Feature}.ts`

기본 구성은 등록 화면과 동일하다.

규칙.

- 상세 query 성공 후 `form.reset()`으로 초기값을 넣는다.
- Select value는 string으로 변환한다.
- 취소, 목록, 뒤로가기 버튼은 dirty 상태에서 `useLeaveGuard`를 적용한다.
- 저장 성공 후 detail 또는 list route로 이동한다.
- mutation 성공 시 관련 queryKey를 invalidate한다.

---

## 6. 모달 흐름

단순 confirm/alert.

- `useAlert()`를 우선 사용한다.
- 확인 전용은 `cancelText: null`을 사용한다.
- 확인/취소는 `confirmText`, `cancelText`를 지정한다.

form 또는 복합 UI 모달.

- `Dialog`를 사용한다.
- Figma title, description, button text를 그대로 사용한다.
- form submit은 mutation 또는 mock handler로 분리한다.

---

## 7. Editor 흐름

editor가 필요한 화면은 기존 공통 editor를 사용한다.

- 경로: `src/components/shared/editor/index.tsx`
- 새 editor 라이브러리 설치 금지
- toolbar, content area, upload 관련 설정은 기존 구현을 따른다.
- 상세 화면은 read-only preview로 렌더링한다.
- 등록/수정 화면은 RHF field와 연결하거나 local state로 관리하되 submit payload에 포함한다.

---

## 8. Date/Period 흐름

날짜, 시간, 기간 입력은 기존 `DateTimeInput`을 우선 사용한다.

- 경로: `src/components/shared/DateTimeInput.tsx`
- mode는 기존 컴포넌트가 제공하는 값 안에서 선택한다.
- 게시일자처럼 제한이 있으면 오늘 포함 이후 날짜만 선택 가능하게 처리한다.
- 검색 기간은 기존 `PeriodFilter` 또는 `PeriodPicker` 패턴을 우선 확인한다.

---

## 9. Table Empty/Error 흐름

모든 table 작업의 기본값.

- table wrapper와 header는 항상 렌더링한다.
- loading, error, empty는 body row 하나로 처리한다.
- body row는 전체 column을 `colSpan`으로 병합한다.
- empty 기본 문구: `현재 표시할 내용이 없습니다.`
- error 기본 문구: `데이터를 불러오지 못했습니다.`
- 시각 스펙은 `AI_CONTEXT/UI_DEFAULTS.md`를 따른다.

---

## 10. Leave Guard 흐름

등록/수정 화면에서 사용한다.

```tsx
const { isDirty } = form.formState;
const { guardedNavigate } = useLeaveGuard(isDirty);

<Button type="button" onClick={() => guardedNavigate(() => router.push('/list-route'))}>
  취소
</Button>
```

규칙.

- 저장 성공 후 이동에는 dirty confirm을 띄우지 않는다.
- 사용자가 직접 누른 취소, 목록, 뒤로가기성 버튼에 적용한다.
- 브라우저 뒤로가기는 `useLeaveGuard` 내부 구현을 따른다.

---

## 11. Route/Menu 흐름

새 도메인 route를 만들 때 확인한다.

- `src/constants/routes.ts`
- `src/app/(auth)/...`
- `src/components/side-menu`

이미 `routes.ts`의 `MENU`에 정의된 경로면 그 경로를 우선 사용한다.
Figma 화면명보다 실제 route naming을 우선한다.
