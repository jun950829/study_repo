# DESIGN_SYSTEM — 디자인 토큰 & 스타일 규칙

## 원칙

- 모든 색상·타이포·radius·shadow는 **admin-* 토큰** 사용.
- shadcn 기본 토큰(`bg-primary`, `text-foreground` 등) 사용 금지.
- 아직 admin 토큰이 없는 항목은 Tailwind 기본값 임시 사용 후 `// TODO` 주석.
- 토큰은 `src/app/globals.css`의 `@theme` 블록에 정의됨.

---

## 색상 토큰

| 토큰 | HEX | 용도 |
|---|---|---|
| `admin-primary` | `#282843` | 텍스트·아이콘·사이드바 기본색 |
| `admin-primary-bg` | `#f4f5fb` | 테이블 짝수 행 배경 (zebra stripe) |
| `admin-gray-1` | `#f8f9fa` | 테이블 헤더·상세 레이블 셀·페이지 배경 |
| `admin-gray-2` | `#e9ecef` | 행 구분선·패널·드롭다운 테두리 |
| `admin-gray-3` | `#dee2e6` | 카드 테두리·인풋 기본 테두리·아이콘 버튼 테두리 |
| `admin-gray-4` | `#ced4da` | Placeholder 텍스트 |
| `admin-gray-5` | `#868e96` | 보조 텍스트·단위 표기·subText |
| `admin-gray-6` | `#495057` | 강조 캡션·라벨 본문 |
| `admin-gray-7` | `#1e1f21` | 섹션 헤딩·본문 강조 |
| `admin-point` | `#be3848` | 필수 asterisk·Toggle on·Tab selected |
| `admin-point-hover` | `#982d3a` | 포인트 hover 상태 |
| `admin-point-disabled` | `#efcfd3` | 포인트 disabled 상태 |
| `admin-error` | `#dc2626` | 에러 상태 테두리·텍스트 |
| `admin-correct` | `#34c978` | 정상/완료 상태 (ValidationInput correct) |

사용 예시.
```tsx
<div className="bg-admin-primary-bg text-admin-primary border-admin-gray-3" />
<span className="text-admin-point">*</span>
<p className="text-admin-error text-admin-caption-2">에러 메시지</p>
```

---

## 타이포그래피 토큰

Pretendard 가변 폰트. letter-spacing 전체 0.

| 토큰 | 크기 | 행높이 | 굵기 | 용도 |
|---|---|---|---|---|
| `text-admin-page-title` | 60px | — | 700 | 페이지 대제목 |
| `text-admin-section-title` | 40px | — | 700 | 섹션 제목 |
| `text-admin-heading` | 16px | 24px | 600 | 소제목·테이블 헤더 |
| `text-admin-body-1` | 16px | 24px | 500 | 중요 본문 |
| `text-admin-body-2` | 16px | 24px | 400 | 일반 본문·인풋 값 |
| `text-admin-caption-1` | 14px | 20px | 500 | 라벨·캡션 강조 |
| `text-admin-caption-2` | 14px | 20px | 400 | 보조 캡션·에러 메시지 |

---

## Radius 토큰

| 토큰 | 값 | 용도 |
|---|---|---|
| `rounded-admin-sm` | 4px | 인풋·버튼·셀렉트·뱃지 |
| `rounded-admin-md` | 8px | 카드·패널·모달 |
| `rounded-admin-lg` | 12px | 큰 컨테이너 |
| `rounded-admin-pill` | 24px | 필 버튼·칩 |
| `rounded-admin-page` | 40px | 페이지 레벨 컨테이너 |

---

## Shadow 토큰

| 토큰 | 값 | 용도 |
|---|---|---|
| `shadow-admin-floating` | `1px 2px 12px rgba(0,0,0,0.04)` | 드롭다운·팝오버·플로팅 패널 |

---

## 아이콘 레지스트리

파일: `src/components/ui/icon.tsx`

SVG 파일을 import해 `Icons` 네임스페이스 객체로 묶어 제공. fill 값이 `#282843`인 SVG는 `currentColor`로 자동 변환됨(`next.config.mjs` SVGR replaceAttrValues 설정).

**예외**: `Icons.Profile`, `Icons.Excel`은 멀티 컬러 SVG — replaceAttrValues 미적용. `className`의 `text-*`로 색상 제어 불가.

```tsx
import { Icons } from '@/components/ui/icon';

<Icons.Delete className="h-6 w-6 text-admin-primary" />
// className으로 크기(h-*, w-*)와 색상(text-*) 제어.
// Profile·Excel은 text-* 색상 무효.
```

등록된 아이콘 목록 (전체).

| 그룹 | 아이콘 이름 |
|---|---|
| 방향·화살표 | `ArrowDown`, `ArrowUp`, `ChevronLeft`, `ChevronRight`, `ChevronDoubleLeft`, `ChevronDoubleRight`, `OrderDown`, `ArrowDropDown` |
| 액션 | `Add`, `Edit`, `Delete`, `Close`, `Cancel`, `Search`, `Download`, `DragHandle`, `Dash`, `ViewMore`, `StickyNote` |
| 상태·피드백 | `Check`, `CheckCircle`, `AlertConfirm`, `AlertError`, `Notice`, `Error`, `Help`, `Info`, `Asterisk` |
| 표시·입력 | `VisibilityOn`, `VisibilityOff`, `Date`, `Time` |
| 폼 컨트롤 | `RadioOn`, `RadioOff`, `CheckboxOn`, `CheckboxOff`, `ListBulletOn`, `ListBulletOff`, `ListSquareOn`, `ListSquareOff` |
| 멀티 컬러 (fill 고정) | `Profile`, `Excel` |

위 목록에 없는 아이콘은 `lucide-react`를 사용.

```tsx
import { Plus, MoreHorizontal, Users } from 'lucide-react';
```

---

## 컴포넌트별 주요 토큰 매핑

| 컴포넌트 | 핵심 토큰 |
|---|---|
| Input (기본) | `border-admin-gray-3 focus:border-admin-primary h-[44px] rounded-admin-sm` |
| Input (에러) | `border-admin-error focus:border-admin-error` |
| Input (비활성) | `bg-admin-gray-1 text-admin-gray-5` |
| Select 트리거 | Input과 동일 (h-[44px], admin 토큰) |
| Select 드롭다운 | `max-h-[250px] shadow-admin-floating rounded-admin-sm` |
| Button navy | `bg-admin-primary text-white hover:bg-admin-primary/90` |
| Button icon-action | `bg-white border-admin-gray-3 p-[6px] rounded-[4px] hover:bg-admin-gray-1` |
| Badge success | `bg-admin-correct/10 text-admin-correct border-admin-correct` |
| Label (required) | `*` in `text-admin-point` |
| Label subText | `text-admin-caption-2 text-admin-gray-5` |
| DetailPanel 레이블 셀 | `w-[206px] bg-admin-gray-1` |
| DataTable 짝수 행 | `bg-admin-primary-bg` |
| DataTable 헤더 | `bg-admin-gray-1` |
| 사이드바 배경 | `bg-admin-primary` |
| 페이지 배경 | `bg-admin-gray-1` |
