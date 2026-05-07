---
name: project-init
description: |
  React 컴포넌트 라이브러리·디자인시스템 프로젝트에 design-harness를 초기화한다.
  코드베이스 파악(빌드 도구·exports·peerDeps 자동 감지) → Q&A → CLAUDE.md · 에이전트 · HANDOFF 전체 구축까지 원스톱.
  라이브러리(npm publish 대상, dist 산출물) 전용. 앱(Next.js/Vite/FastAPI 등)에는 별도 OSS `claude-code-harness` 사용.
  트리거 키워드: "하네스 구축", "라이브러리 초기화", "디자인시스템 하네스", "/project-init"
---

# Project Init (Library)

라이브러리·디자인시스템 프로젝트에 design-harness를 처음 설치한다.
상세 구현 기준: `<하네스경로>/Claude Code/초기_설정_체크리스트.md`

> **이 하네스의 적용 대상**: React 컴포넌트 라이브러리 / 디자인시스템 / SDK 등 npm publish 대상.
> 빌드 산출물(`dist/`)을 외부 앱이 소비하는 패키지.
> 앱 프로젝트(페이지·라우팅 있음)는 본 하네스가 부적합 — `claude-code-harness` 사용.

---

## Step 0 — 환경 확인 (첫 실행 시)

design-harness 스킬이 이 환경에 설치돼 있는지 확인한다. **이미 설치된 환경이면 Step 1로 바로 진행한다.**

```bash
# 하네스 스킬 설치 여부 확인 (전역 + 로컬)
HARNESS_SKILLS=(project-init project-fix project-issue project-pr session-close document-review)
MISSING=()
for skill in "${HARNESS_SKILLS[@]}"; do
  if [ ! -d "$HOME/.claude/skills/$skill" ] && [ ! -d ".claude/skills/$skill" ]; then
    MISSING+=("$skill")
  fi
done
if [ ${#MISSING[@]} -gt 0 ]; then
  echo "⚠ 미설치 스킬: ${MISSING[*]}"
  echo ""
  echo "설치 옵션:"
  echo "  A) 전역 설치: bash <design-harness경로>/install.sh"
  echo "  B) 로컬 설치 (OSS 라이브러리 권장): bash <design-harness경로>/install.sh --local"
else
  echo "✓ design-harness 스킬 모두 설치됨"
fi
```

| 설치 방법 | 명령 | 적용 범위 |
|-----------|------|-----------|
| **전역 설치** | `bash <design-harness경로>/install.sh` | 이 기기의 모든 라이브러리 프로젝트 |
| **로컬 설치** (OSS 권장) | `bash <design-harness경로>/install.sh --local` | 이 프로젝트만. `.claude/skills/`에 생성. git 추적 시 contributor가 `git clone` 후 바로 사용 가능 |

### 재실행 감지 (중간 취소 후 재시작)

```bash
PARTIAL=""
[ -f CLAUDE.md ] && PARTIAL="${PARTIAL}CLAUDE.md "
[ -d docs ] && find docs -maxdepth 4 -name "HANDOFF_NOW.md" 2>/dev/null | grep -q . && PARTIAL="${PARTIAL}docs/HANDOFF_NOW.md "
[ -d .claude/agents ] && ls .claude/agents/*.md 2>/dev/null | head -1 && PARTIAL="${PARTIAL}에이전트(로컬) "
echo "감지: ${PARTIAL:-없음}"
```

| 감지 결과 | 처리 |
|-----------|------|
| 아무것도 없음 | 신규 구축 — Step 1로 진행 |
| 일부 파일 존재 | 사용자에게 재개 지점 선택 (A) 처음부터 / (B) 미완성 단계부터 / (C) 취소 |
| 모두 존재 | 이미 완료된 구축 — Step 10 빌드 검증만 실행 |

---

## Step 1 — 코드베이스 파악 (사용자 개입 없음)

> **실행 디렉토리**: 라이브러리 패키지 루트(`package.json` 있는 곳)에서 실행.
> 모노레포 안의 한 패키지를 구축한다면 그 패키지 루트로 이동 후 실행.

```bash
# git 저장소 확인
if ! git rev-parse --git-dir > /dev/null 2>&1; then
  echo "⚠ git 저장소가 아닙니다. git init 후 다시 실행."
fi

# 기존 하네스 구조 확인
ls CLAUDE.md .claude/ docs/ 2>/dev/null && echo "기존 하네스 존재" || echo "신규 구축"
[ -f CLAUDE.md ] && echo "--- 기존 CLAUDE.md ---" && cat CLAUDE.md
[ -d docs ] && find docs -maxdepth 3 -name "*.md" | sort

# 프로젝트 구조 파악
find . -maxdepth 4 -type d \
  -not -path "*/node_modules/*" -not -path "*/.git/*" \
  -not -path "*/dist/*" -not -path "*/storybook-static/*" | sort

# package.json 라이브러리 시그널 자동 감지
python3 - << 'EOF' 2>/dev/null
import json, pathlib
try:
    pkg = json.loads(pathlib.Path('package.json').read_text())
except Exception as e:
    print(f"⚠ package.json 읽기 실패: {e}"); exit()

print(f"name: {pkg.get('name')}")
print(f"version: {pkg.get('version')}")
print(f"main: {pkg.get('main')}")
print(f"module: {pkg.get('module')}")
print(f"exports: {list(pkg.get('exports', {}).keys()) if isinstance(pkg.get('exports'), dict) else pkg.get('exports')}")

deps = {**pkg.get('dependencies', {}), **pkg.get('devDependencies', {})}
peer = pkg.get('peerDependencies', {})

bundlers = [b for b in ['tsup', 'rollup', 'microbundle', 'unbuild', 'tsdx', 'vite'] if b in deps]
print(f"번들러: {bundlers if bundlers else '(미감지)'}")

print(f"peerDependencies: {list(peer.keys()) if peer else '(없음)'}")

ui_libs = [l for l in ['@radix-ui', 'react-aria', '@headlessui'] if any(l in k for k in deps)]
if ui_libs: print(f"UI 프리미티브: {ui_libs}")

if 'class-variance-authority' in deps: print("variant 시스템: cva")
if any('storybook' in k for k in deps): print("시각 검증: Storybook 설치됨")
if pathlib.Path('.changeset').exists(): print("버전 관리: Changesets")
if pathlib.Path('biome.json').exists(): print("린터·포매터: biome")
elif any(pathlib.Path(p).exists() for p in ['.eslintrc.json', '.eslintrc.js', '.eslintrc.cjs']): print("린터: ESLint")

print(f"scripts: {list(pkg.get('scripts', {}).keys())}")
EOF

# 컴포넌트 디렉토리 자동 감지
echo ""
echo "=== 컴포넌트 디렉토리 후보 ==="
for dir in src/components src/components/primitives src/lib src/ui packages/components; do
  [ -d "$dir" ] && echo "  ✓ $dir ($(ls "$dir" 2>/dev/null | wc -l)개 항목)"
done

# 디자인 토큰 위치 자동 감지
echo ""
echo "=== 디자인 토큰 후보 ==="
for f in src/styles/tokens.css src/tokens/index.css tailwind.config.ts tailwind.config.js src/styles/globals.css; do
  [ -f "$f" ] && echo "  ✓ $f"
done

# git 정보
git remote get-url origin 2>/dev/null
git log --oneline -10 2>/dev/null
git branch -a 2>/dev/null | head -10
```

**파악 항목** (Q&A 자동 추론에 사용):
- **라이브러리 시그널**: 번들러(tsup/rollup/...) + `main`/`module`/`exports` 진입점 + `peerDependencies` 존재 → **라이브러리 확정**
  - 시그널 부족하면 (예: `bin` 필드만 있고 번들러 없음) "이 프로젝트가 라이브러리가 맞는지" 사용자에게 먼저 확인 후 진행. 라이브러리 아니면 본 하네스 부적합 — `claude-code-harness` 사용 권장
- **라이브러리 카테고리 힌트**: Radix·class-variance-authority·tokens.css → **디자인시스템** / 단순 utils만 → **유틸 라이브러리** / API 클라이언트 패턴 → **SDK**
- **시각 검증 도구**: Storybook 설치 여부
- **린터·포매터**: biome / ESLint+Prettier
- **버전 관리**: Changesets / 수동
- **컴포넌트 디렉토리**: `src/components/<카테고리>/<컴포넌트>/` 패턴
- **모노레포 여부**: 현재·상위 `package.json` `workspaces` 키 또는 `pnpm-workspace.yaml`

---

## Step 2 — Q&A

Step 1에서 자동 감지된 결과를 사용자에게 먼저 제시하고, **불명확하거나 사용자 입력이 필요한 항목만** 묻는다. 한 번에 모아서.

```
[design-harness 라이브러리 초기화 Q&A]

Q1. 이 라이브러리 이름을 뭐로 할까요?
    → docs 폴더명·에이전트 파일명에 사용됩니다.
    → package.json `name` 필드와 다를 수 있음 (예: scope 제거: `@user/ui-kit` → `ui-kit`)

Q2. 라이브러리 카테고리는?
    → Step 1 자동 감지 힌트를 먼저 제시. 예: "Radix + cva + tokens.css 감지 → 디자인시스템으로 추정. 맞나요?"
    A) 디자인시스템 — 컴포넌트 + 토큰 + variant
    B) 유틸 라이브러리 — 함수·훅·클래스 모음
    C) SDK — 외부 서비스 클라이언트
    D) 기타 — 자유 입력

    답에 따라 design-system.md 강조도 분기:
    - A 디자인시스템 → 핵심 문서. 토큰 구조·variant 패턴 상세 작성
    - B 유틸 → design-system.md 생략, conventions.md 강화
    - C SDK → design-system.md 생략, agent/architecture.md에 외부 서비스 연동 섹션 강조
    - D → 사용자 입력에 따라 결정

Q3. CLAUDE.md를 git 추적할까요, 개인 설정으로 할까요?
    A) 팀 공유 / OSS 공개 — git 추적 (OSS 라이브러리에 권장)
    B) 개인 설정 — gitignore

Q4. 커스텀 에이전트를 어디에 둘까요?
    A) 전역 (~/.claude/agents/) — 여러 라이브러리에서 재사용
    B) 프로젝트 (.claude/agents/) — 이 라이브러리 전용 (OSS contributor 공유용)

Q5. 브랜치 전략은?
    → Step 1 git log·branch 분석 결과를 먼저 제시.
    A) 트렁크 기반 (main 단독, 1인 OSS·소규모)
    B) GitHub Flow (main + feature 브랜치)
    C) Git Flow (develop + main 분리, release/hotfix)
    D) 커스텀

    확인 항목: base 브랜치 / feature 브랜치 네이밍 / PR 대상

Q6. Claude Code가 접근해야 할 외부 경로가 있나요?
    → 이 프로젝트 폴더 바깥 디렉토리를 Claude Code가 읽어야 하는 경우만 입력.
      해당 없으면 "없음".
    → install.sh 사용 시 design-harness 폴더는 자동 등록됨.

Q7. 커밋 컨벤션은?
    → Step 1 git log 분석 결과 우선 제시.
    A) 컨벤셔널 커밋 (`feat:`, `fix:`, `docs:`, ...) — 라이브러리에 권장
    B) 자유 형식
    C) 커스텀 (예: `[FE]` 등 영역 브래킷)

Q8. 이슈 트래커가 있나요?
    A) GitHub Issues — `Closes #이슈번호` 형식
    B) Linear / Jira — 별도 체계
    C) 없음 — 1인 OSS·초기 단계. 이슈 번호 없이 커밋 허용

Q9. 시각 검증 도구는?
    → Step 1 Storybook 감지 결과 우선 제시.
    A) Storybook (라이브러리에 권장)
    B) Ladle / Histoire 등 다른 도구
    C) 없음 — 단위 테스트로만 검증
```

> **Q&A 모호한 답변 규칙**: "그냥 기본으로", "잘 모르겠어요" → 재질문 1회 후 기본값 적용 + 사용자에게 명시.
> 기본값 없이 임의 추정 금지.

---

## Step 3 — docs/ 구조 생성

**기존 docs/ 구조가 있는 경우 — 사용자에게 명시적으로 묻는다:**

```
docs/ 디렉토리에 기존 자료가 있습니다:
[ls -la docs/ 결과 표시]

어떻게 처리할까요?
(a) 그대로 두고 docs/<라이브러리>/ 만 추가 (가장 보수적)
(b) docs/<라이브러리>/history/ 로 이관 — 회고 자산으로 보관
(c) 직접 정리 후 진행 — 사용자가 수동 정리 후 알려주세요
```

사용자 응답 전까지 docs/ 디렉토리를 수정하지 않는다.

**DOCS_BASE 결정 (`$BASE` 변수):**

| 프로젝트 형태 | BASE 값 |
|---|---|
| 단일 라이브러리 (레포 루트 실행) | `docs/<라이브러리>` |
| 모노레포 패키지별 (패키지 루트 실행) | `docs` |

**디렉토리 생성 — 라이브러리 전용 (`pages/` 대신 `components/`):**

```bash
SERVICE="<Q1 라이브러리명>"
BASE=docs/$SERVICE   # 또는 BASE=docs (모노레포 패키지별)

mkdir -p $BASE/{status,plans,history,agent,components,git-workflow}

touch $BASE/status/HANDOFF_NOW.md
touch $BASE/plans/HANDOFF.md
touch $BASE/history/세션_노트.md
touch $BASE/agent/README.md
touch $BASE/agent/architecture.md
touch $BASE/agent/conventions.md
touch $BASE/agent/design-system.md   # Q2=A(디자인시스템)·D(기타 토큰 사용)면 핵심 문서
touch $BASE/git-workflow/branch-commit.md
```

> Q2=B(유틸 라이브러리)·C(SDK)면 `design-system.md`는 비워두거나 생성 생략. 사용자에게 확인.

---

## Step 4 — CLAUDE.md 작성

**아래 파일을 Read로 먼저 읽어 라이브러리 전용 템플릿을 확인한 뒤 작성한다:**
`<design-harness경로>/Claude Code/템플릿/CLAUDE.md_템플릿.md`

> **하네스 경로 확인**:
> ```bash
> python3 -c "
> import json, pathlib
> s = pathlib.Path.home() / '.claude/settings.json'
> data = json.loads(s.read_text()) if s.exists() else {}
> dirs = data.get('permissions', {}).get('additionalDirectories', [])
> harness = [d for d in dirs if 'design-harness' in d]
> print('design-harness 경로:', harness[0] if harness else '(settings.json에 없음 — install.sh 재실행)')
> "
> ```

**기존 CLAUDE.md가 있는 경우**: 덮어쓰기 전에 사용자에게 (병합 / 대체 / 건너뜀) 선택 요청. 선택 전까지 파일 수정 금지.

**라이브러리 전용 CLAUDE.md 핵심 항목:**

- **프로젝트 유형 명시**: "React 컴포넌트 라이브러리" + 빌드 도구 + exports
- **0-A 작업 영역 판별 표** (한 행 = 한 코드 카테고리, 패키지 아님):
  - 컴포넌트 추가·수정·variant·prop → `<컴포넌트 디렉토리>/<컴포넌트>/`
  - 디자인 토큰 변경 → `<토큰 파일 경로>` (Q2=A인 경우)
  - 빌드·exports·릴리스 → `tsup.config.*` · `package.json` · `.changeset/`
  - 유틸 함수 → `src/utils/`
  - 스토리·예시 → `<storybook 디렉토리>/`
- **STEP 1 강제 규칙** (라이브러리 핵심):
  - **5개 산출물 동시 작성** (신규 컴포넌트): `<name>.tsx` + `<name>.test.tsx` + `index.ts` + `<name>.stories.tsx` + `components/<index>.ts` export 추가. 5개 채우지 않으면 미완료
  - **semantic 토큰만 사용** (Q2=A): raw hex(`#fabc37`) 작성 금지 (Q2=B/C는 생략)
  - **`components/<index>.ts` export 갱신 누락 금지**: 신규 컴포넌트 export 추가 후 빌드 재실행
  - **이슈 번호 없이 커밋 금지** (Q8=C면 이 규칙 비활성화 — 컨벤셔널 커밋만 적용)
  - **base 브랜치 직접 커밋 금지** (Q5 답변)
- **STEP 2 검증 체크**:
  - `<테스트 명령>` 통과
  - `<빌드 명령>` 통과 (dist 갱신)
  - `<린트 명령>` 통과
  - 시각 검증: `<storybook 명령>` (Q9=A인 경우)
- **agent/ 문서 갱신 트리거**:
  - architecture.md — 빌드 시스템 변경 / peerDependencies 변경 / 새 디렉토리 추가
  - conventions.md — 린터 설정 변경 / 새 컨벤션 결정
  - design-system.md — 디자인 토큰 추가·변경 / 새 variant 패턴 (Q2=A)
- **서브에이전트 호출 규칙**:
  - 컴포넌트·토큰 작업 → `<라이브러리>-dev`
  - 문서 갱신 → `<라이브러리>-doc-writer`
  - 코드 리뷰 → `code-reviewer`
  - 버그 추적 → `debugger`
  - 테스트 작성 → `test-writer`

---

## Step 5 — agent/ 컨텍스트 문서 작성

**아래 파일을 Read로 먼저 읽어 라이브러리 전용 템플릿을 확인한 뒤 작성한다:**
`<design-harness경로>/Claude Code/템플릿/agent_문서_템플릿.md`

**README.md** — 에이전트 컨텍스트 인덱스. Step 6에서 에이전트 추가 후 갱신.

**architecture.md** — 라이브러리 구조 (앱과 다른 차원):
- **타입**: 라이브러리 (npm publish 대상)
- **빌드 도구**: tsup / rollup / microbundle 등 + 출력 포맷(ESM/CJS/dual)
- **exports 진입점**: `package.json` `exports` 필드 표
- **peerDependencies / dependencies** 분류
- **디렉토리 구조**: 컴포넌트 단위 (`src/components/<카테고리>/`)
- **컴포넌트 패턴**: forwardRef 사용 기준 (Radix 래핑 시 / 단순 함수 시), variant 시스템 (cva 등)
- **빌드·배포 흐름**: 개발 watch → 테스트 → 린트 → 시각 검증 → changeset → release

**conventions.md** — 린터·포매터 + 라이브러리 특화:
- 파일 네이밍 (kebab-case 등) / 컴포넌트 네이밍 (PascalCase)
- 컴포넌트 파일 내 선언 순서 (cva → interface → 함수 → displayName → 하위 컴포넌트 → export)
- 테스트·스토리 파일 패턴

**design-system.md** (Q2=A 핵심 문서):
- 디자인 토큰 구조 (core / semantic 등 계층)
- core 카테고리 표 (brand / neutral / semantic 색상 등)
- semantic 토큰 표 (text / bg / border / interactive 등)
- 컴포넌트 작업 원칙 (하드코딩 금지·임의 px 금지)
- variant 패턴 (cva 등) + variant 추가 체크리스트
- 새 토큰 추가 절차 (core 추가 → semantic 매핑 → 빌드 → grep 영향 검토 → 이 문서 갱신)
- 자주 사용하는 토큰 빠른 참조

> Q2=B(유틸)·C(SDK)면 design-system.md 생략. agent/README.md 인덱스에서도 제외.

---

## Step 6 — 커스텀 에이전트 정의

Q4 답변 위치(`~/.claude/agents/` 또는 `.claude/agents/`)에 아래 에이전트를 생성한다.
**아래 파일을 Read로 먼저 읽는다:**
`<design-harness경로>/Claude Code/에이전트_정의_가이드.md`

**필수 (항상 생성):**

- **`<라이브러리>-dev.md`** — 컴포넌트·토큰 작성 전담
  - 작업 전 필수: HANDOFF_NOW + agent/ 3종 + 유사 컴포넌트 2개 분석
  - 작업 규칙 — 5개 산출물 / semantic 토큰만 / forwardRef 패턴 일관 / cva variants 키 통일
  - 디자인 토큰 변경 시: core+semantic 함께 갱신 → 빌드 → grep → design-system.md 갱신
  - 검증: 테스트·빌드·린트·시각(Storybook)

- **`<라이브러리>-doc-writer.md`** — 문서 갱신 전담
  - 담당: HANDOFF 3종 + agent/ 3종 (트리거 충족 시) + 컴포넌트 정리 문서 (선택)
  - 갱신 순서: HANDOFF_NOW → 세션_노트 → HANDOFF.md
  - HANDOFF_NOW 항상 60줄 이하 유지

> 라이브러리는 페이지·UI 검증이 없으므로 `playwright-validator` 에이전트 생성하지 않는다.

---

## Step 7 — settings.json 설정

### 7-A. additionalDirectories (Q6 답변 기반)

Q6에서 외부 경로를 받은 경우만 실행. "없음"이면 건너뜀.

```bash
python3 - << 'EOF'
import json, pathlib
path = pathlib.Path.home() / ".claude" / "settings.json"
data = json.loads(path.read_text()) if path.exists() else {}
data.setdefault("permissions", {}).setdefault("additionalDirectories", [])
vault = "<Q6 답변 경로>"
if vault not in data["permissions"]["additionalDirectories"]:
    data["permissions"]["additionalDirectories"].append(vault)
path.write_text(json.dumps(data, indent=2, ensure_ascii=False))
print("완료:", path)
EOF
```

### 7-B. 훅 스크립트 — 라이브러리에 부적합 → 자동 스킵

> 본체 `claude-code-harness`의 페이지 자동 감지 훅(`check-source-doc.sh` 등)은 `pages/<도메인>/` 패턴 기반이라 라이브러리에 부적합. design-harness는 훅을 사용하지 않는다.
>
> **컴포넌트 단위 자동 감지가 필요하다면** 별도로 customize한 훅을 작성해 사용자가 수동 설치할 수 있다. 본 하네스는 기본 제공하지 않는다.

이 Step은 안내만 출력하고 다음 Step으로 진행:
```
ℹ design-harness는 페이지 자동 감지 훅을 사용하지 않습니다.
   라이브러리는 컴포넌트 단위 도메인이라 페이지 훅 패턴이 부적합합니다.
   컴포넌트 단위 자동 감지가 필요하면 직접 훅을 작성해 ~/.claude/hooks/ 에 설치하세요.
```

---

## Step 8 — 메모리 초기화

```bash
PROJECT_PATH=$(pwd | sed 's|/|-|g')
echo "메모리 경로: ~/.claude/projects/${PROJECT_PATH}/memory/"
ls ~/.claude/projects/${PROJECT_PATH}/memory/ 2>/dev/null || echo "(첫 세션 후 자동 생성)"
```

> 메모리 폴더는 Claude Code가 첫 세션 시 자동 생성. 수동 mkdir 불필요.

MEMORY.md가 없으면 최소 구조로 생성:
```markdown
# Memory Index
## Feedback
## Reference
```

Q&A 결과를 저장:
- **작업 규칙**: `feedback_*.md` (예: `feedback_5_artifacts.md` — "신규 컴포넌트는 5개 산출물 동시 작성")
- **외부 리소스**: `reference_*.md` (예: `reference_storybook.md` — "시각 확인은 pnpm storybook localhost:6006")

---

## Step 9 — HANDOFF 3종 + git-workflow 작성

**HANDOFF_NOW.md** (`$BASE/status/HANDOFF_NOW.md`):
**아래 파일을 Read로 먼저 읽는다:** `<design-harness경로>/Claude Code/템플릿/HANDOFF_NOW_템플릿.md`

라이브러리 전용 §1 항목:
- 브랜치 / 활성 이슈 / **빌드 명령** / **테스트 명령** / **시각 검증 명령** (Q9=A)
- 아키텍처 한줄: "<라이브러리명> — <빌드도구> <ESM/CJS> dual 빌드, dist/ 산출물을 외부 앱이 소비. <컴포넌트 디렉토리> 단위로 N개 컴포넌트."
- 주의: "신규 컴포넌트는 5개 산출물 동시 작성" + "semantic 토큰만 사용" (Q2=A)

**HANDOFF.md** (`$BASE/plans/HANDOFF.md`):
**아래 파일을 Read로 먼저 읽는다:** `<design-harness경로>/Claude Code/템플릿/HANDOFF_템플릿.md`

```markdown
## Session Update <오늘> (하네스 초기 구축)

### 변경 파일
- `CLAUDE.md` — 라이브러리 진입점
- `docs/<라이브러리>/agent/` — README, architecture, conventions, design-system (Q2=A인 경우)
- `docs/<라이브러리>/git-workflow/branch-commit.md`
- `<에이전트 위치>/` — <라이브러리>-dev, <라이브러리>-doc-writer

### 이슈
- 하네스 초기 구축 완료
```

**세션_노트.md** (`$BASE/history/세션_노트.md`):
**아래 파일을 Read로 먼저 읽는다:** `<design-harness경로>/Claude Code/템플릿/세션_노트_템플릿.md`

**git-workflow/branch-commit.md** (`$BASE/git-workflow/branch-commit.md`):
**아래 파일을 Read로 먼저 읽는다:** `<design-harness경로>/Claude Code/템플릿/git_워크플로우_템플릿.md`
Q5·Q7·Q8 답변으로 채운다. 라이브러리 단일 패키지면 영역 브래킷(`[FE]`/`[BE]`) 미사용.

**(Q2=A 디자인시스템) 첫 컴포넌트 정리 문서 — 선택 사항**:
복잡한 컴포넌트(예: DnD·차트·복합 폼)가 이미 있고 별도 문서가 필요하면 사용자 확인 후 작성.
**아래 파일을 Read로 먼저 읽는다:** `<design-harness경로>/Claude Code/템플릿/컴포넌트_정리_템플릿.md`

> 단순 컴포넌트(버튼·뱃지 등)는 Storybook 스토리·테스트 코드가 1차 문서 역할.
> design-system.md에 패턴이 정리돼 있으므로 컴포넌트별 정리 문서를 강제하지 않는다.

> **(라이브러리 공통) ui-verification.md 생성하지 않음** — Storybook이 시각 검증 도구이고, 컴포넌트 단위는 단위 테스트(vitest 등)로 충분. agent/README.md 인덱스에도 ui-verification.md 항목 추가하지 않는다.

---

## Step 10 — 검증

**산출물 체크:**

```bash
SERVICE="<Q1 라이브러리명>"
AGENT_DIR="<Q4 경로>"   # ~/.claude/agents/ 또는 .claude/agents/
BASE=docs/$SERVICE        # 모노레포 패키지별이면 docs

echo "=== 산출물 체크 ==="
echo "[1] CLAUDE.md"
[ -f CLAUDE.md ] && echo "  ✓ ($(wc -l < CLAUDE.md)줄)" || echo "  ✗ 없음"

echo "[2] docs/<라이브러리>/ 구조"
for f in status/HANDOFF_NOW.md plans/HANDOFF.md history/세션_노트.md \
         agent/README.md agent/architecture.md agent/conventions.md \
         git-workflow/branch-commit.md; do
  [ -f "$BASE/$f" ] && echo "  ✓ $f" || echo "  ✗ $f (없음)"
done
# Q2=A이면 design-system.md도 필수
[ -f "$BASE/agent/design-system.md" ] && echo "  ✓ agent/design-system.md (Q2=A)"

echo "[3] 에이전트 (필수 2개)"
for a in $SERVICE-dev.md $SERVICE-doc-writer.md; do
  [ -f "$AGENT_DIR/$a" ] && echo "  ✓ $a" || echo "  ✗ $a"
done

echo "[4] HANDOFF_NOW 60줄 이하"
[ -f $BASE/status/HANDOFF_NOW.md ] && \
  L=$(wc -l < $BASE/status/HANDOFF_NOW.md) && \
  if [ "$L" -le 60 ]; then echo "  ✓ ${L}줄"; else echo "  ⚠ ${L}줄 (60 초과)"; fi
```

✗ 누락 항목 있으면 해당 Step으로 돌아가 보완 후 재실행.

**라이브러리 빌드·테스트 동작 확인:**
```bash
# Step 1에서 감지한 실제 명령으로 실행
pnpm build && echo "✓ build OK" || echo "✗ build 실패"
pnpm test && echo "✓ test OK" || echo "✗ test 실패"
pnpm lint && echo "✓ lint OK" || echo "✗ lint 실패"
```

**CLAUDE.md 동작 확인:**
```
이 프로젝트 작업 시작할게
```
→ 에이전트가 자동으로 HANDOFF_NOW.md를 읽고 §1·§2 요약 출력하면 정상.

**gitignore 항목 (Q3=B 개인 설정인 경우):**
```gitignore
CLAUDE.md
```

공통:
```gitignore
.claude/settings.local.json
*.skill
```

---

## Step 11 — 완료 보고

```
[Project Init (Library) 완료]

생성 파일:
- CLAUDE.md
- docs/<라이브러리>/status/HANDOFF_NOW.md
- docs/<라이브러리>/plans/HANDOFF.md
- docs/<라이브러리>/history/세션_노트.md
- docs/<라이브러리>/agent/README.md
- docs/<라이브러리>/agent/architecture.md
- docs/<라이브러리>/agent/conventions.md
- docs/<라이브러리>/agent/design-system.md  (Q2=A 디자인시스템인 경우)
- docs/<라이브러리>/git-workflow/branch-commit.md
- <에이전트 위치>/<라이브러리>-dev.md
- <에이전트 위치>/<라이브러리>-doc-writer.md

스킵된 항목 (라이브러리 특성):
- Step 7-B 훅 스크립트 (페이지 패턴 부적합)
- ui-verification.md (Storybook 갈음)
- pages/<도메인>/FE_<기능>.md (components/ 디렉토리에 컴포넌트 단위 정리, 선택)

다음 단계:
1. CLAUDE.md 내용 검토 후 라이브러리 컨벤션에 맞게 조정
2. 첫 작업 시작 (예: 컴포넌트 추가)
3. 5개 산출물 동시 작성 워크플로우 체험
4. 30일 후 design-system.md·HANDOFF 누적 자산 점검
```
