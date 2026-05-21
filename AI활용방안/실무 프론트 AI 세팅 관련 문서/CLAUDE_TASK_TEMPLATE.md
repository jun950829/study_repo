# CLAUDE_TASK_TEMPLATE — Claude 작업 요청 템플릿

Claude에게 Figma 기반 화면 구현을 맡길 때 아래 템플릿을 사용한다.
작업마다 Figma URL, 구현 범위, 라우트, 데이터 조건만 바꾼다.

---

## 기본 템플릿

```text
# 작업
현재 프로젝트의 AI_CONTEXT 문서를 기준으로 Figma 디자인을 구현해줘.
코드 작성 전 Figma MCP로 디자인을 분석하고, 기존 공통 UI와 admin token을 우선 사용해.

# 프로젝트 루트
/Users/momenti/Desktop/projects/moda/moda-frontend-admin

# 반드시 먼저 읽을 문서
전체 프로젝트를 전부 스캔하지 말고 아래 문서부터 읽어.

1. AI_CONTEXT/AI_ONBOARDING.md
2. AI_CONTEXT/FIGMA_MCP_WORKFLOW.md
3. AI_CONTEXT/CLAUDE_TASK_TEMPLATE.md
4. AI_CONTEXT/DIRECTORY_MAP.md
5. AI_CONTEXT/DESIGN_SYSTEM.md
6. AI_CONTEXT/COMPONENT_RULES.md
7. AI_CONTEXT/UI_DEFAULTS.md
8. AI_CONTEXT/COMMON_FLOWS.md
9. AI_CONTEXT/FORM_RULES.md
10. AI_CONTEXT/QUERY_RULES.md
11. AI_CONTEXT/RENDERING_RULES.md
12. 작업 대상 feature 문서가 있으면 해당 문서

# Figma 참조
- 화면명:
- URL:
- fileKey:
- nodeId:

# 구현 범위
- 목록:
- 등록:
- 상세:
- 수정:
- 모달:
- 버튼/화면 플로우:

# 데이터 조건
- API가 이미 있으면 기존 query/mutation/agent 패턴을 사용.
- API가 없으면 feature 내부 mock data로 화면이 보이게 구성.
- mock/TODO는 최소화하고 완료 보고에 명시.

# 공통 UI 사용 원칙
- Button, Input, Select, Checkbox, Badge, Dialog, AlertDialog, DropdownMenu, Table은 src/components/ui 우선 사용.
- Header, Sidebar, BreadCrumb, DataTable, DetailPanel, DateTimeInput, Editor, Pagination, useLeaveGuard는 src/components/shared 또는 src/hook 기존 구현 우선 사용.
- 동일 기능의 컴포넌트를 새로 만들지 말고 기존 컴포넌트 props/조합으로 해결.
- 기존 공통 UI로 불가능한 도메인 조합만 src/components/views/{feature} 내부에 만든다.

# Figma MCP 작업 방식
- 구현 전 get_design_context, get_variable_defs, get_metadata를 호출해서 분석.
- Figma generated React code는 복붙하지 않는다.
- Figma remote asset URL은 사용하지 않는다.
- 색상, 타이포, radius, shadow는 AI_CONTEXT/DESIGN_SYSTEM.md의 admin token으로 매핑.
- spacing은 Figma 값을 참고하되 기존 admin page pattern과 공통 UI spacing을 우선.

# 구현 규칙
- Next.js App Router 패턴 유지.
- page.tsx는 Server Component로 두고 View만 import.
- 실제 상태/이벤트/query/form은 src/components/views/{feature}의 client component에서 처리.
- 새 파일 첫 줄에는 한국어 한 줄 주석을 추가.
- Form은 react-hook-form + zodResolver + Zod 사용.
- API는 src/utils/fetch.ts의 agent() 사용.
- 서버 상태는 TanStack Query 사용.
- 알림/확인은 useAlert() 사용.
- 등록/수정 화면의 취소, 뒤로가기, 목록 이동에는 dirty 상태일 때 useLeaveGuard를 적용.

# 검증
작업 완료 후 아래만 실행.
- npx tsc --noEmit
- npm run build

# 완료 보고 형식
완료 요약:
- 

변경 파일:
- 

검증:
- npx tsc --noEmit:
- npm run build:

Figma와 다르게 처리한 부분:
-

mock/TODO:
-
```

---

## 리스트 화면 추가 지시

리스트 화면이 포함되면 기본 템플릿에 아래를 추가한다.

```text
# 리스트/table 규칙
- table column은 Figma 순서를 따른다.
- 데이터가 없어도 table과 table header는 항상 보인다.
- loading/error/empty 상태에서도 table body 영역 안에서 안내 UI를 보여준다.
- empty/error UI는 AI_CONTEXT/UI_DEFAULTS.md의 table empty 규칙을 따른다.
- 페이지네이션이 있으면 기존 DataTable + Pagination 흐름을 사용한다.
- 비고/관리 column의 icon button은 Button variant="icon-action"과 Icons registry를 사용한다.
```

---

## 등록/수정 화면 추가 지시

등록 또는 수정 화면이 포함되면 기본 템플릿에 아래를 추가한다.

```text
# 등록/수정 form 규칙
- Figma의 label/value row 구조를 우선 확인한다.
- 공통 FormFields, FormInput, DateTimeInput, Select, Textarea, Editor를 우선 사용한다.
- 게시일자/노출일자처럼 날짜 제한이 있으면 오늘 포함 이후 날짜만 선택 가능하게 처리한다.
- 취소/뒤로가기/목록 버튼은 dirty 상태에서 useLeaveGuard로 이탈 confirm을 띄운다.
- 저장 성공/실패 안내는 useAlert()로 처리한다.
```

---

## 상세 화면 추가 지시

상세 화면이 포함되면 기본 템플릿에 아래를 추가한다.

```text
# 상세 화면 규칙
- read-only 데이터는 DetailPanel 패턴을 우선 사용한다.
- HTML/editor content는 read-only preview로 렌더링한다.
- 하단 버튼은 Figma 정렬을 따르되 기존 Button variant와 admin token을 사용한다.
- 수정 버튼은 해당 edit route로 이동한다.
```

---

## 모달 추가 지시

모달이 포함되면 기본 템플릿에 아래를 추가한다.

```text
# 모달 규칙
- 단순 확인/취소 모달은 useAlert() 또는 AlertDialog를 우선 사용한다.
- form이 들어가는 모달은 Dialog를 사용한다.
- Figma의 title, description, button text를 그대로 사용한다.
- 버튼 색상과 정렬은 기존 Dialog/AlertDialog 정책을 따른다.
```
