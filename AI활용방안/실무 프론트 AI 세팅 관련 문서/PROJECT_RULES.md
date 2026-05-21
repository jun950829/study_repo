# PROJECT_RULES — 프로젝트 공통 규칙

## 파일 헤더 주석

모든 신규 `.ts` / `.tsx` 파일의 첫 줄은 한국어 한 줄 주석이어야 한다.

```ts
// 사용자 생성 뮤테이션 훅
```

- `'use client'` 디렉티브가 있으면 그 **아래** 줄에 작성.
- `*.config.ts`, `package.json` 등 설정 파일은 제외.
- 이유: 에이전트가 파일 전체를 읽지 않고도 역할을 파악할 수 있어야 함.

---

## TypeScript

- `tsconfig.json`: `strict: true`. null check, 타입 단언 최소화.
- 경로 alias: `@/*` → `src/*`. `@/components/*`, `@/hook/*`, `@/utils/*` 등 별도 alias 존재.
- 전역 타입은 `src/types/type.d.ts`에 `declare interface`로 정의.

```ts
// 이미 선언된 전역 타입
IResponsePaged<T>   // 페이지네이션 응답
IApiResponse<T>     // 단건 API 응답
ErrorData           // { status, code, message }
```

---

## 코드 스타일

Prettier 설정 (`prettier.config` or `.prettierrc`):
```json
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2,
  "plugins": ["prettier-plugin-organize-imports", "prettier-plugin-tailwindcss"]
}
```

- import 자동 정렬: `prettier-plugin-organize-imports` 적용됨.
- Tailwind 클래스 자동 정렬: `prettier-plugin-tailwindcss` 적용됨.

---

## ESLint

```json
{ "extends": ["next/core-web-vitals", "next/typescript", "plugin:storybook/recommended"] }
```

- `next/core-web-vitals` + TypeScript 규칙 적용.
- 스토리 파일은 storybook 규칙 추가 적용.

---

## Git / 커밋

- `commitizen` + `cz-conventional-changelog` 사용. `npm run commit`으로 대화형 커밋.
- `husky` + `lint-staged`: `*.{ts,tsx}` 파일에 commit 전 `eslint --fix` + `prettier --write` 자동 실행.
- 커밋 단위: 논리적으로 분리 가능한 최소 단위. 여러 기능을 한 커밋에 섞지 않는다.

---

## 스타일링 규칙

- 모든 색상/타이포/radius/shadow는 admin 토큰 사용. shadcn 기본 토큰(`bg-primary`, `text-foreground` 등) 사용 금지.
- 예외: 아직 admin 토큰이 없는 항목은 Tailwind 기본값 임시 사용 후 반드시 TODO 주석.
- `cn()` (`src/lib/utils.ts`): 조건부 클래스 병합 시 항상 사용.

---

## 변경 범위 원칙

- 요청된 변경만 수정. 인접 코드 리팩터링 금지.
- 내가 만든 미사용 import/변수/함수는 즉시 제거.
- 기존 코드의 미사용 dead code는 발견 시 언급만, 삭제하지 않음.

---

## 테스트 / 빌드 확인

- 코드 수정 후 반드시 `npx tsc --noEmit` 실행.
- 완료 전 `npm run build` 성공 확인.
- Storybook 테스트: `vitest` + `@storybook/addon-vitest` + Playwright Chromium 연동. `.stories.tsx` 파일이 테스트 대상.
