# Anthropic Claude Skills 완벽 정리

> 공식 문서 기반 정리  
> Anthropic: https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf

---

# Claude Skills란?

Claude Skill은 단순한 프롬프트가 아니라,  
재사용 가능한 AI 워크플로우 모듈이다.

쉽게 말하면:

```txt
"매번 반복해서 설명하던 작업 방식"
```

을 하나의 패키지처럼 만들어두고 Claude가 필요할 때 자동으로 사용하는 시스템이다.

예시:

매번 이런 프롬프트를 입력하는 대신:

```txt
Use Expo Router, TypeScript, NativeWind, shared theme system...
```

Skill로 만들어두면:

```txt
expo-nativewind-screen-builder/
  SKILL.md
  references/
  scripts/
```

Claude가 자동으로 해당 작업 방식을 적용할 수 있다.

---

# 핵심 개념: Progressive Disclosure

Anthropic 문서에서 가장 중요하게 강조하는 개념.

## 동작 방식

### 1. YAML Frontmatter

항상 Claude가 읽는다.

```yaml
---
name: expo-nativewind-screen-builder
description: Creates Expo Router React Native screens...
---
```

→ 어떤 Skill인지 판단하는 트리거 역할

---

### 2. SKILL.md 본문

관련 있을 때만 로드된다.

즉:

```txt
필요한 경우에만 상세 규칙 로딩
```

---

### 3. 추가 파일

`references/`, `scripts/`, `assets/`

필요한 순간에만 읽는다.

---

## 왜 중요한가?

거대한 프롬프트를 매 채팅마다 넣지 않아도 된다.

즉:

```txt
토큰 절약
성능 향상
불필요한 컨텍스트 감소
```

Anthropic은 이 구조를 매우 중요하게 본다.

---

# 기본 폴더 구조

필수:

```txt
your-skill/
  SKILL.md
```

권장 구조:

```txt
your-skill/
  SKILL.md
  references/
  scripts/
  assets/
```

---

# 중요한 규칙

## 반드시 파일명이 `SKILL.md`

다른 이름 불가.

```txt
README.md ❌
skill.md ❌
SKILL.MD ❌
SKILL.md ✅
```

---

## 폴더명은 kebab-case 권장

```txt
expo-nativewind-screen-builder ✅
ExpoSkill ❌
frontend_helper ❌
```

---

# YAML Frontmatter = Skill 트리거 시스템

가장 중요한 부분.

Claude는 description을 보고:

```txt
"이 스킬을 써야 하나?"
```

를 판단한다.

---

# 좋은 예시

```yaml
---
name: expo-nativewind-screen-builder
description: Creates Expo Router React Native screens using TypeScript, NativeWind, shared theme tokens, and reusable UI components. Use when user asks to build screens, tabs, onboarding pages, settings pages, or app UI for Expo projects.
---
```

좋은 이유:

- 무엇을 하는지 명확
- 언제 사용하는지 명확
- 트리거 키워드 포함
- 범위가 구체적

---

# 나쁜 예시

```yaml
description: Helps with frontend.
```

문제점:

- 너무 추상적
- 언제 써야 하는지 불명확
- 범위가 너무 넓음
- Claude가 트리거 판단 어려움

---

# Anthropic이 권장하는 description 구성

```txt
무엇을 하는가
+ 언제 사용하는가
+ 어떤 요청에서 트리거되는가
```

또한:

- 1024자 이하
- XML angle bracket 금지 (`< >`)

---

# Skill은 단순 프롬프트가 아니다

Anthropic 관점에서 Skill은:

```txt
prompt + workflow + documentation + validation system
```

에 가깝다.

포함 가능한 것들:

```txt
- 코딩 컨벤션
- API 사용 규칙
- UI 스타일 규칙
- 테스트 명령어
- 검증 로직
- 예제 코드
- 트러블슈팅
- reusable scripts
```

즉:

```txt
AI용 내부 라이브러리
```

같은 개념이다.

---

# 주요 사용 사례

Anthropic은 Skill을 3가지 유형으로 나눈다.

---

# 1. 문서 / 결과물 생성

반복적으로 동일한 형식의 결과물을 만들 때.

예시:

```txt
- 프로젝트 보고서 생성
- 디자인 시스템 적용
- React 화면 생성
- 발표자료 생성
```

---

# 2. 워크플로우 자동화

여러 단계를 반복 수행하는 경우.

예시:

```txt
Google Ads 분석
→ 성능 낮은 캠페인 탐지
→ 키워드 추천
→ 광고 문구 생성
```

---

# 3. MCP 강화

Anthropic 핵심 철학:

```txt
MCP = 도구 접근
Skill = 도구 사용법
```

Anthropic 비유:

```txt
MCP는 주방
Skill은 레시피
```

예시:

```txt
GitHub MCP:
→ 저장소 접근 가능

Skill:
→ 우리 팀의 PR 리뷰 프로세스 적용
```

---

# 좋은 Skill 설계 방법

Anthropic은 먼저:

```txt
2~3개의 실제 사용 사례
```

를 정의하라고 권장한다.

---

# 예시

## Use Case

```txt
Expo 설정 화면 만들기
```

## Trigger

```txt
- create settings page
- build profile screen
```

## Workflow

```txt
1. 라우팅 구조 확인
2. TypeScript 사용
3. NativeWind 적용
4. 공통 컴포넌트 사용
5. theme token 적용
```

## Result

```txt
Production-ready screen
```

---

# 테스트 개념

Anthropic은 Skill도 소프트웨어처럼 테스트하라고 강조한다.

---

# 1. Trigger 테스트

Skill이 적절한 상황에서만 로드되는지 확인.

## 트리거되어야 하는 경우

```txt
- Build Expo settings screen
- Create onboarding UI
- Add tab navigation
```

## 트리거되면 안 되는 경우

```txt
- Explain Python loops
- Write Google Ads copy
```

---

# 2. 기능 테스트

워크플로우가 실제로 잘 동작하는지 검증.

예시:

## Input

```txt
Create guest settings screen
```

## Expected

```txt
- TypeScript component
- NativeWind styling
- theme-safe colors
- expo-router compatible
```

---

# 3. 성능 테스트

Anthropic이 매우 중요하게 보는 부분.

예시:

```txt
기존:
15번 대화 왕복

Skill 적용 후:
2번 clarification 만으로 완료
```

즉:

```txt
반복 대화 감소
실패 감소
토큰 절약
```

이 핵심이다.

---

# Anthropic의 대표 Skill 패턴 5가지

---

# 1. Sequential Workflow

순차 실행이 필요한 경우.

```txt
1. 저장소 분석
2. 아키텍처 파악
3. 코드 생성
4. lint 실행
5. 수정
```

---

# 2. Multi-MCP Coordination

여러 도구를 함께 사용하는 경우.

예시:

```txt
Figma
→ Drive
→ Linear
→ Slack
```

---

# 3. Iterative Refinement

반복 검증을 통해 품질 향상.

```txt
초안 생성
→ 검증
→ 수정
→ 재검증
→ 최종 결과
```

---

# 4. Context-aware Tool Selection

상황에 따라 다른 도구 선택.

예시:

```txt
코드 파일 → GitHub
대용량 파일 → Cloud Storage
협업 문서 → Notion
```

---

# 5. Domain-specific Intelligence

도메인 전문 규칙 내장.

예시:

```txt
법률 번역 견적 시스템
- 문서 유형 판별
- 언어쌍별 가격 정책
- 고객 응답 생성
```

---

# 자주 발생하는 문제들

Anthropic 기준 대부분의 실패 원인:

```txt
- SKILL.md 이름 오류
- YAML 문법 오류
- vague description
- over-triggering
- 지나치게 긴 SKILL.md
- 모호한 규칙
```

---

# 권장 사항

Anthropic 권장:

```txt
SKILL.md는 약 5000단어 이하 유지
```

세부 문서는:

```txt
references/
```

로 분리.

---

# 가장 중요한 핵심 철학

Anthropic이 말하는 핵심은:

```txt
"프롬프트를 잘 써라"
```

가 아니다.

진짜 핵심은:

```txt
반복되는 AI 작업을
재사용 가능하고
테스트 가능하며
배포 가능한 시스템으로 만들어라
```

이다.

---

# 개발자 관점에서 해석하면

Skill은 사실상:

```txt
AI 전용 내부 프레임워크
```

에 가깝다.

즉 단순 지시가 아니라:

```txt
- 언제 사용할지
- 어떤 절차로 실행할지
- 어떤 파일을 참고할지
- 품질 기준이 무엇인지
- 실패 시 어떻게 처리할지
```

를 하나의 시스템으로 패키징하는 개념이다.

---

# React Native / Expo 개발에 적용 예시

## expo-nativewind-skill

포함 가능 내용:

```txt
- Expo Router 규칙
- TypeScript 구조
- NativeWind 스타일 규칙
- Theme token 사용
- 공통 컴포넌트 사용 방식
- react-query 패턴
- react-hook-form + zod 규칙
- API 에러 처리 규칙
- screen 생성 규칙
- 폴더 구조 정책
```

---

# 결론

Anthropic의 Skill 시스템은 단순 프롬프트 저장 기능이 아니다.

핵심은:

```txt
AI 작업 프로세스를
재사용 가능한 엔지니어링 자산으로 만드는 것
```

이다.

특히 개발자에게는:

```txt
반복되는 개발 컨벤션
프로젝트 구조
코드 생성 방식
검증 규칙
```

을 시스템화할 수 있다는 점에서 매우 강력하다.
