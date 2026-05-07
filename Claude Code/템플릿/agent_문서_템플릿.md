---
tags: [하네스, 템플릿, agent, 컨텍스트, 라이브러리]
updated: 2026-04-30
---

# agent/ 문서 템플릿 — 라이브러리 전용

`docs/<라이브러리명>/agent/` 디렉토리에 위치하는 에이전트 컨텍스트 문서 3종 템플릿.
각 섹션을 분리해 해당 파일에 복사·작성한다.

> **에이전트 사용 안내**
> 이 파일은 3개의 하위 문서 템플릿을 담고 있다.
> 각 섹션(`---`로 구분)을 별도 파일로 분리해 작성한다.
>
> | 섹션 | 파일 경로 | 작성 대상 |
> |------|-----------|----------|
> | § 1 architecture | `docs/<라이브러리명>/agent/architecture.md` | 모든 라이브러리 |
> | § 2 conventions | `docs/<라이브러리명>/agent/conventions.md` | 모든 라이브러리 |
> | § 3 design-system | `docs/<라이브러리명>/agent/design-system.md` | **디자인시스템·UI 라이브러리만**. SDK·CLI·유틸 라이브러리는 생략 |
>
> 디자인시스템·UI 라이브러리에서는 § 3 design-system.md가 **핵심 문서**다. CLAUDE.md `agent/` 트리거에서 갱신 빈도가 가장 높다.
>
> **분석 전 필수 확인 목록 — 라이브러리**
> - `package.json` → `exports` / `main` / `module` / `types` / `peerDependencies` / `sideEffects`
> - 빌드 도구 — `tsup.config.ts` · `rollup.config.*` · `vite.config.*` (라이브러리 모드)
> - 컴포넌트 디렉토리 — `src/components/` · `src/primitives/` · `lib/` 등
> - 디자인 토큰 — `src/styles/tokens/` · `tailwind.config.*` · `tokens.json` (디자인시스템 한정)
> - 시각 검증 도구 — `.storybook/` · `playground/` · `examples/`
> - 테스트 — `vitest.config.*` · `jest.config.*` · 테스트 파일 위치 패턴
> - 릴리스 — `.changeset/` · `release-please-config.*` · `semantic-release` 설정
> - 코드 스타일 — `biome.json` · `.eslintrc.*` · `.prettierrc.*`
> - variant 시스템 — `class-variance-authority` 또는 동등 라이브러리 사용 여부

---

## § 1 — architecture.md

> 파일 경로: `docs/<라이브러리명>/agent/architecture.md`

```markdown
# [라이브러리명] 아키텍처 (에이전트용 요약)

> 이 문서는 에이전트가 작업을 시작할 때 읽는 **라이브러리 구조 인덱스**입니다.
>
> **문서 신선도**: 마지막 확인 YYYY-MM-DD

---

## 1. 라이브러리 개요

<!-- FILL: 라이브러리 메타 정보 표
  분석 기준:
  - package.json: name·version·peerDependencies
  - 빌드 도구: tsup / rollup / vite (lib mode) / esbuild
  - 출력 포맷: ESM·CJS·UMD 중 어느 것 (tsup format / rollup output)
  - 타입 시스템: TS strict 여부, .d.ts 출력 방식
  - 테스트 도구: vitest / jest
  - 시각 검증: Storybook / Playground / 없음 (UI 라이브러리만)
  - 버전 관리: Changesets / semantic-release / 수동
-->

[라이브러리명]은 [라이브러리 유형] 라이브러리이며 `dist/` 산출물을 외부 앱이 npm 패키지처럼 소비합니다.

| 항목 | 값 |
|---|---|
| **타입** | 라이브러리 (npm publish 대상) — 앱 아님 |
| **빌드 도구** | [tsup 8.x / rollup / vite lib mode] ([ESM + CJS dual] 등) |
| **타입 시스템** | TypeScript [버전] (`dts: true` — `.d.ts` 함께 출력) |
| **패키지 매니저** | [pnpm / npm / yarn] |
| **린터·포매터** | [biome / eslint+prettier] |
| **테스트** | [vitest / jest] + Testing Library |
| **시각 검증** | [Storybook / Playground / 해당 없음] |
| **버전 관리** | [Changesets / semantic-release / 수동] |

---

## 2. exports 진입점

<!-- FILL: package.json `exports` 필드 분석 후 표 작성
  exports 가 없으면 main/module/types 기준
  스타일 entry(`./styles`)는 디자인시스템 한정
-->

`package.json` `exports` 필드 기준:

| 진입점 | 파일 | 용도 |
|---|---|---|
| `[라이브러리명]` | `dist/index.js` (ESM) / `dist/index.cjs` (CJS) | 컴포넌트·유틸 import |
| `[라이브러리명]/styles` | `dist/styles.css` | 디자인 토큰 CSS 로드 (디자인시스템 한정) |

**주의 사항** (해당 시 작성):
- `dist/styles.css`가 디자인 토큰만 포함하는 구조라면 — 사용처(외부 앱)의 Tailwind/PostCSS가 dist를 스캔해야 유틸리티 클래스가 생성됨을 명시
- `sideEffects: false` 처리 여부 (tree-shaking 영향)
- subpath exports 동결·확장 정책

---

## 3. peerDependencies / dependencies

<!-- FILL: package.json 분석 후 표 작성
  peer/dep 분리:
  - peer = 사용처에서 제공해야 하는 것 (react, react-dom 등)
  - dep = 라이브러리가 가져가는 것 (Radix, cva, clsx 등)

  tsup external 정책도 함께 기재 (어떤 패키지를 외부로 두는지)
-->

| 종류 | 패키지 | 비고 |
|---|---|---|
| **peer** | `[패키지 ^버전]` | 사용처에서 제공 |
| **dep** | `[그룹별로 분류]` | [번들 포함 여부·external 정책] |

> **[빌드도구] external**: `[external 목록]`. 나머지는 사용처가 transitive dependency로 가져감.

---

## 4. 디렉토리 구조

<!-- FILL: 실제 src 구조 (find src -maxdepth 3 -type d) -->

```
src/
├── index.ts                  # 모든 export 진입점
├── components/               # (디자인시스템·UI 라이브러리)
│   ├── index.ts              # 카테고리 재export
│   └── primitives/
│       ├── index.ts          # 컴포넌트 export
│       └── <컴포넌트>/
│           ├── <컴포넌트>.tsx
│           ├── <컴포넌트>.test.tsx
│           └── index.ts
├── styles/                   # (디자인시스템 한정)
│   ├── tokens.css
│   └── tokens/
│       ├── core.css
│       └── semantic.css
├── utils/
│   └── cn.ts                 # className 결합 유틸
└── types/                    # (SDK·타입 라이브러리)

stories/                      # Storybook 스토리 (UI 라이브러리)
.storybook/                   # Storybook 설정
.changeset/                   # 버전 관리 (Changesets 사용 시)
dist/                         # 빌드 산출물 (gitignore)
```

---

## 5. 컴포넌트 패턴 *(디자인시스템·UI 라이브러리 한정)*

<!-- FILL: SDK·유틸 라이브러리는 본 섹션을 "주요 모듈 패턴"으로 교체하거나 삭제.
  디자인시스템 라이브러리는 아래 3개 하위 항목을 그대로 채움. -->

### 5-1. 디렉토리 구조 (1 컴포넌트당)

```
src/components/<카테고리>/<kebab-case>/
├── <kebab-case>.tsx          # 컴포넌트 본체
├── <kebab-case>.test.tsx     # 단위 테스트
└── index.ts                  # export 재노출

stories/<kebab-case>.stories.tsx          # 스토리
src/components/<카테고리>/index.ts         # ← export 추가 갱신 필수
```

### 5-2. forwardRef 사용 기준

| 컴포넌트 종류 | forwardRef | 예시 |
|---|---|---|
| Radix·다른 라이브러리 래핑 (Trigger·Content 분리) | ✅ 사용 | `[예시 컴포넌트들]` |
| 단순 props 전달 함수 | ❌ 미사용 (함수 선언) | `[예시 컴포넌트들]` |
| DOM ref 노출 필요 (Button·Input 류) | ✅ 사용 | `[예시 컴포넌트들]` |

### 5-3. variant 시스템

<!-- FILL: cva 미사용 라이브러리는 본 섹션 삭제. 다른 variant 라이브러리(tv·tailwind-variants 등) 사용 시 그에 맞춰 작성. -->

`class-variance-authority` (cva) 기반. variant·size 두 키로 표준화:

```ts
const buttonVariants = cva("기본 클래스", {
  variants: {
    variant: { primary: "...", secondary: "...", ghost: "...", destructive: "..." },
    size: { sm: "...", md: "...", lg: "..." },
  },
  defaultVariants: { variant: "primary", size: "md" },
});
```

---

## 6. 빌드·배포 흐름

<!-- FILL: 실제 명령으로 교체. 시각 검증·릴리스가 없으면 행 삭제. -->

```
[개발] [개발 명령]      →  dist/ 자동 갱신
[테스트] [테스트 명령]  →  N files / N tests
[린트] [린트 명령]
[시각 검증] [시각 검증 명령]   →  localhost:[포트] (UI 라이브러리)
[릴리스] [릴리스 명령]   →  npm publish
```

에이전트는 코드 작업 시:
1. 수정 대상이 **어느 카테고리**(예: Radix 래핑 / 단순 / DnD)인지 먼저 분류
2. 동일 카테고리 사례 2개 이상 읽고 패턴 답습
3. 빌드·테스트·(시각 검증) 검증 후 완료
```

---

## § 2 — conventions.md

> 파일 경로: `docs/<라이브러리명>/agent/conventions.md`

```markdown
# [라이브러리명] 코딩 컨벤션 (에이전트용 요약)

---

## 1. 기준 설정 파일

<!-- FILL: 실제 설정 파일 목록 -->

- `[린터·포매터 설정]` — 포매터·린터 통합 (예: biome.json / .eslintrc + .prettierrc)
- `tsconfig.json` — TypeScript strict
- `[빌드 설정]` — 빌드 entry·external (예: tsup.config.ts)

---

## 2. 핵심 규칙

<!-- FILL: 린터 설정·기존 코드 분석 후 작성
  분석 기준:
  - 파일·디렉토리 네이밍 (kebab-case·camelCase·PascalCase)
  - 컴포넌트 네이밍·선언 방식 (function·forwardRef)
  - export 방식 (named·default)
  - 타입 정의 패턴 (interface·type·VariantProps)
  - className 결합 유틸 (cn·twMerge·classnames 등)
-->

| 항목 | 규칙 |
|------|------|
| **포매터** | [biome / prettier] (들여쓰기·따옴표·세미콜론 등 자동) |
| **파일 네이밍** | [kebab-case / camelCase] (예: `dnd-list/dnd-list.tsx`) |
| **컴포넌트 네이밍** | PascalCase (예: `DndList`, `EmptyState`) |
| **디렉토리** | [kebab-case / camelCase] |
| **컴포넌트 선언** | 함수형 (`function Foo()`) — 단순 / `forwardRef` — ref 노출 시 |
| **export** | named export만 사용 (default export 금지) |
| **타입** | `interface FooProps` 또는 `VariantProps<typeof fooVariants>` |
| **className 결합** | `cn(...)` 유틸 (`src/utils/cn.ts`) — `clsx + twMerge` |

---

## 3. 컴포넌트 파일 내 선언 순서 *(디자인시스템·UI 라이브러리)*

```ts
1. cva variants 정의 (있으면)
2. interface Props 정의
3. 컴포넌트 함수 (forwardRef 또는 function)
4. displayName (forwardRef 사용 시 필수)
5. 하위 컴포넌트 (있으면 — 예: AccordionItem, AccordionTrigger)
6. export {...}
```

> 클래스·SDK·유틸 라이브러리는 본 섹션을 모듈 내 선언 순서(상수 → 타입 → 함수/클래스 → export)로 교체.

---

## 4. 테스트 파일 패턴

<!-- FILL: 테스트 도구·관행에 맞춰 작성 -->

- `<이름>.test.tsx` 같은 폴더에 위치 (또는 `tests/` 디렉토리)
- [vitest / jest] + Testing Library
- `describe(이름, () => { ... })` 그룹
- 최소 항목 — 디자인시스템: 렌더링 / 주요 prop / 이벤트 핸들러 / variant
- 최소 항목 — SDK: 정상 흐름 / 에러 흐름 / 옵션·파라미터 분기

---

## 5. 스토리 파일 패턴 *(UI 라이브러리 한정)*

- `stories/<kebab-case>.stories.tsx`
- `meta.title = "[카테고리]/<PascalCase>"` (예: `"Primitives/Button"`)
- `Default` 스토리 + variant·size별 스토리

> Storybook 미사용 라이브러리는 본 섹션 삭제.

---

## 6. 디자인 토큰 사용 *(디자인시스템 한정)*

> 상세 규칙은 `design-system.md` 참조.

- 컴포넌트 className에 raw hex(`#fabc37`) 작성 금지
- semantic 토큰 변수(`bg-bg-brand-default` 등) 사용
- spacing·radius도 토큰 변수 우선

---

## 7. 에이전트 작업 시 주의

- 새 컴포넌트 추가 시 **N개 산출물 동시 작성** (CLAUDE.md STEP 1 참조)
- [린터] 자동 포맷에 맡김 — 수동 들여쓰기 조정 금지
- 기존 사례 카테고리와 동일한 패턴을 따른다
- 리팩토링은 **요청된 부분만** 변경 (Surgical Changes)
```

---

## § 3 — design-system.md *(디자인시스템·UI 라이브러리만 작성)*

> 파일 경로: `docs/<라이브러리명>/agent/design-system.md`

> SDK·CLI·유틸 라이브러리는 이 섹션을 통째로 생략한다.
> 디자인시스템·UI 라이브러리에서는 **핵심 문서**다.

```markdown
# [라이브러리명] 디자인 시스템 (에이전트용 가이드)

---

## 1. 디자인 토큰 구조

<!-- FILL: 실제 토큰 구조에 맞게 작성

  [2-tier 구조 예시 — 권장 기본값]
  - core (raw 값) → semantic (의미 토큰) → 컴포넌트는 semantic만 참조

  [3-tier 구조 예시 — 더 큰 시스템]
  - primitive (raw) → semantic (의미) → component (컴포넌트별 토큰) → 컴포넌트가 참조

  [1-tier 구조 예시 — 작은 시스템]
  - 단일 토큰 파일에 의미 키만 정의

  본 템플릿은 2-tier 기본값으로 작성. 다른 구조면 트리·표 교체.
-->

[라이브러리명]은 **2계층 토큰** 구조를 따릅니다.

```
src/styles/
├── tokens.css                 # 진입점 (core + semantic import)
└── tokens/
    ├── core.css               # raw 값 (#fabc37 등) — 직접 참조 금지
    └── semantic.css           # core 참조하는 의미 토큰 — 컴포넌트가 사용
```

**원칙**: 컴포넌트는 **semantic 토큰만** 참조한다. core 토큰을 직접 쓰지 않는다.

---

## 2. core — raw 값 카테고리

<!-- FILL: 실제 core 토큰 카테고리·prefix·단계 -->

| 카테고리 | 변수 prefix | 단계 |
|---|---|---|
| **brand** | `--color-brand-*` | 25, 50, 100~900 |
| **neutral** | `--color-neutral-*` | 25, 50, 100~900 |
| **red** | `--color-red-*` | 25, 50, 100~900 |
| **green** | `--color-green-*` | 25, 50, 100~900 |
| **white/black** | `--color-white`, `--color-black` | — |

---

## 3. semantic — 의미 토큰 (컴포넌트가 직접 사용)

<!-- FILL: 실제 semantic 토큰 카테고리·패턴·예시 -->

| 카테고리 | 패턴 | 예시 |
|---|---|---|
| **text** | `--color-text-{primary, secondary, tertiary, disabled, inverse, brand-default, brand-hover, danger-default, success-default, warning-default}` | `--color-text-primary` |
| **bg** | `--color-bg-{primary, secondary, tertiary, inverse, brand-default, brand-hover, brand-subtle, danger-default, danger-subtle, ...}` | `--color-bg-brand-default` |
| **border** | `--color-border-{default, strong, brand-default, danger-default, disabled, focus}` | `--color-border-default` |
| **interactive** | `--color-interactive-{primary, secondary, ghost, destructive}-{bg, bg-hover, text, border}` | `--color-interactive-primary-bg` |

> Tailwind v4 사용 시 자동으로 `bg-bg-brand-default` 같은 클래스로 노출.
> Tailwind v3·다른 스타일링 도구는 별도 변환 단계 명시.

---

## 4. 작업 원칙

| 규칙 | 위반 예 | 올바른 예 |
|---|---|---|
| **하드코딩 색상 금지** | `bg-[#fabc37]` | `bg-bg-brand-default` |
| **core 토큰 직접 사용 금지** | `bg-brand-500` | `bg-bg-brand-default` (semantic 경유) |
| **임의 px 금지** | `p-[7px]` | `p-2` (Tailwind 기본 spacing) |
| **새 색상 필요 시 토큰 먼저 추가** | 컴포넌트에 색상 직접 정의 | `tokens/semantic.css`에 토큰 추가 후 참조 |

---

## 5. variant 패턴 (cva)

<!-- FILL: cva 미사용 라이브러리는 본 섹션 삭제 또는 다른 variant 라이브러리(tv·tailwind-variants)로 교체. -->

컴포넌트는 `class-variance-authority` 사용. variant·size 두 키로 표준화:

```ts
const buttonVariants = cva("기본 공통 클래스", {
  variants: {
    variant: {
      primary: "bg-bg-brand-default text-white",
      secondary: "bg-white border border-border-default",
      ghost: "hover:bg-bg-tertiary",
      destructive: "bg-bg-danger-default text-white",
    },
    size: {
      sm: "h-8 px-3 text-sm",
      md: "h-10 px-4 text-base",
      lg: "h-12 px-6 text-lg",
    },
  },
  defaultVariants: { variant: "primary", size: "md" },
});
```

**variant 추가 시 체크리스트:**
- 기존 variant와 의미 충돌 없는지
- 모든 size에서 시각적 일관성 유지되는지
- semantic 토큰만 사용했는지
- 스토리에 새 variant 추가했는지
- 테스트에 variant prop 검증 추가했는지

---

## 6. 새 토큰 추가 절차

1. 의미 토큰이 부족한 상황을 판단 (예: `info` 카테고리 추가 필요)
2. core 토큰에 색상 단계 추가 (필요 시) — `tokens/core.css`
3. semantic 토큰 추가 — `tokens/semantic.css`
4. `[빌드 명령]` 실행해 `dist/styles.css` 갱신 확인
5. 기존 컴포넌트에 영향이 있는지 grep 검토
6. `agent/design-system.md` 갱신 (이 문서)

---

## 7. 자주 사용하는 토큰 빠른 참조

<!-- FILL: 실제 사용 빈도가 높은 토큰 추출 -->

| 용도 | 토큰 |
|------|------|
| 본문 텍스트 | `text-text-primary` |
| 보조 텍스트 | `text-text-secondary` |
| 비활성 텍스트 | `text-text-disabled` |
| 기본 배경 | `bg-bg-primary` |
| 카드/패널 배경 | `bg-bg-secondary` |
| 브랜드 강조 배경 | `bg-bg-brand-default` |
| 위험 액션 배경 | `bg-bg-danger-default` |
| 기본 보더 | `border-border-default` |
| 포커스 보더 | `border-border-focus` |
```
