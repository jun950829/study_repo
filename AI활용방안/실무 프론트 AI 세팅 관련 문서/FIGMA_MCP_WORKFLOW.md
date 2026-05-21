# FIGMA_MCP_WORKFLOW — Figma 기반 구현 절차

Figma URL을 기준으로 새 화면, 모달, 공통 UI를 구현할 때 반드시 이 절차를 따른다.
이 문서는 Figma를 코드로 옮기는 순서를 고정해서 불필요한 프로젝트 스캔과 중복 컴포넌트 생성을 막기 위한 기준이다.

---

## 1. 입력값 정리

작업 프롬프트에서 아래 값을 먼저 추출한다.

- `fileKey`
- `nodeId`
- 화면 종류: 목록, 등록, 상세, 수정, 모달, 공통 컴포넌트
- 연결 화면: 버튼 클릭, 상세 이동, 수정 이동, 삭제/확인 모달
- 사용할 local asset 경로
- 금지 사항: 새 라이브러리 설치, Figma generated code 복붙, remote asset URL 사용

Figma URL에 `node-id=403-8113`처럼 들어오면 `nodeId`는 `403:8113`으로 변환한다.

---

## 2. Figma MCP 호출 순서

Figma 디자인을 구현하기 전, 같은 메시지에서 아래 3개를 병렬로 호출한다.

1. `get_design_context(fileKey, nodeId)`
2. `get_variable_defs(fileKey, nodeId)`
3. `get_metadata(fileKey, nodeId)`

각 결과의 용도는 고정한다.

| MCP 결과 | 사용 목적 |
|---|---|
| `get_design_context` | 실제 레이아웃, 텍스트, 컴포넌트 구조, 스크린샷 기준 확인 |
| `get_variable_defs` | Figma 변수와 프로젝트 admin token 매핑 |
| `get_metadata` | 노드 계층, 반복 패턴, frame 크기, 주요 영역 구조 확인 |

Figma MCP 결과가 충분하지 않으면 같은 node에 대해 추가 호출하지 말고, 필요한 하위 node만 좁혀서 다시 확인한다.

---

## 3. 구현 전 산출물

코드 수정 전에 아래 5개를 먼저 정리한다.

1. 레이아웃 구조
   - 화면 종류
   - 전체 frame 크기
   - 주요 영역 트리
   - flex/grid 방향, padding, gap, align

2. 디자인 토큰 매핑
   - 색상
   - 타이포
   - spacing
   - radius
   - shadow
   - `AI_CONTEXT/DESIGN_SYSTEM.md`의 admin token을 최우선 사용

3. 사용 컴포넌트
   - Figma instance
   - 반복되는 row/card/table/form pattern
   - 버튼, 입력, 셀렉트, 체크박스, 모달, 테이블, 에디터

4. 공통 UI 매핑
   - `AI_CONTEXT/COMPONENT_RULES.md`
   - `AI_CONTEXT/UI_DEFAULTS.md`
   - 기존 공통 UI로 해결되는 항목과 feature 내부 조합 컴포넌트로 둘 항목을 분리

5. 구현 계획
   - 3~4단계로 압축
   - 라우트, View, table-columns, hook/mock data, modal 흐름을 포함
   - 검증은 마지막에 `npx tsc --noEmit` + `npm run build` 1회

---

## 4. 코드 변환 원칙

- Figma generated React code를 그대로 복붙하지 않는다.
- Figma remote image URL을 사용하지 않는다.
- local asset이 지정된 경우 지정 경로만 사용한다.
- Figma 절대 좌표를 페이지 전체에 그대로 하드코딩하지 않는다.
- 기존 admin layout의 Sidebar, Header, main padding 흐름을 유지한다.
- Figma spacing은 가능한 admin token, 공통 컴포넌트 padding, 기존 page pattern으로 매핑한다.
- 동일한 UI가 2회 이상 반복되면 feature 내부 조합 컴포넌트로 분리한다.
- 공통화 가치가 명확한 경우에만 `src/components/shared` 또는 `src/components/ui`를 수정한다.

---

## 5. 프로젝트 문서 읽기 순서

Figma 구현 작업에서는 아래 순서로 읽는다.

1. `AI_CONTEXT/AI_ONBOARDING.md`
2. `AI_CONTEXT/DIRECTORY_MAP.md`
3. `AI_CONTEXT/DESIGN_SYSTEM.md`
4. `AI_CONTEXT/COMPONENT_RULES.md`
5. `AI_CONTEXT/UI_DEFAULTS.md`
6. `AI_CONTEXT/COMMON_FLOWS.md`
7. `AI_CONTEXT/FORM_RULES.md`
8. `AI_CONTEXT/QUERY_RULES.md`
9. `AI_CONTEXT/RENDERING_RULES.md`
10. 작업 대상 feature 문서

이후 필요한 파일만 좁혀서 읽는다.

---

## 6. 파일 탐색 제한

전체 프로젝트를 스캔하지 않는다.

우선 허용 범위.

- 작업 대상 `src/app/(auth)` 또는 `src/app/(no-auth)` 라우트
- 작업 대상 `src/components/views/{feature}`
- 작업 대상 `src/components/table-columns/{feature}`
- 관련 `src/hook/{feature}`
- `src/components/ui`
- `src/components/shared`
- `src/constants/routes.ts`
- `src/constants/queryKeys.ts`
- `src/utils/validations.ts`
- `src/utils/fetch.ts`

읽지 않는 범위.

- `node_modules`
- `.next`
- `dist`
- `build`
- `coverage`
- generated 파일
- 관련 없는 feature 전체

---

## 7. Figma 비교 기준

구현 완료 후 아래만 비교한다.

- page title과 breadcrumb 위치
- 본문 card width, padding, gap
- table column 순서, header, row height, empty/error body
- form label cell width, input 높이, required 표시
- button 위치, 크기, 색상, disabled 상태
- modal width, title/body/button 정렬
- editor toolbar와 content area 높이

픽셀 완전 일치보다 기존 공통 UI와 admin token 일관성을 우선한다.

---

## 8. 금지 사항

- Figma generated code 복붙 금지
- Figma remote asset URL 사용 금지
- 새 UI 라이브러리 설치 금지
- Header, Sidebar 중복 구현 금지
- 공통 UI와 동일한 기능의 feature 전용 컴포넌트 중복 생성 금지
- 요청되지 않은 Storybook 추가 금지
- 요청되지 않은 테스트 파일 추가 금지
