# PROJECT_HEALTH_REPORT — 프로젝트 구조 진단

작성일: 2026-05-21

대상 프로젝트: `/Users/momenti/Desktop/projects/moda/moda-frontend-admin`

이 문서는 현재 프로젝트가 AI 기반 바이브 코딩, Figma MCP 기반 구현, 장기 유지보수에 적합한 구조인지 점검한 결과다.

---

## 1. 결론

현재 프로젝트는 빌드 가능한 상태이고, AI 친화적인 기반도 이미 많이 갖춰져 있다.

종합 평가는 다음과 같다.

| 항목 | 평가 |
|---|---|
| 빌드 안정성 | 양호 |
| TypeScript 기본 설정 | 양호 |
| Figma 기반 작업 준비 | 양호 |
| 공통 UI 기반 | 양호 |
| 도메인별 구현 일관성 | 보통 |
| mock/API 전환 준비도 | 보통 이하 |
| 폴더 구조 일관성 | 보통 |
| 장기 유지보수성 | 개선 필요 |
| AI_CONTEXT 기반 작업성 | 양호 |

핵심 판단.

- 새로 추가한 `AI_CONTEXT` 문서 덕분에 Claude/Codex가 작업을 시작하는 방식은 꽤 좋아졌다.
- 공통 UI, admin token, DataTable, Dialog, Alert, Editor, Tooltip 같은 기반은 존재한다.
- 최근 구현된 도메인들은 `features/{domain}/types|schemas|mocks` 구조를 쓰고 있어 방향이 좋다.
- 다만 기존 도메인과 최근 도메인의 구조가 섞여 있어, 앞으로 도메인이 늘어나면 유지보수 비용이 빠르게 커질 수 있다.
- `mock data`, `schema`, `type`, `query hook`, `view`, `page.tsx` 책임 경계가 도메인별로 다르다.

---

## 2. 검증 결과

실행한 명령.

```bash
npm run lint
npx tsc --noEmit
npm run build
```

결과.

- `npm run lint`: 성공. 경고 있음.
- `npx tsc --noEmit`: 성공.
- `npm run build`: 성공.

주요 경고.

- Storybook story name 중복 경고.
- `ImageUploadBox.tsx`, `strip-banner/detail.tsx`에서 `<img>` 사용 경고.

빌드가 성공하므로 당장 개발을 막는 치명적 문제는 없다.

---

## 3. 현재 잘 되어 있는 점

### AI 친화성

- `AI_CONTEXT` 폴더가 존재하고 문서가 비교적 구체적이다.
- `FIGMA_MCP_WORKFLOW.md`, `CLAUDE_TASK_TEMPLATE.md`, `COMMON_FLOWS.md`가 있어 Figma 기반 작업 지시가 쉬워졌다.
- 신규 파일 첫 줄 한국어 헤더 주석 규칙이 잡혀 있다.
- Figma generated code 복붙 금지, remote asset 금지, 공통 UI 우선 사용 규칙이 문서화되어 있다.

### 프론트 아키텍처

- Next.js App Router 기반 라우트 구조가 명확하다.
- `page.tsx` → `components/views/{domain}` 흐름이 대부분 유지된다.
- `src/components/ui`와 `src/components/shared`가 분리되어 있다.
- `src/components/table-columns/{domain}` 분리가 도입되어 table column 관리가 좋아졌다.
- TanStack Query, RHF, Zod, Zustand, Tailwind v4, Radix 기반 UI는 현재 트렌드와 잘 맞는다.

### 디자인 시스템

- `globals.css`에 admin token이 정의되어 있다.
- `DESIGN_SYSTEM.md`가 token 사용 기준을 설명한다.
- `Icons` registry와 SVGR 설정이 있어 아이콘 색상 제어가 가능하다.
- `AdminAlert`, `Tooltip`, `SortableHeader`, `DataTable`, `Pagination`, `DateTimeInput`, `ImageUploadBox`, `Editor` 등 반복 UI가 공통화되어 있다.

### 최근 도메인 구조

최근 추가된 도메인은 아래 구조를 따른다.

```text
src/features/{domain}/
  mocks/
  schemas/
  types/

src/hook/{domain}/
src/components/views/{domain}/
src/components/table-columns/{domain}/
```

이 구조는 좋다. 앞으로 모든 도메인을 이 방향으로 맞추면 된다.

---

## 4. 주요 문제

### 4-1. 도메인 구조가 두 가지 이상으로 섞여 있다.

최근 도메인.

- `admin-account`
- `admin-role`
- `terms`
- `privacy-policy`
- `strip-banner`

위 도메인은 `features/{domain}` 구조를 쓴다.

기존 도메인.

- `member`
- `withdrawal-reason`
- `user`
- `company-intro-history`
- `company-map`
- `membership-info`

위 도메인은 mock, schema, type, submit 로직 위치가 제각각이다.

예시.

- `member/index.tsx`에 대량 mock data가 직접 들어 있다.
- `withdrawal-reason` 상세/수정 page에 mock detail이 직접 들어 있다.
- `company-*` 계열 form에는 API 연결 TODO가 view 내부에 있다.
- `utils/validations.ts`와 `features/*/schemas`가 동시에 존재한다.

결과적으로 AI가 새 도메인을 구현할 때 어떤 패턴을 따라야 하는지 흔들릴 수 있다.

### 4-2. `page.tsx`의 책임이 도메인마다 다르다.

좋은 패턴.

- `page.tsx`는 Server Component로 두고 View만 import한다.
- 실제 로직은 `components/views/{domain}`에서 처리한다.

흔들리는 패턴.

- 일부 `page.tsx`가 `'use client'`를 포함한다.
- 일부 `page.tsx`가 mock data와 submit handler를 직접 가진다.
- 일부 동적 route는 `params: Promise<...>`를 쓰고, 일부는 `params: { id: string }`을 쓴다.

권장 방향.

- Next.js 14 기준으로 `page.tsx`는 얇게 유지한다.
- 동적 params 타입을 프로젝트 표준으로 통일한다.
- mock/detail/submit 로직은 View 또는 hook/feature mock으로 이동한다.

### 4-3. mock/API 전환 전략이 완전히 통일되지 않았다.

좋은 패턴.

- `features/{domain}/mocks/*.mock.ts`
- `hook/{domain}/useGet*.ts`가 mock function을 호출한다.
- 나중에 `agent()`로 교체하기 쉽다.

흔들리는 패턴.

- 일부 도메인은 실제 `agent()`를 사용한다.
- 일부 도메인은 hook mock을 사용한다.
- 일부 도메인은 View/page에 mock을 직접 둔다.
- mock 주석과 TODO가 도메인마다 다른 방식으로 남아 있다.

권장 방향.

- mock은 항상 `src/features/{domain}/mocks`에 둔다.
- hook은 항상 API와 mock의 경계가 된다.
- View는 mock인지 API인지 알지 못하게 만든다.

### 4-4. validation 위치가 섞여 있다.

현재 상태.

- 공통/기존 schema는 `src/utils/validations.ts`.
- 최근 도메인 schema는 `src/features/{domain}/schemas`.

권장 방향.

- 신규 도메인 schema는 `src/features/{domain}/schemas`.
- `utils/validations.ts`는 로그인, 공통 password rule 등 범용 schema만 유지한다.
- 기존 도메인 schema는 단계적으로 feature 폴더로 이동한다.

### 4-5. 버튼 컴포넌트 계층이 늘어나고 있다.

현재 존재.

- `Button`
- `FirstButton`
- `SecondButton`
- `IconButton`
- `ExcelDownloadButton`

이 자체가 문제는 아니지만, 도메인마다 `Button variant="point"`를 쓰거나 `FirstButton`을 쓰는 방식이 섞인다.

권장 방향.

- `FirstButton`, `SecondButton`, `IconButton`은 Figma 명명 기반 컴포넌트로 유지한다.
- 일반 form/list에서는 어느 버튼 계층을 우선할지 문서화한다.
- `Button` variants와 Figma button family 간 매핑표를 `COMPONENT_RULES.md`에 보강한다.

### 4-6. DataTable 공통화는 되어 있으나 기능이 일부 View에 흩어져 있다.

현재 DataTable이 제공하는 것.

- header 유지.
- loading/error/empty body row.
- pagination.
- row click.

View에 반복되는 것.

- sort state.
- filtered/sorted/paginated 계산.
- pageCount 계산.
- search/reset 처리.

권장 방향.

- 당장은 View에 둬도 괜찮다.
- 도메인이 더 늘면 `useTableQueryState`, `useClientTableState` 같은 작은 hook으로 검색/정렬/페이지네이션 상태를 분리한다.
- 서버 API 연결 이후에는 client-side sorting/pagination을 제거하고 서버 파라미터로 통일한다.

### 4-7. 산출물과 개발 로그가 루트에 남아 있다.

루트에 다음 파일/폴더가 존재한다.

- `storybook-static`
- `debug-storybook.log`
- `tsconfig.tsbuildinfo`
- `context-notes.md`
- `checklist.md`
- `memory/`
- `docs/`

`.gitignore`에는 `storybook-static`, `*.tsbuildinfo`, `*storybook.log`가 포함되어 있으므로 Git 추적 위험은 낮다.
다만 AI가 파일 탐색 시 혼란을 줄 수 있다.

권장 방향.

- `.claudeignore`에 `storybook-static`, `*.tsbuildinfo`, `debug-storybook.log`, `memory`, 오래된 `docs`를 추가한다.
- 실제 온보딩 기준은 `AI_CONTEXT`로 단일화한다.

### 4-8. `.claudeignore`가 너무 좁고 일부 기준이 애매하다.

현재 `.claudeignore`.

```text
node_modules
.next
dist

pnpm-lock.yaml
package-lock.json
```

문제.

- `storybook-static`, `coverage`, `build`, `debug-storybook.log`, `tsconfig.tsbuildinfo`가 빠져 있다.
- `package-lock.json`을 숨기면 의존성 충돌을 볼 때 Claude가 정확한 lock 상태를 못 본다.

권장 방향.

- 일반 작업에서는 lockfile을 안 읽어도 되지만, dependency 작업에서는 읽을 수 있어야 한다.
- `.claudeignore`는 산출물 중심으로 정리한다.

---

## 5. 도메인별 상태

| 도메인 | 상태 | 구조 평가 | 주요 메모 |
|---|---|---|---|
| `admin-account` | 구현됨 | 좋음 | features/types/schemas/mocks 구조 사용 |
| `admin-role` | 구현됨 | 좋음 | 권한 체크 구조 존재, validation은 아직 `utils/validations.ts` |
| `strip-banner` | 구현됨 | 좋음 | link target 공통 options 사용 |
| `terms` | 구현됨 | 좋음 | feature schema/mock/type 구조 사용 |
| `privacy-policy` | 구현됨 | 좋음 | terms와 유사 구조 |
| `member` | 구현됨 | 개선 필요 | View 내부 대량 mock, API hook과 mock 혼재 |
| `withdrawal-reason` | 구현됨 | 개선 필요 | page 내부 mock/detail/submit 로직 존재 |
| `company-intro-history` | 구현됨 | 개선 필요 | API TODO가 View 내부에 남아 있음 |
| `company-map` | 구현됨 | 개선 필요 | API TODO가 View 내부에 남아 있음 |
| `membership-info` | 구현됨 | 개선 필요 | editor 기반 구조는 좋으나 feature 경계 필요 |
| `user` | 초기 도메인 | 보통 | 기존 CRUD 패턴, 최신 feature 구조와 다름 |
| `dashboard` | shell | 보통 | 데이터 미연동 |

---

## 6. 우선순위 개선 과제

### P0. AI 기준 문서 최신화.

- `DIRECTORY_MAP.md`를 현재 `src/features`, `shared/Tooltip`, `shared/SortableHeader`, 사이트/관리자/약관 route 기준으로 갱신한다.
- `COMPONENT_RULES.md`에 `Button` variant 전체, FirstButton/SecondButton/IconButton 사용 기준을 추가한다.
- `FORM_RULES.md`에 feature schema 위치 정책을 추가한다.
- `QUERY_RULES.md`에 mock hook에서 agent hook으로 전환하는 표준을 추가한다.

### P1. mock 위치 통일.

- `member` mock을 `src/features/member/mocks`로 이동한다.
- `withdrawal-reason` mock detail을 `src/features/withdrawal-reason/mocks`로 이동한다.
- company 관련 mock/TODO를 `features/company-*` 또는 hook 경계로 이동한다.

### P1. page.tsx 얇게 만들기.

- `members/withdrawal/new/page.tsx`
- `members/withdrawal/[reasonId]/page.tsx`
- `members/withdrawal/[reasonId]/edit/page.tsx`

위 파일의 client 로직과 mock을 View/hook/feature로 이동한다.

### P1. feature boundary 정리.

모든 도메인을 아래 구조로 맞춘다.

```text
src/features/{domain}/
  types/
  schemas/
  mocks/
  constants/
  components/
```

필요한 것만 만든다. 빈 폴더는 만들지 않는다.

### P2. query key와 option 상수 확장.

- query key export를 한 줄 export에서 grouped export로 정리한다.
- option 상수는 `constants/options.ts`에 계속 둘지, feature option으로 둘지 기준을 정한다.
- 범용 옵션은 `constants/options.ts`, 도메인 전용 옵션은 `features/{domain}/constants`로 둔다.

### P2. DataTable 주변 반복 제거.

- sorting toggle 로직이 여러 목록에 반복된다.
- client-side filtering/sorting/pagination 로직도 반복된다.
- 당장 큰 추상화는 금지하고, 3개 이상 동일한 패턴이 생긴 뒤 hook으로 분리한다.

### P2. Storybook/빌드 경고 정리.

- redundant story name warning 제거.
- `<img>` 경고는 `next/image` 또는 명시적 eslint disable 정책 중 하나로 정리한다.

### P3. `.claudeignore` 정리.

권장.

```text
node_modules
.next
dist
build
coverage
storybook-static
*.tsbuildinfo
*storybook.log
.env.local
memory
```

dependency 작업을 정확히 해야 하므로 `package-lock.json`은 기본적으로 숨기지 않는 쪽을 권장한다.

---

## 7. Claude 작업 시 주의점

Claude에게 한 번에 전체 리팩토링을 맡기지 않는다.

권장 순서.

1. 문서 최신화만 먼저 수행.
2. `member` 구조 정리.
3. `withdrawal-reason` 구조 정리.
4. `company-*` 구조 정리.
5. 버튼/폼/쿼리 규칙 문서 보강.
6. Storybook/이미지 경고 정리.

각 단계마다 `npx tsc --noEmit`, `npm run build`를 실행한다.

---

## 8. 최종 판단

이 프로젝트는 바이브 코딩 세팅이 나쁜 상태가 아니다.
오히려 AI_CONTEXT와 공통 UI 기반은 잘 잡혀 있다.

지금 필요한 것은 새로운 추상화를 많이 만드는 것이 아니라, 이미 생긴 좋은 패턴을 기존 도메인으로 역전파하는 일이다.

특히 아래 원칙만 고정하면 이후 Figma 페이지 구현 속도와 품질이 안정된다.

- `page.tsx`는 얇게 유지한다.
- View는 UI orchestration만 담당한다.
- API/mock 경계는 hook이 담당한다.
- type/schema/mock은 feature가 담당한다.
- UI primitive/shared는 중복 생성하지 않는다.
- AI_CONTEXT를 실제 코드 상태와 계속 맞춘다.
