# COMPONENT_RULES — 컴포넌트 작성 규칙

## 레이어 구분

| 폴더 | 용도 | 예시 |
|---|---|---|
| `src/components/ui/` | Radix 기반 원자 컴포넌트. admin 토큰 커스텀 적용. | Button, Input, Label, Select |
| `src/components/shared/` | admin 도메인 복합 컴포넌트. ui/ 컴포넌트를 조합. | DataTable, FormInput, MultiSelectDropdown |
| `src/components/side-menu/` | 사이드바 전용. SideMenu/MenuGroup/SideMenuItem. | — |
| `src/components/table-columns/{feature}/` | TanStack Table ColumnDef. feature별로 파일 분리. | `table-columns/user/index.tsx` |
| `src/components/views/{feature}/` | 페이지 뷰. `page.tsx`가 import하는 클라이언트 컴포넌트. | `views/user/index.tsx` |

---

## ui/ 컴포넌트 인벤토리

### Button
```tsx
<Button variant="navy">저장</Button>
// variants: default | destructive | outline | secondary | ghost | link | navy | icon-action | outline-admin
// navy: admin-primary 배경의 주요 액션 버튼
// icon-action: 테이블 액션 셀 아이콘 버튼 (bg-white, border-admin-gray-3, p-[6px], rounded-[4px], hover:bg-admin-gray-1)
// outline-admin: admin-gray-3 border + white background + admin-primary text. 목록/취소 계열 버튼
```

### Input
```tsx
<Input placeholder="입력" error />
// error?: boolean — border-admin-error, focus 유지
// h-[44px], rounded-admin-sm, border-admin-gray-3 기본
// disabled: bg-admin-gray-1, text-admin-gray-5
```

### Label
```tsx
<Label required subText="(최대 20자)">항목명</Label>
// required?: boolean — 오른쪽에 * (admin-point)
// subText?: string — 인라인으로 이어서 표시 (admin-caption-2, text-admin-gray-5)
```

### Select
```tsx
<Select>
  <SelectTrigger error>     // error?: boolean
    <SelectValue placeholder="선택" />
  </SelectTrigger>
  <SelectContent>           // max-h 250px (5.5행)
    <SelectItem value="v">항목</SelectItem>
  </SelectContent>
</Select>
```

### Badge
```tsx
<Badge variant="success">활성</Badge>
// variants: default | secondary | outline | success
```

---

## shared/ 컴포넌트 인벤토리

### FormInput (독립형 인풋 래퍼)
```tsx
<FormInput
  label="항목명"
  required
  subText="(최대 20자)"
  value={v}
  onChange={(e) => setV(e.target.value)}
  maxLength={20}
  showCount
  errorMessage="올바른 값을 입력해 주세요."
  helperText="영문·숫자 조합"
  wrapperClassName="w-[384px]"
/>
// RHF 없이 독립 사용. label/error/helperText/count 통합.
```

### FormFields (RHF 연동)
```tsx
import { FormFields, FormFieldType } from '@/components/shared/form-fields';

<FormFields
  fieldType={FormFieldType.INPUT}   // INPUT | PASSWORD | TEXTAREA | SELECT | NUMBER
  control={form.control}
  name="email"
  label="이메일"
  placeholder="이메일을 입력하세요"
  options={[{ label: '관리자', value: 'ADMIN' }]}  // SELECT만 사용
/>
// react-hook-form의 FormField/FormItem/FormMessage 연동.
```

### MultiSelectDropdown
```tsx
<MultiSelectDropdown
  sort="multi"      // single | multi | searchable | all
  options={[{ label: '항목1', value: 'v1' }]}
  value={selected}  // string[]
  onChange={setSelected}
  label="카테고리"
  required
  errorMessage="선택해 주세요."
  disabled
/>
// sort="searchable": 드롭다운 내 검색 인풋 포함
// sort="all": 전체 선택 체크박스 포함
// 리스트 max-h: 242px (5.5행 × 44px)
```

### DataTable
```tsx
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
// TanStack Table 기반. zebra stripe (admin-primary-bg 짝수행).
// 서버 페이지네이션: manualPagination=true + onPaginationChange.
```

### DetailPanel
```tsx
<DetailPanel
  rows={[
    { label: '이름', value: '홍길동', required: true },
    { label: '상태', value: <Badge>활성</Badge> },
  ]}
/>
// 상세 화면 key-value 테이블. 레이블 셀 w-[206px], bg-admin-gray-1.
// required=true: 레이블 앞에 asterisk 아이콘 (admin-point).

// 값 셀 안에서 중첩 정보 표시:
<DetailItem title="주문 1" fields={[{ label: '수량', value: '3개' }]} />
```

### PasswordInput
```tsx
<PasswordInput
  label="비밀번호"
  required
  errorMessage="비밀번호가 일치하지 않습니다."
  wrapperClassName="w-[384px]"
/>
// Eye/EyeOff 토글. type="password" ↔ "text" 전환.
```

### ValidationInput
```tsx
<ValidationInput
  status="correct"       // enabled | correct | error
  label="인증번호"
  errorMessage="인증번호가 올바르지 않습니다."
/>
// correct: border-admin-correct + CheckCircle 아이콘
// error: border-admin-error + XCircle 아이콘 + errorMessage
```

### DateTimeInput
```tsx
<DateTimeInput
  mode="date"     // date | time | datetime | date-period | time-period | datetime-period
  label="날짜"
  value={v}
  onChange={setV}
  endValue={endV}       // period 모드에서 종료값
  onEndChange={setEndV}
/>
// period 모드: w-[480px], 시작~종료 두 인풋 + ~ 구분자
```

### FileAttachmentInput
```tsx
<FileAttachmentInput
  label="첨부파일"
  required
  value={file}
  onChange={setFile}   // (file: File | null) => void
  errorMessage="파일을 첨부해 주세요."
  accept=".pdf,.docx"
/>
```

### ImageUploadBox
```tsx
<ImageUploadBox
  label="대표 이미지"
  value={previewUrl}
  onChange={(file) => { ... }}  // (file: File | null) => void
  status="error"  // enabled | hover | uploaded | disabled | disabled_uploaded | error
  errorMessage="파일 업로드에 실패했습니다."
/>
// 클릭 영역: 200×200px. 아이콘: 32px (hit area 48px).
```

### PeriodFilter / PeriodPicker
```tsx
// 대시보드 차트 전용
<PeriodFilter value={period} onChange={setPeriod} />
// collapsed(w-92px 버튼) / expanded(리스트) 토글

<PeriodPicker startDate={start} endDate={end} onStartChange={setStart} onEndChange={setEnd} />
// 항상 filled 상태. w-[269px]. 대시보드 헤더 영역용.
```

### BreadCrumb
```tsx
<BreadCrumb />
// pathname을 '/'로 split → 세그먼트별 Link 생성. props 없음.
// 페이지 컴포넌트 최상단에 배치.
```

---

## 아이콘 사용

### SVG 아이콘 (Icons.*)
```tsx
import { Icons } from '@/components/ui/icon';

<Icons.Delete className="h-6 w-6 text-admin-primary" />
// className으로 크기(h-*, w-*) + 색상(text-*) 제어.
// fill="#282843" → currentColor 자동 변환됨.
```

주요 아이콘: `ArrowDown/Up`, `ChevronLeft/Right`, `ChevronDoubleLeft/Right`,
`Add`, `Edit`, `Delete`, `Close`, `Search`, `Download`, `DragHandle`,
`Check`, `CheckCircle`, `AlertConfirm`, `AlertError`, `VisibilityOn/Off`,
`Date`, `Time`, `RadioOn/Off`, `CheckboxOn/Off`, `Profile`, `Excel`

### lucide-react
```tsx
import { Plus, MoreHorizontal, Users } from 'lucide-react';
// 위 Icons.*에 없는 아이콘은 lucide-react 사용.
```

### 테이블 비고(actions) 셀 — 아이콘 색상 규칙

테이블 행 우측 비고 칸의 icon-action 버튼은 액션 종류에 따라 색상이 고정된다.

| 액션 | 아이콘 | `text-*` | 색상 |
|---|---|---|---|
| 수정 / 편집 | `Icons.Edit` | `text-admin-primary` | #282843 (Navy) |
| 삭제 | `Icons.Delete` | `text-admin-point` | #BE3848 (Point/Red) |
| 다운로드 | `Icons.Download` | `text-admin-primary` | #282843 |
| 보기 / 상세 | `Icons.ViewMore` | `text-admin-primary` | #282843 |

**색상은 반드시 `Button`에 부여한다.**
`asChild` + `Link` 조합에서 Icon 컴포넌트 자체에 달면 color 상속이 끊길 수 있다.

```tsx
// 올바른 패턴
<Button variant="icon-action" size="icon" className="h-8 w-8 text-admin-primary" asChild>
  <Link href="..."><Icons.Edit className="h-4 w-4" /></Link>
</Button>
<Button variant="icon-action" size="icon" className="h-8 w-8 text-admin-point" onClick={onDelete}>
  <Icons.Delete className="h-4 w-4" />
</Button>

// 잘못된 패턴 — Icon에 직접 text-* 부여
<Button variant="icon-action" asChild>
  <Link href="..."><Icons.Edit className="h-4 w-4 text-admin-primary" /></Link>  {/* ❌ */}
</Button>
```

비고 셀 컨테이너는 반드시 `justify-center` 를 추가해 중앙 정렬한다.
`text-center` 는 inline 요소에만 작동하며 `div` 에는 효과가 없다.

```tsx
<div className="flex items-center justify-center gap-2">
  {/* icon buttons */}
</div>
```

---

## SideMenuItem (내부 구현 참고)

CVA로 `sort × status` 조합 스타일 처리.

```tsx
sort: 'first'   // 1-depth: 아이콘 + 텍스트 + 아코디언 화살표
sort: 'second'  // 2-depth: ListBullet 아이콘 + 텍스트
sort: 'third'   // 3-depth: ListSquare 아이콘 + 텍스트 (현재 미사용)

status: 'enabled'   // text-admin-gray-5, hover → primary
status: 'selected'  // text-admin-primary, font-medium
```

`href` prop 있으면 `<Link>`, 없으면 `<button>` 렌더링.

---

## Storybook

- 파일 위치: 컴포넌트 파일과 동일 폴더, 같은 이름에 `.stories.tsx` suffix.
- **예외**: `side-menu/` 컴포넌트의 스토리는 `src/stories/side-menu/`에 위치 (컴포넌트 파일과 분리됨).
- `title` 형식: `UI/ComponentName` 또는 `Shared/ComponentName`.
- import: `@storybook/nextjs-vite`.
- 상태가 필요한 스토리는 `render` 함수 내부에 `useState` 사용.
