# design-harness

Claude Code harness for **React component libraries / design systems** (npm publish targets, `dist/` consumed by external apps).

> Sister project of [`claude-code-harness`](https://github.com/kyungjongKim/claude-code-harness) (the app-focused harness).
> Use this harness when your project is a library — not a Next.js / Vite app.

For Korean documentation see [README.ko.md](./README.ko.md).

---

## Why a separate harness?

App harness assumes pages, routes, UI verification (Playwright), `pages/<domain>/FE_*.md` docs.
Library projects don't have those — they have:

- Components / modules (not pages)
- `dist/` build output via tsup / rollup / microbundle
- `peerDependencies` (not full app deps)
- Storybook for visual verification (not Playwright)
- Design tokens (core / semantic layers) instead of app state

Forcing one harness onto both creates noise. This harness ships only what library work needs.

---

## Quick start

```bash
# 1. Clone next to the harness
git clone https://github.com/<owner>/design-harness.git ~/projects/design-harness

# 2. Install (project-local, recommended for OSS libraries)
cd <your-library-project>
bash ~/projects/design-harness/install.sh --local

# 3. Bootstrap the project
claude
> /project-init
```

`/project-init` will analyze your library (build tool, peerDeps, exports), ask a few questions, and generate `CLAUDE.md`, `docs/<library>/`, custom agents, and Git workflow docs tailored to library work.

## Verify

After installation, confirm the skills are in place:

```bash
# Check skills installed
ls .claude/skills/ | grep -E "project-init|project-fix|session-close|document-review"

# Confirm CLAUDE.md was created (after /project-init)
wc -l CLAUDE.md

# GitHub CLI auth (required for /project-fix, /project-pr, /project-issue)
gh auth status
```

---

## What you get

- `CLAUDE.md` — STEP 0~3 enforcement loop tuned for component / token / build work
- `docs/<library>/agent/` — `architecture.md` (build / exports / peerDeps), `conventions.md`, `design-system.md`
- `.claude/agents/<library>-dev.md` + `<library>-doc-writer.md` — component & doc agents that follow your existing patterns
- `docs/<library>/components/` — per-component docs (replaces `pages/`)
- Git workflow with conventional commits + optional issue tracker (1-person OSS friendly)

---

## Skills

| Skill | Purpose |
|---|---|
| `/project-init` | Bootstrap a library project (this is what you run first) |
| `/session-close` | End-of-session HANDOFF update (3 files in order) |
| `/project-fix` | Bug / QA issue → sub-issue + branch prep |
| `/project-pr` | Create PR with issue link + Co-Authored-By |
| `/project-issue` | Interactive GitHub issue creation |
| `/document-review` | Scenario-based doc audit |

---

## License

MIT — see [LICENSE](./LICENSE).
