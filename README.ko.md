# design-harness

**React 컴포넌트 라이브러리 / 디자인시스템** 전용 Claude Code 하네스. npm publish 대상 라이브러리(`dist/` 산출물을 외부 앱이 소비)에 맞춰 만들어진 워크플로우.

> 자매 프로젝트 [`claude-code-harness`](https://github.com/Kyungjong-kim/claude-code-harness)는 **앱(Next.js / Vite / FastAPI 등) 전용**.
> 라이브러리·디자인시스템 프로젝트라면 본 하네스를 사용하세요.

---

## 왜 별도 하네스인가?

앱 하네스는 페이지·라우트·UI 검증(Playwright)·`pages/<도메인>/FE_*.md` 문서 패턴을 가정합니다. 라이브러리는 다음과 같은 다른 도메인입니다:

- **컴포넌트·모듈 단위** (페이지가 없음)
- **`dist/` 빌드 산출물** — tsup / rollup / microbundle 등 번들러 기반
- **`peerDependencies`** — 앱 전체 의존성과 다름
- **Storybook 시각 검증** (Playwright 부적합)
- **디자인 토큰** (core / semantic 2계층) — 앱 상태 관리와 다른 차원

하나의 하네스에 모두 끼워 넣으면 노이즈만 늘어납니다. 본 하네스는 라이브러리 작업에 필요한 것만 담았습니다.

---

## 기대치 설정

`/project-init` 1회 실행으로 끝나는 부트스트랩이 아닙니다. 라이브러리 운영은 누적이 핵심입니다.

| 시점 | 누적 자산 | 비고 |
|------|-----------|------|
| **M0** (초기 구축) | CLAUDE.md ~150줄, docs/<라이브러리>/agent/ 4종, 에이전트 2종 | `/project-init` 직후 |
| **M1** (1개월 운영) | + design-system.md 토큰·variant 정리, HANDOFF 누적, 컴포넌트 5~10개 분량 패턴 정립 | 컨벤션이 잡혀가는 시점 |
| **M6** (6개월 운영) | + 기능 정리 문서 누적, 회고 자산(history/), 컴포넌트 20+개 패턴 표준화 | 새 컴포넌트 추가가 보일러플레이트 답습 수준으로 가벼워짐 |

핵심은 **5개 산출물 동시 작성** 규칙: 신규 컴포넌트 = `<name>.tsx` + `<name>.test.tsx` + `index.ts` + `<name>.stories.tsx` + `components/<index>.ts` export 추가. 5개 모두 채우지 않으면 미완료. 이것이 라이브러리 운영의 가장 큰 보일러플레이트 자동화 효과입니다.

---

## 시작하기

### 1. 하네스 클론 및 설치

```bash
git clone https://github.com/Kyungjong-kim/design-harness.git ~/projects/design-harness

# 라이브러리 프로젝트로 이동
cd <your-library-project>

# 프로젝트 로컬 설치 (OSS 라이브러리에 권장 — .claude/skills/ 에 격리)
bash ~/projects/design-harness/install.sh --local

# 또는 전역 설치 (개인 환경에서 여러 라이브러리 작업)
bash ~/projects/design-harness/install.sh
```

**Windows native (PowerShell):** `bash install.sh` 대신 `install.ps1` 사용 (플래그 동일: `-Local`·`-Personal`·`-Yes`·`-Force`).

```powershell
powershell -ExecutionPolicy Bypass -File ~\projects\design-harness\install.ps1 -Local
```

> 🧪 native Windows 지원은 **실험적**입니다 — `install.ps1`은 `install.sh`의 1:1 포팅(Windows Store `python3` 스텁 회피·실제 `python` 탐지, `cp`/`rm` 대신 `robocopy`, UTF-8 BOM). 임시 디렉터리 dry run으로 확인했으나 실환경 검증은 진행 중. 완전 검증 경로는 WSL2.

### 2. Claude Code 실행 후 부트스트랩

```bash
claude
> /project-init
```

### 3. 설치 검증

```bash
# 스킬 설치 확인 (--local 모드)
ls .claude/skills/ | grep -E "project-init|project-fix|session-close|document-review"

# CLAUDE.md 생성 확인 (/project-init 후)
wc -l CLAUDE.md

# GitHub CLI 인증 확인 (/project-fix · /project-pr · /project-issue 사용 시)
gh auth status
```

`/project-init`는 다음을 수행합니다:
- 빌드 도구(tsup/rollup) · `peerDependencies` · `exports` 자동 감지
- 라이브러리 카테고리(디자인시스템 / 유틸 라이브러리 / SDK) Q&A
- `CLAUDE.md` 생성 — STEP 0~3 + 5개 산출물 강제 + 토큰 사용 규칙
- `docs/<라이브러리>/` 구조 — `components/` (페이지가 아닌 컴포넌트 도메인)
- `agent/architecture.md` · `conventions.md` · `design-system.md` 작성
- `.claude/agents/<라이브러리>-dev.md` + `<라이브러리>-doc-writer.md` 생성
- Git 워크플로우 — 컨벤셔널 커밋 + 이슈 트래커 옵션화 (1인 OSS 지원)

### 3. 첫 컴포넌트 추가

부트스트랩 후 첫 작업은 새 컴포넌트 추가가 자연스럽습니다.

```
> Progress 컴포넌트 추가해줘
```

에이전트는 자동으로:
1. 유사 컴포넌트 2개 분석 (Radix / 단순 / DnD 카테고리 분류)
2. 5개 산출물 구현 계획 제시 → 사용자 확인
3. 작성 후 `pnpm test` / `pnpm build` / `pnpm lint` 검증
4. `pnpm storybook`으로 시각 확인 안내

---

## 본체와의 차이

| 항목 | claude-code-harness (앱) | **design-harness (라이브러리)** |
|---|---|---|
| 도메인 단위 | `pages/<도메인>/` | **`components/<컴포넌트>/`** |
| 시각 검증 | Playwright (`ui-verification.md`) | **Storybook + 단위 테스트** |
| 핵심 문서 | `FE_<기능>.md` (기능 단위) | **`design-system.md` (토큰·variant 통합) + 컴포넌트별 정리 (선택)** |
| 빌드 가정 | `next dev` / `vite dev` | **`tsup --watch` / `rollup -w`** |
| 강제 규칙 | API 레이어 분리·상태관리 패턴 | **5개 산출물 동시 작성 / semantic 토큰만 사용** |
| 훅 (페이지 자동 감지) | 사용 (`pre-commit-doc-check.sh`) | **미사용 — 페이지 단위 부적합** |
| 이슈 번호 | 강제 | **옵션** (1인 OSS 친화) |
| 영역 브래킷 (`[FE]`/`[BE]`) | 사용 | **미사용** (단일 패키지) |

---

## 주요 문서

| 문서 | 경로 | 역할 |
|---|---|---|
| CLAUDE.md 템플릿 | `Claude Code/템플릿/CLAUDE.md_템플릿.md` | 라이브러리 전용 진입점 템플릿 |
| Agent 문서 템플릿 | `Claude Code/템플릿/agent_문서_템플릿.md` | architecture / conventions / design-system 템플릿 |
| 컴포넌트 정리 템플릿 | `Claude Code/템플릿/컴포넌트_정리_템플릿.md` | 컴포넌트별 상세 정리 (선택) |
| 에이전트 정의 가이드 | `Claude Code/에이전트_정의_가이드.md` | `<라이브러리>-dev` / `<라이브러리>-doc-writer` 정의 |
| 적용 사례 | `examples/` | 실 적용 사례 — M0·M1·M6 진화 흐름 (마스킹·일반화) |

---

## 라이선스

MIT — [LICENSE](./LICENSE) 참고.

---

## 변경 이력

[CHANGELOG.md](./CHANGELOG.md) 참고.
