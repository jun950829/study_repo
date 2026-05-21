# VIBE_CODING_PROCESS — AI 바이브 코딩 운영 프로세스

이 문서는 Moda Admin 프로젝트에서 Claude, Codex, Figma MCP를 사용해 화면을 빠르게 만들되 구조와 품질을 유지하기 위한 작업 프로세스다.

목표.

- AI가 프로젝트 전체를 불필요하게 스캔하지 않게 한다.
- Figma 디자인을 공통 UI와 admin token으로 정확히 변환한다.
- 도메인별 파일 구조와 구현 방식이 계속 같게 유지된다.
- mock 기반 구현에서 API 연결로 자연스럽게 넘어갈 수 있게 한다.
- 매 작업 후 빌드 가능한 상태를 유지한다.

---

## 1. 기본 원칙

AI에게 바로 “구현해줘”라고 던지지 않는다.

항상 아래 흐름을 따른다.

1. 작업 범위 정의.
2. 참조 Figma node 정리.
3. AI_CONTEXT 문서 읽기.
4. Figma MCP 분석.
5. 구현 계획 산출.
6. 코드 구현.
7. 타입/빌드 검증.
8. 결과 보고.
9. 필요한 경우 AI_CONTEXT 갱신.

이 흐름을 생략하면 도메인별 구조가 흔들리고, 공통 UI 중복 생성이 늘어난다.

---

## 2. 작업 유형별 프로세스

### 2-1. Figma 기반 신규 화면 구현

사용 상황.

- 목록/등록/상세/수정 페이지 구현.
- 모달 구현.
- Figma 시안과 동일한 화면 제작.

읽을 문서.

1. `AI_CONTEXT/AI_ONBOARDING.md`
2. `AI_CONTEXT/FIGMA_MCP_WORKFLOW.md`
3. `AI_CONTEXT/CLAUDE_TASK_TEMPLATE.md`
4. `AI_CONTEXT/DIRECTORY_MAP.md`
5. `AI_CONTEXT/DESIGN_SYSTEM.md`
6. `AI_CONTEXT/COMPONENT_RULES.md`
7. `AI_CONTEXT/UI_DEFAULTS.md`
8. `AI_CONTEXT/COMMON_FLOWS.md`
9. `AI_CONTEXT/FORM_RULES.md`
10. `AI_CONTEXT/QUERY_RULES.md`
11. `AI_CONTEXT/RENDERING_RULES.md`

필수 MCP 분석.

- `get_design_context(fileKey, nodeId)`
- `get_variable_defs(fileKey, nodeId)`
- `get_metadata(fileKey, nodeId)`

구현 전 산출물.

- 레이아웃 구조.
- 디자인 토큰 매핑.
- 사용 컴포넌트 목록.
- 공통 UI 매핑.
- 구현 계획.

금지.

- Figma generated React code 복붙.
- Figma remote asset URL 사용.
- Header/Sidebar 중복 구현.
- 기존 공통 UI와 같은 기능의 신규 컴포넌트 생성.
- 요청되지 않은 Storybook/test 파일 생성.

---

### 2-2. 기존 도메인 확장

사용 상황.

- 이미 있는 도메인에 새 화면 추가.
- table column 변경.
- form field 추가.
- modal flow 추가.

진행 순서.

1. 해당 route 확인.
2. `src/components/views/{domain}` 확인.
3. `src/components/table-columns/{domain}` 확인.
4. `src/features/{domain}/types|schemas|mocks` 확인.
5. `src/hook/{domain}` 확인.
6. 기존 패턴에 맞춰 최소 수정.

원칙.

- 기존 도메인의 파일 배치를 우선 따른다.
- 새 타입은 `src/features/{domain}/types`에 둔다.
- 새 schema는 `src/features/{domain}/schemas`에 둔다.
- mock은 `src/features/{domain}/mocks`에 둔다.
- View는 mock인지 API인지 알지 못하게 한다.
- hook이 API/mock 경계를 담당한다.

---

### 2-3. 공통 UI 추가 또는 수정

사용 상황.

- 같은 UI가 3회 이상 반복된다.
- Figma 공통 컴포넌트와 1:1로 매핑되는 UI가 없다.
- 기존 공통 UI props 확장으로 해결 가능한 경우.

진행 순서.

1. `COMPONENT_RULES.md` 확인.
2. `UI_DEFAULTS.md` 확인.
3. 기존 `src/components/ui`와 `src/components/shared` 검색.
4. 기존 컴포넌트 확장으로 가능한지 판단.
5. 불가능할 때만 신규 공통 컴포넌트 생성.
6. Storybook은 사용자가 요청한 경우에만 추가.
7. 문서에 사용 기준 추가.

분류 기준.

| 위치 | 용도 |
|---|---|
| `src/components/ui` | Radix 기반 primitive, 버튼, 인풋, 셀렉트, 테이블 wrapper |
| `src/components/shared` | 관리자 도메인에서 반복되는 조합 UI |
| `src/components/views/{domain}` | 특정 도메인에서만 쓰는 조합 UI |
| `src/features/{domain}/components` | 도메인 내부에서 여러 View가 공유하는 조합 UI |

---

### 2-4. 리팩토링

사용 상황.

- 도메인 구조 통일.
- mock 위치 정리.
- page.tsx 얇게 만들기.
- 공통 UI 중복 제거.

읽을 문서.

1. `AI_CONTEXT/PROJECT_HEALTH_REPORT.md`
2. `AI_CONTEXT/CLAUDE_REFACTOR_REQUEST.md`
3. `AI_CONTEXT/COMMON_FLOWS.md`
4. `AI_CONTEXT/DIRECTORY_MAP.md`

원칙.

- 한 번에 전체 도메인을 고치지 않는다.
- 한 도메인씩 정리한다.
- 기능 동작을 바꾸지 않는다.
- mock을 제거하지 말고 위치만 정리한다.
- 각 단계마다 `npx tsc --noEmit`과 `npm run build`를 실행한다.

---

## 3. Claude에게 작업을 맡기는 방식

Claude에게 줄 프롬프트는 항상 아래 요소를 포함한다.

- 작업명.
- 프로젝트 루트.
- 먼저 읽을 AI_CONTEXT 문서.
- Figma URL, fileKey, nodeId.
- 구현 범위.
- 라우트 기준.
- 공통 UI 사용 원칙.
- 데이터/API/mock 기준.
- 금지 사항.
- 검증 명령.
- 완료 보고 형식.

기본 템플릿은 `AI_CONTEXT/CLAUDE_TASK_TEMPLATE.md`를 사용한다.

리팩토링 요청은 `AI_CONTEXT/CLAUDE_REFACTOR_REQUEST.md`를 사용한다.

---

## 4. Codex에게 맡기는 방식

Codex는 아래 작업에 사용한다.

- 프로젝트 구조 진단.
- AI_CONTEXT 문서 생성/정리.
- Claude에게 전달할 프롬프트 작성.
- Figma node 분석 결과를 구현 계획으로 변환.
- 빌드/타입/린트 결과 확인.
- 코드 리뷰와 위험 요소 점검.

Codex에게 맡길 때는 결과물을 명확히 지정한다.

예시.

```text
이 Figma URL들을 보고 Claude에게 전달할 구현 프롬프트를 작성해줘.
AI_CONTEXT 문서를 기준으로 공통 UI를 적극 사용하도록 해줘.
```

```text
현재 도메인 구조가 AI 친화적인지 점검하고,
개선점을 AI_CONTEXT 문서로 만들어줘.
```

---

## 5. Figma MCP 사용 기준

Figma URL이 있는 작업은 MCP 분석을 기본값으로 둔다.

필수 확인 항목.

- frame 크기.
- 레이아웃 방향.
- spacing.
- typography.
- color.
- radius.
- shadow.
- component instance.
- 반복 row/card/table pattern.
- modal/dialog state.
- disabled/error/empty state.

주의.

- Figma의 절대 좌표를 그대로 페이지에 하드코딩하지 않는다.
- admin layout의 `main` padding과 page flow를 우선한다.
- Figma 값은 `DESIGN_SYSTEM.md`의 admin token으로 매핑한다.
- Figma screenshot은 시각 기준이고, 코드는 프로젝트 공통 UI 기준으로 작성한다.

---

## 6. 도메인 표준 구조

새 도메인은 아래 구조를 기본으로 한다.

```text
src/app/(auth)/{route}/
  page.tsx
  create/page.tsx
  [id]/page.tsx
  [id]/edit/page.tsx

src/components/views/{domain}/
  index.tsx
  create.tsx
  detail.tsx
  edit.tsx

src/components/table-columns/{domain}/
  index.tsx

src/features/{domain}/
  types/
  schemas/
  mocks/
  constants/
  components/

src/hook/{domain}/
  useGet{Domain}List.ts
  useGet{Domain}Detail.ts
  usePost{Domain}.ts
  useUpdate{Domain}.ts
  useDelete{Domain}.ts
```

필요한 파일만 만든다.
빈 폴더는 만들지 않는다.

---

## 7. 파일 책임 기준

| 파일/폴더 | 책임 |
|---|---|
| `app/**/page.tsx` | Server Component 진입점. View import만 담당 |
| `components/views/{domain}` | 화면 상태, form, router, query 조합 |
| `components/table-columns/{domain}` | TanStack Table column 정의 |
| `features/{domain}/types` | 도메인 타입 |
| `features/{domain}/schemas` | 도메인 Zod schema |
| `features/{domain}/mocks` | API 미연동 mock data와 mock function |
| `features/{domain}/constants` | 도메인 전용 옵션 |
| `features/{domain}/components` | 도메인 내부 공통 조합 UI |
| `hook/{domain}` | API/mock 호출 경계, React Query |
| `constants/options.ts` | 여러 도메인에서 쓰는 공통 옵션 |
| `constants/queryKeys.ts` | React Query key |
| `components/ui` | primitive UI |
| `components/shared` | 관리자 공통 조합 UI |

---

## 8. 구현 검증 기준

코드 수정이 있으면 아래를 실행한다.

```bash
npx tsc --noEmit
npm run build
```

lint까지 필요한 작업이면 아래도 실행한다.

```bash
npm run lint
```

검증 결과는 완료 보고에 반드시 포함한다.

---

## 9. 완료 보고 형식

AI는 작업 완료 시 아래 형식으로 보고한다.

```text
완료 요약:
-

변경 파일:
-

검증:
- npx tsc --noEmit:
- npm run build:
- npm run lint:

Figma와 다르게 처리한 부분:
-

mock/TODO:
-

남은 개선점:
-
```

---

## 10. 작업 후 문서 갱신 기준

아래 상황이면 AI_CONTEXT를 갱신한다.

- 새 공통 UI가 추가됨.
- 기존 공통 UI props 또는 variant가 바뀜.
- 새 도메인 표준 구조가 생김.
- mock/API 전환 방식이 바뀜.
- Figma 작업 방식이 바뀜.
- Claude 프롬프트 템플릿이 바뀜.

갱신 우선 문서.

- `DIRECTORY_MAP.md`
- `COMPONENT_RULES.md`
- `COMMON_FLOWS.md`
- `FORM_RULES.md`
- `QUERY_RULES.md`
- `UI_DEFAULTS.md`
- `CLAUDE_TASK_TEMPLATE.md`

---

## 11. 현재 프로젝트에서 우선 적용할 순서

1. `PROJECT_HEALTH_REPORT.md` 기준으로 문서 최신화.
2. `member` 도메인 구조 정리.
3. `withdrawal-reason` 도메인 구조 정리.
4. `company-*` 도메인 mock/API 경계 정리.
5. 버튼 계층과 DataTable 사용 규칙 문서 보강.
6. Storybook 및 이미지 경고 정리.
7. `.claudeignore` 산출물 제외 강화.

---

## 12. 핵심 운영 문장

이 프로젝트의 AI 바이브 코딩은 다음 한 문장으로 요약한다.

Figma는 분석 자료로 쓰고, 구현은 AI_CONTEXT와 공통 UI를 기준으로 하며, 도메인 구조는 `features + hook + views + table-columns` 패턴으로 통일한다.
