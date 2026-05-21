# CLAUDE_REFACTOR_REQUEST — 프로젝트 유지보수성 개선 요청 프롬프트

아래 내용을 Claude에게 그대로 전달한다.

```text
# 작업
현재 프로젝트의 구조를 AI 친화적이고 유지보수성이 높은 형태로 정리해줘.
전체를 한 번에 갈아엎지 말고, 기존 빌드가 성공하는 상태를 유지하면서 단계적으로 정리한다.

# 프로젝트 루트
/Users/momenti/Desktop/projects/moda/moda-frontend-admin

# 반드시 먼저 읽을 문서
1. AI_CONTEXT/AI_ONBOARDING.md
2. AI_CONTEXT/PROJECT_HEALTH_REPORT.md
3. AI_CONTEXT/DIRECTORY_MAP.md
4. AI_CONTEXT/PROJECT_RULES.md
5. AI_CONTEXT/DESIGN_SYSTEM.md
6. AI_CONTEXT/COMPONENT_RULES.md
7. AI_CONTEXT/UI_DEFAULTS.md
8. AI_CONTEXT/COMMON_FLOWS.md
9. AI_CONTEXT/QUERY_RULES.md
10. AI_CONTEXT/FORM_RULES.md
11. AI_CONTEXT/RENDERING_RULES.md

# 절대 하지 말 것
- 전체 프로젝트를 무작정 스캔하지 마.
- node_modules, .next, dist, build, coverage, storybook-static은 읽지 마.
- 기존 UI를 새 컴포넌트로 중복 구현하지 마.
- shadcn/Radix/TanStack/RHF/Zod 외 새 라이브러리를 설치하지 마.
- 기능 동작을 바꾸는 리팩토링을 하지 마.
- Figma 디자인 변경을 시도하지 마.
- 한 번에 모든 도메인을 대규모로 수정하지 마.
- 요청 범위 밖의 시각 스타일을 바꾸지 마.

# 현재 진단 요약
프로젝트는 빌드 가능하고 AI_CONTEXT 기반도 좋다.
하지만 도메인별 구조가 섞여 있다.

좋은 최신 패턴:
- src/features/{domain}/types
- src/features/{domain}/schemas
- src/features/{domain}/mocks
- src/hook/{domain}
- src/components/views/{domain}
- src/components/table-columns/{domain}

개선이 필요한 오래된 패턴:
- page.tsx 내부에 client logic 또는 mock data 존재
- view 내부에 대량 mock data 존재
- utils/validations.ts에 도메인 schema가 과도하게 모임
- mock/API 경계가 도메인별로 다름
- Button 계층 사용 기준이 문서와 일부 불일치
- AI_CONTEXT 일부 문서가 실제 코드보다 오래됨

# 목표
최종적으로 아래 상태를 만든다.

1. AI_CONTEXT 문서가 실제 코드 구조와 일치한다.
2. 모든 도메인의 type/schema/mock/query/view/table-column 위치가 일관된다.
3. page.tsx는 Server Component로 얇게 유지한다.
4. mock data는 feature mocks로 모은다.
5. View는 mock/API 여부를 알지 못한다.
6. 공통 UI 사용 기준이 문서화되어 새 Figma 페이지 구현 때 흔들리지 않는다.
7. build와 typecheck가 계속 성공한다.

# 작업 순서

## 1단계. 문서 최신화만 수행
코드 동작 변경 없이 AI_CONTEXT 문서를 현재 코드 기준으로 업데이트한다.

수정 대상:
- AI_CONTEXT/DIRECTORY_MAP.md
- AI_CONTEXT/COMPONENT_RULES.md
- AI_CONTEXT/FORM_RULES.md
- AI_CONTEXT/QUERY_RULES.md
- AI_CONTEXT/UI_DEFAULTS.md

반영할 내용:
- src/features 구조 추가
- AdminAlert, Tooltip, SortableHeader, FirstButton, SecondButton, IconButton 설명 추가
- Button variant 전체 반영
- schema 위치 정책 정리
- mock hook에서 agent hook으로 전환하는 정책 추가
- DataTable이 지원하는 loading/error/empty/onRowClick/pagination 설명 추가

검증:
- 문서만 수정했다면 build는 생략 가능
- 그래도 markdown 링크와 경로는 실제 파일 존재 여부를 확인

## 2단계. member 도메인 구조 정리
회원 관리 도메인을 최신 feature 구조로 맞춘다.

목표 구조:
- src/features/member/types
- src/features/member/mocks
- src/features/member/schemas
- src/hook/member
- src/components/views/member
- src/components/table-columns/member

수행:
- components/views/member/index.tsx 내부 대량 mock data를 features/member/mocks로 이동
- member 관련 타입을 features/member/types로 이동 또는 재수출
- View는 useGetMemberList 결과 또는 mock hook 결과만 사용
- ACTIVE_MOCK_DATA, ACTIVE_MOCK_ERROR 같은 전역 전환 상수를 제거
- page.tsx는 기존처럼 View import만 유지

검증:
- npx tsc --noEmit
- npm run build

## 3단계. withdrawal-reason 도메인 구조 정리
탈퇴 사유 관리 도메인의 page 내부 mock과 client logic을 정리한다.

목표 구조:
- src/features/withdrawal-reason/types
- src/features/withdrawal-reason/mocks
- src/features/withdrawal-reason/schemas
- src/hook/withdrawal-reason
- src/components/views/withdrawal-reason
- src/components/table-columns/withdrawal-reason가 필요하면 생성

수행:
- members/withdrawal/[reasonId]/page.tsx의 mock detail 제거
- members/withdrawal/[reasonId]/edit/page.tsx의 mock detail 제거
- members/withdrawal/new/page.tsx의 client submit logic을 View로 이동
- detail/edit/create page.tsx는 View import만 하도록 정리
- 기존 hook을 통해 mock/API 경계를 통일

검증:
- npx tsc --noEmit
- npm run build

## 4단계. company 계열 도메인 구조 정리
회사 소개 및 연혁, 회사 약도, 멤버십 안내 도메인의 TODO와 mock/API 경계를 정리한다.

대상:
- company-intro-history
- company-map
- membership-info

수행:
- 반복되는 detail/form row 패턴을 feature 내부 조합 컴포넌트로 묶을지 검토
- API 미연동 TODO를 hook 또는 feature mock 경계로 이동
- View 내부 submit placeholder를 줄인다
- editor 사용 방식은 기존 shared/editor를 유지

검증:
- npx tsc --noEmit
- npm run build

## 5단계. 공통 컴포넌트 규칙 보강
동작 변경 없이 문서와 작은 타입 정리를 한다.

대상:
- Button / FirstButton / SecondButton / IconButton 사용 기준
- DataTable props와 사용 예시
- MultiSelectDropdown empty/search 상태
- Tooltip 사용 예시
- AdminAlert/useAlert 사용 예시

검증:
- npx tsc --noEmit
- npm run build

## 6단계. 경고와 산출물 정리
빌드 경고와 AI 탐색 혼선을 줄인다.

수행:
- Storybook redundant story name warning 정리
- ImageUploadBox와 strip-banner/detail의 img 경고 처리
- .claudeignore에 산출물 추가
- package-lock.json을 .claudeignore에서 제외할지 검토 후 제안

검증:
- npm run lint
- npx tsc --noEmit
- npm run build

# 변경 원칙
- 한 단계씩 작게 수정한다.
- 각 단계 완료 후 변경 파일과 검증 결과를 보고한다.
- 기존 동작을 바꾸지 않는다.
- mock을 제거하지 말고 위치만 정리한다.
- API가 없는 도메인은 mock hook을 유지한다.
- 새 추상화는 3개 이상 반복되는 패턴에만 만든다.
- 공통 UI는 src/components/ui 또는 src/components/shared에 이미 있으면 반드시 재사용한다.

# 완료 보고 형식
완료 요약:
-

변경 파일:
-

검증:
- npm run lint:
- npx tsc --noEmit:
- npm run build:

남은 TODO:
-

주의할 점:
-
```
