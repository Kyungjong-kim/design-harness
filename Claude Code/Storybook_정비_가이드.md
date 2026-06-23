# Storybook 정비 가이드

> 컴포넌트가 늘면서 Storybook 사이드바가 산만해질 때, "외부자가 처음 열어도 정돈된" 상태로 만드는 정비 체크리스트. 라이브러리 첫인상(채용·동료)을 좌우한다.

---

## 1. 카테고리 택소노미 — 고정 N종

스토리 `title`의 `/` 앞 그룹이 난립하면(예: `Form`·`Input`·`Data`·`Display`·`Data Display` 중복) 사이드바가 깨진다. **소수의 고정 카테고리로 통일**한다.

예시 8종 (라이브러리에 맞게 조정):

```
Foundation · Actions · Inputs · Data Display · Feedback · Navigation · Overlay · Layout
```

- 중복·동의어 그룹 통합 (`Form`+`Input`→`Inputs`, `Display`+`Data`→`Data Display`).
- 신규 그룹 임의 추가 금지 — `Conventions.mdx`에 고정 목록을 명시해 강제.
- 컴포넌트명(`/` 뒤)은 유지, 그룹 prefix만 정정.

```bash
# title 라인만 일괄 정정 (예)
sed -i '' -e 's/title: "Form\//title: "Inputs\//' stories/*.stories.tsx
```

---

## 2. 사이드바 순서 — storySort

알파벳 정렬은 의미 순서를 깨뜨린다. `.storybook/preview.ts`에 의도된 순서 고정:

```ts
options: {
  storySort: {
    order: [
      "Docs", ["Welcome", "Design Tokens", "Conventions"],
      "Foundation", "Actions", "Inputs", "Data Display",
      "Feedback", "Navigation", "Overlay", "Layout",
    ],
  },
},
```

MDX 랜딩(`Welcome`·`Design Tokens`·`Conventions`)은 `Docs/` 그룹으로 최상단.

---

## 3. autodocs 전수 적용

`tags: ["autodocs"]`가 빠진 스토리는 Docs 탭(전 변형 + 자동 Props 표 + a11y)이 안 생긴다. 전 컴포넌트에 적용:

```bash
# 누락 식별
for f in stories/*.stories.tsx; do grep -q autodocs "$f" || echo "$f"; done
```

autodocs Docs 페이지 = 사실상 "한눈에 보기" 갤러리. 별도 인앱 쇼케이스를 새로 만들 필요 없음.

---

## 4. MDX 랜딩 문서

| 문서 | 내용 |
|---|---|
| `Welcome.mdx` | 개요·카테고리표(택소노미와 일치)·설치·사용·문서 링크 |
| `DesignTokens.mdx` | 색 스와치 + 비색상 토큰(spacing·radius·size·typography·shadow)표 |
| `Conventions.mdx` | 컴포넌트 작성 규칙 + **카테고리 고정 목록**(신규 스토리 title prefix 강제) |

---

## 5. 흔한 함정 — Docs 페이지 table 스타일 누수

`preview-head.html`에 마크다운 표용 전역 CSS(`.sbdocs table { border … }`)를 두면, **컴포넌트가 `<table>`로 렌더되는 경우**(예: 달력 그리드) Docs 페이지에서 셀마다 보더·패딩이 입혀져 깨진다. Canvas는 멀쩡한데 Docs만 깨지는 증상.

해결 — 렌더된 스토리(`.docs-story`) 내부 table은 마크다운 표 스타일 제외:

```css
/* 마크다운 표 스타일은 MDX 문서용. 컴포넌트 프리뷰 내부 table 제외. */
.sbdocs .docs-story th,
.sbdocs .docs-story td { border: 0; padding: 0; background: none; }
.sbdocs .docs-story table { margin: 0; }
```

> preview-head.html 변경은 HMR 안 됨 — Storybook 재기동 필요.

---

## 6. 정비 체크리스트

```
- [ ] 카테고리 N종으로 통일 (중복 그룹 병합)
- [ ] storySort로 사이드바 순서 고정
- [ ] autodocs 전 컴포넌트 적용 (누락 0)
- [ ] Welcome·DesignTokens·Conventions MDX 최신화 (카테고리·신규 컴포넌트·토큰 반영)
- [ ] Conventions에 카테고리 고정 목록 명시 (신규 스토리 강제)
- [ ] 컴포넌트 table 렌더 시 Docs 표 누수 점검
- [ ] storybook build 통과 (MDX 컴파일·broken story 0)
```

검증: `pnpm build-storybook` exit 0 + 사이드바 육안 확인(사용자 직접).
