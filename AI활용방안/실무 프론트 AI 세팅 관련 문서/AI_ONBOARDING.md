# AI_ONBOARDING — Moda Admin

## 프로젝트 정의

Moda 서비스의 관리자 포털. 회원·콘텐츠·점포 등 백오피스 데이터를 CRUD하는 SPA.

- **기술 스택**: Next.js 14 App Router / React 18 / TypeScript strict / Tailwind v4
- **상태**: TanStack Query(서버) + Zustand(UI) + react-hook-form(폼)
- **디자인**: Figma admin 토큰 기반 커스텀 디자인 시스템

---

## 시작점 파일 읽기 순서

새 세션에서 작업을 시작할 때 아래 순서로 읽는다.

1. `AI_CONTEXT/DIRECTORY_MAP.md` — 폴더 구조와 각 파일의 역할
2. `AI_CONTEXT/DESIGN_SYSTEM.md` — 사용 가능한 토큰/클래스
3. `AI_CONTEXT/COMPONENT_RULES.md` — 컴포넌트 작성 규칙
4. `AI_CONTEXT/UI_DEFAULTS.md` — 정렬/빈 상태/이탈 alert 공통 UX 기본값
5. `AI_CONTEXT/COMMON_FLOWS.md` — 목록/등록/상세/수정/모달 공통 흐름
6. `AI_CONTEXT/VIBE_CODING_PROCESS.md` — AI 바이브 코딩 운영 프로세스
7. `AI_CONTEXT/QUERY_RULES.md` — API 호출 및 서버 상태 관리
8. `AI_CONTEXT/FORM_RULES.md` — 폼 구현 패턴
9. `AI_CONTEXT/RENDERING_RULES.md` — 페이지 구조 및 렌더링
10. 작업 대상 feature가 있으면 `AI_CONTEXT/FEATURES/{feature}.md`

Figma URL을 기준으로 화면을 구현하는 작업이면 아래 문서를 2번 직후에 추가로 읽는다.

1. `AI_CONTEXT/FIGMA_MCP_WORKFLOW.md` — Figma MCP 분석·구현 절차
2. `AI_CONTEXT/CLAUDE_TASK_TEMPLATE.md` — Claude 작업 요청 표준 템플릿

---

## 현재 구현 상태

| 영역 | 상태 |
|---|---|
| 로그인 (`/sign-in`) | 완성 |
| 대시보드 (`/dashboard`) | UI 껍데기만 있음, 데이터 미연동 |
| 사용자 관리 (`/users`) | CRUD 완성 (목록/생성/수정/삭제) |
| 공통 UI 컴포넌트 | 기본 세트 완성 (일부 Figma 기본 UX는 `UI_DEFAULTS.md` 상태표 확인) |
| 사이드 메뉴 | 완성 (2-depth 아코디언, collapsed 지원) |
| 인증 미들웨어 | 코드 있음, **현재 주석 처리 상태** |
| 나머지 8개 메뉴 그룹 | `routes.ts`에 정의됨, **페이지 미구현** |

---

## 핵심 규칙 요약

- 새 파일 첫 줄: 한국어 헤더 주석 (`// 역할 설명`)
- 스타일: 오직 admin 토큰 사용 (`text-admin-primary`, `bg-admin-gray-1` 등)
- API 호출: 반드시 `agent()` 래퍼 사용 (`src/utils/fetch.ts`)
- 서버 상태: TanStack Query (`useQuery` / `useMutation`)
- 전역 알림: `useAlert()` (`src/providers/alert-provider.tsx`)
- 폼: react-hook-form + zodResolver + Zod 스키마

전체 규칙은 `AI_CONTEXT/PROJECT_RULES.md` 참조.
