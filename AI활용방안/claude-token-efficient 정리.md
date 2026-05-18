# claude-token-efficient 정리

> Repo: https://github.com/drona23/claude-token-efficient

---

# claude-token-efficient 란?

`claude-token-efficient`는 Claude Code용 설정 저장소이다.

핵심 아이디어는 매우 단순하다.

```txt
프로젝트 루트에 짧은 CLAUDE.md 파일을 두고,
Claude가 verbose한 챗봇처럼 행동하지 않고
프로젝트 맞춤형 코딩 어시스턴트처럼 동작하게 만든다.
```

즉:

```txt
- 불필요한 설명 감소
- 토큰 낭비 감소
- 프로젝트 규칙 준수
- 과한 설계 방지
```

를 목표로 한다.

---

# 기본 구조

예시:

```txt
my-next-app/
  CLAUDE.md
  app/
  components/
  package.json
```

또는:

```txt
my-expo-app/
  CLAUDE.md
  app/
  components/
  package.json
```

Claude Code는 프로젝트 루트의 `CLAUDE.md`를 읽고
해당 프로젝트의 기본 행동 규칙으로 사용한다.

---

# 해결하려는 문제

기본적인 AI 코딩 어시스턴트는 종종 다음 문제를 가진다.

```txt
- "좋은 질문입니다!" 같은 불필요한 문장
- 질문 반복
- 수정 전에 긴 설명
- 요청하지 않은 기능 추가
- 단순한 문제를 과도하게 설계
- 잘못된 가정에도 동의
- 테스트 없이 완료했다고 말함
```

Repo에서는 이런 행동을:

```txt
"토큰 낭비이며 개발 생산성에 큰 도움이 되지 않는다"
```

고 설명한다.

---

# 핵심 개념

`CLAUDE.md`는:

```txt
Claude를 위한 팀 규칙 문서
```

같은 개념이다.

매번 이렇게 말하는 대신:

```txt
- 파일 먼저 읽어라
- 과한 설계 하지 마라
- 간단하게 구현해라
- 패키지 함부로 추가하지 마라
- 답변 짧게 해라
- 테스트 후 완료라고 해라
```

프로젝트에 한 번만 정의해두는 방식이다.

---

# 왜 중요한가?

핵심은 단순 토큰 절약이 아니다.

더 중요한 건:

```txt
Claude의 행동을 예측 가능하게 만드는 것
```

이다.

즉:

```txt
"우리 프로젝트 스타일대로 동작하는 Claude"
```

를 만드는 개념이다.

---

# Next.js 프로젝트에서 중요한 이유

Next.js 프로젝트에서는 AI가 쉽게 오버엔지니어링한다.

---

# 예시

## 요청

```txt
Add a protected dashboard route.
```

---

## 나쁜 AI 동작

```txt
- 새로운 auth framework 추가
- 불필요한 middleware abstraction 생성
- 여러 helper 파일 생성
- 관련 없는 layout 수정
- auth 이론 장문 설명
```

---

## 좋은 동작

```txt
- 현재 프로젝트 구조 확인
- 기존 auth/session 로직 읽기
- 필요한 파일만 수정
- App Router 규칙 사용
- 중요한 변경점만 설명
```

---

# 예시 코드

```tsx
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  const token = request.cookies.get("session")?.value;

  if (!token && request.nextUrl.pathname.startsWith("/dashboard")) {
    return NextResponse.redirect(new URL("/login", request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ["/dashboard/:path*"],
};
```

핵심은:

```txt
새로운 아키텍처를 만드는 것이 아니라
현재 프로젝트 기준으로
가장 작은 유효 수정(minimal useful fix)을 만드는 것
```

이다.

---

# Expo 프로젝트에서도 동일

Expo에서도 같은 문제가 발생한다.

---

# 예시

## 요청

```txt
Create a settings screen.
```

---

## 나쁜 AI 동작

```txt
- 새로운 navigation library 설치
- expo-router 무시
- custom theme system 생성
- 상태관리 추가
- 기존 컴포넌트 구조 무시
```

---

## 좋은 동작

```txt
- 기존 app/ route 구조 확인
- expo-router 사용
- 기존 NativeWind 사용 여부 확인
- 기존 UI 컴포넌트 재사용
- 불필요한 native 수정 금지
```

---

# 예시 코드

```tsx
// app/(tabs)/settings.tsx
import { Link } from "expo-router";
import { View, Text } from "react-native";

export default function SettingsScreen() {
  return (
    <View className="flex-1 bg-background px-4 pt-6">
      <Text className="text-2xl font-semibold text-foreground">
        Settings
      </Text>

      <View className="mt-6 gap-3">
        <Link
          href="/sign-in"
          className="rounded-xl bg-primary px-4 py-3 text-center text-primary-foreground"
        >
          Sign in
        </Link>

        <Link
          href="/sign-up"
          className="rounded-xl border border-border px-4 py-3 text-center text-foreground"
        >
          Create account
        </Link>
      </View>
    </View>
  );
}
```

핵심은:

```txt
Claude가 generic React Native 코드를 생성하지 않고
현재 Expo 프로젝트 구조를 존중하는 것
```

이다.

---

# Repo에서 권장하는 사용 방식

2가지 방식을 제안한다.

---

# 1. 규칙 직접 붙여넣기

짧은 작업이나 일회성 세션용.

---

# 2. 프로젝트 루트에 CLAUDE.md 추가

반복 작업이나 팀 프로젝트용.

실제 팀 환경에서는:

```txt
프로젝트 루트의 CLAUDE.md 방식이 훨씬 유용
```

하다.

---

# 중요한 한계

이 방식이 Claude를 완벽하게 만드는 것은 아니다.

Repo도 명확하게 말한다.

```txt
CLAUDE.md가 AI의 환각(hallucination)을 해결해주지는 않는다.
```

즉:

```txt
- 코드 리뷰
- 테스트
- 타입체크
- lint
- 사람 승인
```

은 여전히 필요하다.

더 정확한 표현은:

```txt
CLAUDE.md는
반복되는 행동 문제를 줄이고
프로젝트 규칙을 더 잘 따르게 만든다.
```

이다.

---

# Benchmark 내용

Repo에서는 간단한 벤치마크를 공유한다.

결과:

```txt
응답 단어 수 약 63% 감소
```

하지만 Repo 스스로도:

```txt
엄격한 통계 실험이 아닌
방향성 정도의 참고 데이터
```

라고 설명한다.

즉:

```txt
"무조건 생산성이 향상된다"
```

로 받아들이기보다는:

```txt
실제 우리 프로젝트에서 테스트 필요
```

로 보는 게 맞다.

---

# 개발 작업에 가장 적합한 프로필

Repo에는:

```txt
CLAUDE.coding.md
```

프로필이 존재한다.

이 프로필은 특히 개발 작업에 유용하다.

---

# 특징

```txt
- 코드 먼저 반환
- boilerplate 최소화
- speculative feature 금지
- 불필요한 abstraction 금지
- 수정 전 파일 읽기
- 리뷰 범위 최소화
```

---

# 특히 잘 맞는 작업

```txt
- Next.js App Router 수정
- Expo Router 화면 생성
- TypeScript 에러 수정
- 리팩토링
- 디버깅
- 코드 리뷰
- 소규모 기능 개발
```

---

# 팀 적용 추천 방식

Repo를 그대로 복붙하는 것보다는:

```txt
우리 프로젝트에 맞는
짧은 CLAUDE.md를 직접 만드는 것이 좋다.
```

---

# 추천 예시

```md
# CLAUDE.md

## General rules

- Read existing files before editing.
- Do not edit files blindly.
- Prefer targeted changes over full rewrites.
- Keep output concise.
- Return code first when code is requested.
- Explain only non-obvious decisions.
- Do not add packages without checking package.json first.
- Do not add abstractions for single-use logic.
- Do not add speculative features.
- Do not say done unless the change was tested or typechecked.
- If tests cannot be run, say why.
- User instructions override this file.

## Next.js rules

- Respect the existing App Router structure.
- Prefer Server Components unless client state, browser APIs, or event handlers are required.
- Use "use client" only when needed.
- Check existing auth, i18n, validation, API, and styling patterns before adding new ones.
- Do not introduce new state management or data fetching libraries unless requested.
- Do not create new folders or architecture unless the current structure requires it.

## Expo rules

- Respect expo-router file-based routing.
- Use existing route groups and layout files.
- Use NativeWind classes if the project already uses NativeWind.
- Reuse existing UI components before creating new ones.
- Use existing theme constants before adding new theme logic.
- Do not modify ios/ or android/ unless native changes are required.
- Do not add React Navigation manually if expo-router already handles routing.

## Review rules

- State the issue.
- Show the fix.
- Stop.
```

---

# 팀에 설명하는 방법

예시 설명:

> 이 Repo는 라이브러리가 아니라 Claude Code용 규칙 파일 모음이다.
> 
> 핵심 파일인 `CLAUDE.md`는 Claude가 프로젝트 내부에서 어떻게 행동해야 하는지를 정의한다.
> 
> 목적은:
> 
> - 불필요한 출력 감소
> - 과한 설계 방지
> - 프로젝트 규칙 준수
> - 일관된 개발 스타일 유지
> 
> 이다.
> 
> Next.js와 Expo 프로젝트에서는:
> 
> - 기존 파일 먼저 읽기
> - 현재 라우팅 구조 존중
> - 기존 스타일링 시스템 사용
> - 불필요한 패키지 추가 금지
> - 전체 구조 리팩토링 대신 작은 수정 우선
> 
> 같은 행동을 Claude에게 강제할 수 있다.
> 
> 단, 코드 리뷰와 테스트를 대체하지는 않는다.
> 
> 먼저 짧은 CLAUDE.md로 시작해서
> 실제 반복 문제를 기준으로 점진적으로 개선하는 것이 좋다.

---

# 실제 적용 추천 플랜

## 1단계

Next.js 프로젝트 하나에 짧은 `CLAUDE.md` 추가

---

## 2단계

실제 작업에 며칠간 사용

확인할 것:

```txt
- 관련 없는 파일 수정 감소 여부
- 응답 길이 감소 여부
- App Router 규칙 준수 여부
- 불필요한 abstraction 감소 여부
- 테스트/typecheck 실행 여부
```

---

## 3단계

Expo 프로젝트에도 적용

---

# 중요한 최종 포인트

`CLAUDE.md`는:

```txt
많을수록 좋은 것이 아니다.
```

너무 길어지면:

```txt
- 입력 토큰 증가
- 유지보수 어려움
- 규칙 충돌 증가
- Claude 혼란 증가
```

문제가 발생할 수 있다.

핵심은:

```txt
짧고
명확하고
반복 문제 중심
```

으로 유지하는 것이다.
