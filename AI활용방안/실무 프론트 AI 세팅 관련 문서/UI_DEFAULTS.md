# UI_DEFAULTS — 프로젝트 공통 UX 기본값

프로젝트 전체에서 반복 등장하는 공통 UI 패턴 4종을 Figma 디자인과 1:1로 정의한다.
새 화면을 만들거나 기존 화면을 수정할 때 아래 패턴 중 하나라도 등장하면 반드시 이 문서의 스펙을 따른다.

대상 패턴.
- 정렬 가능한 테이블 헤더 (sort)
- 테이블 데이터 0건 (empty)
- 드롭다운 옵션 0개 (empty)
- 등록·수정 중 페이지 이탈 alert (leave guard)

---

## 0. 표준 컴포넌트 인벤토리 (현재 상태 vs 목표)

| 패턴 | 표준 컴포넌트 경로 | 현재 상태 | 비고 |
|---|---|---|---|
| §1 테이블 정렬 헤더 | `src/components/ui/sortable-header.tsx` (목표) | **신규 필요** | 현재 `sort-icon.tsx` 는 방향/selected 분기 없음 |
| §2 테이블 Empty | `src/components/shared/data-table.tsx` (내장) | **구현 완료** | header 유지 + loading/error/empty body row 구현됨. 전용 `PriorityHigh` 아이콘만 미적용 |
| §3 드롭다운 Empty | `MultiSelectDropdown` / `select.tsx` 내부 | **부분 구현** | 검색 결과 0 케이스만 있음. 옵션 0 케이스 분기 추가 |
| §4 페이지 이탈 Alert | `src/hook/useLeaveGuard.ts` + `useAlert()` (구현 완료) | **완료** | 별도 leave-alert 컴포넌트 없이 `useAlert()` 재사용 결정 |

누락된 SVG (`src/public/svg/icons/` 기준).
- `custom/order_up.svg` — 정렬 ascending 아이콘. 신규 추가 + `Icons.OrderUp` 등록 필요.
- `priority_high.svg` — 테이블 빈 상태 아이콘 (40px 컨테이너 안의 24px 느낌표). 신규 추가 + `Icons.PriorityHigh` 등록 필요.

---

## 1. 테이블 정렬 (Sortable Column Header)

**Figma**. https://www.figma.com/design/RmYvmE5RaSYISPTHVHbcLj/?node-id=878-44512 (정렬 선택 시),
node 624-46732 (정렬 가능 동시 노출).

### 트리거
- 모든 `DataTable` 에서 column 이 정렬 가능하면 헤더 우측에 아이콘 노출.
- 첫 진입 시 **기본 정렬 컬럼은 selected 상태 + 방향(asc 또는 desc)** 로 노출한다.

### 시각 스펙
헤더 텍스트 우측 8px gap, 아이콘 20×20.

| 상태 | 아이콘 (Icons.\*) | 색상 |
|---|---|---|
| 정렬 가능, 미선택 | `ArrowDropDown` (custom/arrow_drop_down.svg) | `admin-gray-5` (#868e96) |
| 선택 + 오름차순 | `OrderUp` (custom/order_up.svg) — **신규 추가** | `admin-primary` (#282843) |
| 선택 + 내림차순 | `OrderDown` (custom/order_down.svg) | `admin-primary` (#282843) |

헤더 셀은 `cursor-pointer`, 클릭 영역 전체. 키보드 접근은 `<button>` 으로 감싸 Enter/Space 지원.

### 상호작용 규칙
- 클릭 사이클. **미선택 → 컬럼의 기본 방향 → 반대 방향 → 미선택** (또는 사이클 마지막에서 기본 방향으로 복귀).
- 다른 컬럼 클릭 시 현재 컬럼은 미선택, 새 컬럼이 기본 방향으로 selected.
- 정렬은 서버사이드 호출. URL query (`?sort=field,asc`) 와 동기화하면 새로고침/뒤로가기에도 일관성 유지.
- **첫 진입 시 default sort 컬럼은 화면별로 다르므로 페이지 컨테이너에서 명시 지정한다** (예. `defaultSort: { id: 'createdAt', dir: 'desc' }`). DataTable 은 이를 받아 첫 렌더에서 selected 상태로 그린다.

### API 권장 형태
```tsx
<DataTable
  columns={columns}
  data={data}
  defaultSort={{ id: 'createdAt', dir: 'desc' }}
  onSortChange={(sort) => refetch({ sort })}
/>
```

`columns` 정의에서 정렬 가능 표시.
```tsx
{ id: 'createdAt', header: '최초 등록 일시', sortable: true, ... }
```

### 구현 체크리스트
- [ ] `order_up.svg` 추가 + `Icons.OrderUp` 등록.
- [ ] `sortable-header.tsx` 작성. props. `label`, `state: 'none' | 'asc' | 'desc'`, `onToggle`.
- [ ] `DataTable` 에 `sortable`, `defaultSort`, `onSortChange` 추가. 헤더 렌더에 `SortableHeader` 사용.
- [ ] 화면 컴포넌트에서 `defaultSort` 를 컬럼 정의와 함께 지정.

---

## 2. 테이블 Empty State (데이터 0건)

**Figma**. https://www.figma.com/design/RmYvmE5RaSYISPTHVHbcLj/?node-id=624-46732

### 트리거
- `DataTable` 의 `data.length === 0` (로딩 끝난 후).
- 로딩 중에는 별도 스켈레톤/스피너 사용 (이 문서 범위 외).

### 시각 스펙
- 위치. 테이블 헤더 아래, `<tbody>` 안의 단일 행에 `colSpan={columns.length}`.
- 배경. `bg-white`.
- 수직 패딩. `pt-[80px] pb-[88px]` (Figma 정확값).
- 가운데 정렬. `flex flex-col items-center gap-[12px]`.
- 아이콘. 40×40 컨테이너, 내부에 `priority_high` (느낌표) SVG. 색상 `admin-gray-5` 톤.
- 텍스트. **"현재 표시할 내용이 없습니다."** Body2 (Regular 16/24), `text-admin-gray-5`.

### 구현 체크리스트
- [ ] `priority_high.svg` 추가 + `Icons.PriorityHigh` 등록.
- [ ] `data-table.tsx` 의 빈 상태 분기 교체. 현재 `"데이터가 없습니다."` 와 `text-muted-foreground` 를 위 스펙으로 바꾼다.
- [ ] 기존 `src/components/shared/no-data.tsx` 의 `FileX` 아이콘 / "데이터가 없습니다." 텍스트도 같은 스펙으로 통일 (또는 deprecate 하고 DataTable 내장으로 흡수).

---

## 3. 드롭다운 Empty State (옵션 0개)

**Figma**. https://www.figma.com/design/RmYvmE5RaSYISPTHVHbcLj/?node-id=661-69659

### 트리거
- 드롭다운 옵션 배열 자체가 0개일 때 → **"항목이 없습니다."**.
- 검색이 있는 드롭다운에서 검색 결과만 0개 → 기존 **"검색 결과가 없습니다."** 유지 (별도 케이스).

두 케이스 텍스트가 다르다. 옵션 자체가 비었는지, 검색으로 필터된 결과가 비었는지 분기한다.

### 시각 스펙
- 드롭다운 패널 내부, 옵션 리스트 자리에 한 줄 표시.
- `flex items-center gap-[8px] p-[12px] rounded-[4px] w-full`.
- 아이콘. 24×24, `Icons.Error` (`error.svg` — 이미 존재) 또는 별도 등록된 exclamation. 색상 `admin-gray-5` 톤.
- 텍스트. Body2 Regular 16/24, `text-admin-gray-5`.

### 구현 체크리스트
- [ ] `MultiSelectDropdown.tsx` 의 옵션 0 케이스 분기 추가 (현재는 `filteredOptions.length === 0` 만 처리).
  ```ts
  if (options.length === 0) → "항목이 없습니다."
  else if (filteredOptions.length === 0) → "검색 결과가 없습니다."
  ```
- [ ] `src/components/ui/select.tsx` (shadcn) 사용처에도 동일 규칙 적용. Radix Select 는 빈 옵션을 직접 처리하지 않으므로 옵션 배열 길이를 호출부에서 체크.

---

## 4. 페이지 이탈 Alert (Leave Confirmation)

**Figma**. https://www.figma.com/design/RmYvmE5RaSYISPTHVHbcLj/?node-id=411-161657

### 트리거
다음 4가지 상황에서 폼이 **dirty 상태** (사용자가 입력/수정했고 아직 저장 안 함) 일 때 노출.
1. 사이드 메뉴(또는 다른 라우트 링크) 클릭으로 페이지 이탈 시도.
2. 페이지 내 '이전' 버튼 클릭.
3. 브라우저 뒤로가기 (popstate).
4. (옵션) 탭 닫기 / 새로고침 — 이건 브라우저 기본 `beforeunload` 로 처리. 커스텀 alert UI 는 표시 못 함.

### 시각 스펙
- 컨테이너. `bg-white border border-admin-gray-2 rounded-admin-md shadow-admin-floating`, 가운데 모달.
- 오버레이. `bg-black/40` (또는 `bg-black/80` shadcn 기본). 클릭 시 닫기는 비활성 (취소 동작과 동일 처리 권장).
- 상단 close. `pt-[8px] px-[8px] flex justify-end` → 24px close 아이콘을 `p-[8px] rounded-[4px]` 안에. 클릭은 취소와 동일.
- 본문. `pb-[20px] px-[20px] flex flex-col items-center gap-[16px]`.
  - 아이콘. 60×60 `Icons.AlertError` (이미 등록되어 있음. 적색 마름모 + !).
  - 텍스트 2줄, `text-admin-primary text-center`, Body2 Regular 16/24.
    - "이 페이지에서 나가시겠습니까?"
    - "페이지를 벗어나면 작성한 내용이 저장되지 않습니다."
- 푸터. `pb-[32px] pt-[20px] px-[20px] flex justify-center gap-[12px]`.
  - **취소** 버튼. Second Button `outlined` 96px (`bg-white border-admin-gray-3 text-admin-primary`).
  - **확인** 버튼. Second Button `filled` 96px (`bg-admin-primary text-white`).

### 구현 방식 (확정)

별도 `leave-alert.tsx` 컴포넌트 없이 기존 `useAlert()` (alert-provider) 재사용으로 결정.

**훅 위치**: `src/hook/useLeaveGuard.ts`

```ts
// 사용 예시 — 모든 등록/수정 뷰에서 동일 패턴 적용
const { isDirty } = form.formState;
const { guardedNavigate } = useLeaveGuard(isDirty);

// 취소 버튼
onClick={() => guardedNavigate(() => router.push('/목록경로'))}
```

**내부 동작**
- `isDirty` 가 true 일 때만 가드 활성화.
- 브라우저 뒤로가기: `window.addEventListener('popstate', ...)` 로 가로채고, 취소 시 `history.pushState` 로 원복.
- 취소 버튼: `guardedNavigate(navigate)` — dirty 상태면 alert 노출 후 확인 시에만 이동.
- alert 문구: title `'페이지에서 나가시겠습니까?'`, description `'작성 중인 내용은 저장되지 않습니다.'`, confirmText `'나가기'`.

### 구현 체크리스트
- [x] `src/hook/useLeaveGuard.ts` 작성 완료.
- [x] 이용약관 등록(`terms/create.tsx`) 적용 완료.
- [x] 이용약관 수정(`terms/edit.tsx`) 적용 완료.
- [x] 개인정보 처리방침 등록(`privacy-policy/create.tsx`) 적용 완료.
- [x] 개인정보 처리방침 수정(`privacy-policy/edit.tsx`) 적용 완료.
- [ ] 이후 추가되는 **모든 등록/수정 뷰**에도 동일 패턴 적용 필수.

### 확인 시 동작
- "확인" 클릭 → 페이지 이탈 진행 (원래 의도한 라우트로 이동).
- "취소" / close / 오버레이 → 모달 닫고 현재 페이지에 머무름.

---

## 5. 디자인 토큰 빠른 참조

상세는 `DESIGN_SYSTEM.md` 참조. 이 문서 4개 패턴에서 자주 쓰는 토큰만 발췌.

| 용도 | 토큰 |
|---|---|
| 본문 텍스트 | `text-admin-primary` (#282843) |
| 보조 텍스트 (empty 안내문구) | `text-admin-gray-5` (#868e96) |
| 입력/카드 테두리 | `border-admin-gray-3` (#dee2e6) |
| 구분선 | `border-admin-gray-2` (#e9ecef) |
| 모달/카드 radius | `rounded-admin-md` (8px) |
| 버튼/셀 radius | `rounded-admin-sm` (4px) |
| 플로팅 그림자 | `shadow-admin-floating` |
| 빈 안내 텍스트 타이포 | `text-admin-body-2` (Regular 16/24) |

---

## 6. 작업 시작 순서

1. `AI_CONTEXT/AI_ONBOARDING.md` 의 읽기 순서에 따라 컨텍스트 로드.
2. 작업이 이 문서 §1~§4 중 하나라도 건드리면 §0 인벤토리에서 표준 컴포넌트 상태 확인.
3. 표준 컴포넌트가 "신규 필요" 면 먼저 만든다. 이미 있으면 import 해서 쓴다.
4. 새 파일 첫 줄에 한국어 한 줄 주석 (`PROJECT_RULES.md` 참조).
5. 완료 전에 `npx tsc --noEmit` + `npm run build` 통과 확인.
6. Storybook 은 사용자가 요청한 경우에만 추가하거나 수정한다.

---

## 7. 확정 정책과 남은 작업

- `DataTable` 내장 empty/error/loading body row를 기본으로 사용한다.
- `NoData` 컴포넌트는 table 외부의 독립 빈 상태가 필요할 때만 사용한다.
- `useLeaveGuard`는 현재 hook 방식을 표준으로 사용한다.
- Leave Alert는 `useAlert()`를 재사용한다.
- 정렬 헤더 표준 컴포넌트와 `OrderUp` 아이콘은 아직 신규 작업이 필요하다.
- Dropdown 옵션 0개 상태는 아직 보강 작업이 필요하다.

---

## 8. 구현 완료 전 자가 검증 체크리스트

새 도메인 페이지(목록/등록/상세/수정)를 구현한 뒤, PR 올리기 전에 아래 항목을 순서대로 점검한다.
한 항목이라도 누락되면 구현 완료가 아니다.

### 8-1. 공통 컴포넌트 중복 구현 방지

구현을 시작하기 전, 그리고 완료 후 다시 한번 아래를 확인한다.

| 확인 항목 | 올바른 사용 | 잘못된 사용 |
|---|---|---|
| 페이지네이션 | `src/components/shared/Pagination.tsx` | 직접 구현한 이전/다음 버튼 |
| 전역 confirm/alert | `useAlert()` from `alert-provider.tsx` | `window.confirm`, 별도 상태 모달 |
| 페이지 이탈 가드 | `useLeaveGuard(isDirty)` from `src/hook/useLeaveGuard.ts` + `guardedNavigate` | 직접 `router.push`, dirty 체크 없이 이동 |
| 삭제 불가 등 커스텀 모달 | Radix `Dialog` (`src/components/ui/dialog.tsx`) | 새 모달 UI 직접 제작 |
| 날짜 입력 | `DateTimeInput` (`src/components/shared/DateTimeInput.tsx`) | `<input type="date">` 직접 사용 |
| 에디터 | `src/components/shared/editor/index.tsx` (Tiptap) | 새 에디터 라이브러리 설치 |
| 폼 필드 | `FormFields` / `FormInput` from `shared/form-fields` | Label+Input 직접 조합 |
| 빵 부스러기 | `BreadCrumb` (`src/components/shared/bread-crumb.tsx`) | 직접 경로 텍스트 하드코딩 |
| 테이블 | `DataTable` (`src/components/shared/data-table.tsx`) | HTML `<table>` 직접 구현 |

### 8-2. 아이콘 색상 점검

`COMPONENT_RULES.md` — "테이블 비고 셀 아이콘 색상 규칙" 참조.

- [ ] Edit 아이콘 버튼: `text-admin-primary`
- [ ] Delete 아이콘 버튼: `text-admin-point`
- [ ] `text-*`는 Icon이 아닌 **Button** 에 부여했는가.
- [ ] 비고 셀 컨테이너에 `justify-center` 가 있는가.

### 8-3. 디자인 토큰 점검

- [ ] 카드/패널 border: `border-admin-gray-2` (하드코딩 `#E9ECEF` 금지)
- [ ] 페이지 배경: `bg-admin-gray-1`
- [ ] 레이블 셀 배경: `bg-admin-gray-1`, width `w-[206px]`
- [ ] 카드 radius: `rounded-admin-md` (8px)
- [ ] 인풋/버튼/셀 radius: `rounded-admin-sm` (4px)
- [ ] 검색 카드 내 조건행과 버튼 영역 사이에 `border-t border-admin-gray-2` 구분선이 있는가.

### 8-4. 레이아웃 점검

- [ ] `BreadCrumb` 가 한국어 레이블로 표시되는가 (`bread-crumb.tsx` 매핑 테이블 확인).
- [ ] 페이지 제목이 `text-[24px] font-semibold` 로 표시되는가.
- [ ] 하단 버튼이 `justify-end gap-4` 우측 정렬, `h-12 w-[132px]` 크기인가.
- [ ] 등록 버튼이 `bg-admin-point`, 저장·확인 계열이 `bg-admin-point`, 목록·취소가 `variant="outline-admin"` 인가.

### 8-5. 빌드/타입 검증

- [ ] `npx tsc --noEmit` 통과
- [ ] `npm run build` 통과
- [ ] `npm run lint` 에서 새로 작성한 파일에 에러 없음

---

## 9. 변경 이력

- 2026-05-19. 초안 (Claude, Figma MCP 분석 기반). 4개 패턴 + 디자인 토큰 + 작업 순서.
- 2026-05-21. §8 구현 완료 전 자가 검증 체크리스트 추가 (아이콘 색상 누락·공통 UI 미사용 방지).
