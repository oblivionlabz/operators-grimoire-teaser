---
name: agents-md
description: Scaffold repo-root AGENTS.md (the Dec-2025 Linux Foundation / AAIF standard read by every coding agent) and symlink CLAUDE.md to it. Pass stack type or 'detect'. Warns on .cursorrules/GEMINI.md duplication.
allowed-tools: Bash Read Write Edit Glob
argument-hint: [detect|nextjs|go|rust|python|android|agent-app]
---

Scaffold AGENTS.md in: **$(pwd)**  (stack: **$ARGUMENTS**)

```!
STACK="$ARGUMENTS"
[ -z "$STACK" ] && STACK="detect"

ROOT="$(git rev-parse --show-toplevel 2>/dev/null || pwd)"
cd "$ROOT" || exit 1
echo "Repo root: $ROOT"
echo "Stack arg: $STACK"

# Detect stack if requested
if [ "$STACK" = "detect" ]; then
  [ -f package.json ] && STACK="nextjs"
  [ -f go.mod ] && STACK="go"
  [ -f Cargo.toml ] && STACK="rust"
  [ -f pyproject.toml ] || [ -f setup.py ] && STACK="python"
  [ -f build.gradle.kts ] || [ -f settings.gradle.kts ] && STACK="android"
  [ -f claude_agent.toml ] || [ -d src/subagents ] && STACK="agent-app"
  [ "$STACK" = "detect" ] && STACK="nextjs"  # fallback
  echo "Detected: $STACK"
fi

# Warn on duplicates
for f in .cursorrules GEMINI.md .windsurfrules .clinerules .aiderrules; do
  [ -f "$f" ] && echo "WARN: $f exists — will duplicate rules with AGENTS.md. Consider symlinking $f → AGENTS.md after writing."
done
```

## Phase 1 — Why AGENTS.md

`AGENTS.md` at repo root is the vendor-neutral README for coding agents (Cursor, Claude Code, Copilot, Gemini CLI, Codex, Windsurf, Cline, Aider…). Dec-2025 Linux Foundation / AAIF announcement put it on a formal standards track; GitHub blog reported 2,500+ repos had adopted it within weeks of launch. One file, every agent reads it, no per-vendor fragmentation.

**The rule:** one AGENTS.md. Vendor-specific files (`CLAUDE.md`, `.cursorrules`, `GEMINI.md`) should either (a) not exist, or (b) be a thin shim that `@include`s AGENTS.md plus a few vendor-only overrides.

## Phase 2 — Recommended 2026 section set

Every AGENTS.md should have these sections. Keep each tight — agents scan, they don't read essays.

1. **Project overview** — 3-5 lines on what this repo is and who runs it.
2. **Build / test / lint commands** — the exact CLI one-liners. Agents run these; don't make them guess.
3. **Code style** — formatter authority, linter authority, any non-default choices and *why*.
4. **Architecture notes** — load-bearing files, directory layout, cross-cutting patterns (event bus, auth middleware, state store).
5. **Security notes** — what never goes in a commit (secrets, PII), where to load them from, threat model in one sentence.
6. **Commit / PR conventions** — conventional commits? branch naming? required reviewers? PR template?
7. **Deploy steps** — how a change reaches prod (CI pipeline name, manual gates, rollback command).
8. **Agent-specific gotchas** — things agents have gotten wrong here before. Living section, append-only.

## Phase 3 — Write the file (stack-tailored)

Write `AGENTS.md` at repo root. Template below; swap the `[STACK:*]` blocks for the detected stack. Keep real commands, not placeholders.

```markdown
# AGENTS.md — <project-name>

> Vendor-neutral agent contract per the AGENTS.md spec (agents.md). Every coding agent — Claude Code, Cursor, Copilot, Codex, Gemini CLI, Windsurf, Cline, Aider — should read this file first.

## Project overview

<3-5 lines. What does this repo do, who owns it, current stage (prototype / prod / maintenance).>

## Build / test / lint

[STACK:nextjs]
```
pnpm install
pnpm dev            # local dev server on :3000
pnpm build
pnpm test           # vitest
pnpm lint           # biome check
pnpm typecheck      # tsc --noEmit
```

[STACK:go]
```
go mod download
go build ./...
go test ./... -race -count=1
golangci-lint run
```

[STACK:rust]
```
cargo build
cargo test --all-features
cargo clippy -- -D warnings
cargo fmt --check
```

[STACK:python]
```
uv sync
uv run pytest -x
uv run ruff check .
uv run mypy src/
```

[STACK:android]
```
./gradlew build
./gradlew test
./gradlew connectedAndroidTest   # requires emulator/device
./gradlew lint
./gradlew ktlintCheck
```

[STACK:agent-app]
```
uv sync
uv run python -m src              # smoke run
uv run pytest -x
# Cache-hit assertion: cache_read_input_tokens ≥ 0.7 × total (see tests/test_cache.py)
```

## Code style

- Formatter authority: <prettier | gofmt | rustfmt | ruff format | ktlint> — CI rejects unformatted diffs.
- Linter authority: <biome | golangci-lint | clippy -D warnings | ruff | detekt>.
- Non-default choices: <e.g., "no default exports in TS", "errors.As everywhere, never errors.Is for wrapped", "no `unwrap()` in non-test code">.

## Architecture

- Entry point: `<path/to/main>`
- State: `<where app state lives — Redux / Zustand / context / global singleton / none>`
- Auth: `<middleware location + pattern>`
- DB: `<Postgres / SQLite / Mongo — ORM or raw — migration tool>`
- Observability: `<OpenTelemetry / Datadog / Sentry / stdout-only>`

## Security

- Secrets load from `<OpenBao / 1Password / AWS SM / .env.local>` — never committed.
- PII never logged. Use `redact(obj)` helper from `<path>` before any log call that touches user data.
- Threat model (one line): `<e.g., "multi-tenant SaaS, tenant-ID isolation is the #1 invariant — every DB query must include tenant_id in the WHERE clause">`.

## Commits & PRs

- Conventional commits (`feat:`, `fix:`, `chore:`, `refactor:`, `test:`, `docs:`).
- Branch naming: `<user>/<short-slug>`.
- PR ceiling: <400 lines diff; anything larger gets split or explained in the description>.
- Required reviewers: `<CODEOWNERS / default reviewer team>`.

## Deploy

- CI: `<GitHub Actions workflow name>` — must be green.
- Prod deploy: `<command or "auto on merge to main" or "manual via Argo">`.
- Rollback: `<command>`.

## Agent gotchas (living section)

<!-- Append new gotchas as they're discovered. Keep them specific. -->

- <e.g., "The /api/billing route uses Stripe's live webhook secret in dev. Do not regenerate webhook secrets casually — it breaks the dev loop for everyone.">
```

## Phase 4 — Wire CLAUDE.md to AGENTS.md

Two strategies. Pick one.

**Strategy A — pure symlink (no Claude-specific overrides needed):**

```bash
[ -f CLAUDE.md ] && mv CLAUDE.md CLAUDE.md.bak.$(date +%s)
ln -s AGENTS.md CLAUDE.md
git add AGENTS.md CLAUDE.md
```

**Strategy B — thin override shim (Claude gets extra rules on top):**

```bash
cat > CLAUDE.md <<'MD'
<!-- Vendor override for Claude Code. The canonical agent contract is AGENTS.md. -->

@include AGENTS.md

## Claude-specific

- Default effort for this repo is `xhigh` on Opus 4.7 — agentic refactors need the headroom.
- Prefer the `Explore` subagent for any search spanning >3 files; keeps main context lean.
- When a rule in AGENTS.md conflicts with this file, AGENTS.md wins — this file only *adds*.
MD
git add AGENTS.md CLAUDE.md
```

Claude Code resolves `@include <path>` by inlining the target file's content, so Strategy B gives you Claude-only additions without forking the vendor-neutral contract.

## Phase 5 — De-duplicate vendor files

If the warnings in preflight flagged existing `.cursorrules` / `GEMINI.md` / `.windsurfrules`, offer to collapse them:

```bash
# Option: symlink all vendor files to AGENTS.md
for f in .cursorrules GEMINI.md .windsurfrules .clinerules .aiderrules; do
  if [ -f "$f" ] && [ ! -L "$f" ]; then
    mv "$f" "$f.bak.$(date +%s)"
    ln -s AGENTS.md "$f"
    echo "Symlinked $f → AGENTS.md (backup at $f.bak.*)"
  fi
done
```

Or, if those files contained vendor-unique content you want to keep, fold the useful bits into AGENTS.md first and delete the rest.

## Phase 6 — Verify

```bash
ls -la AGENTS.md CLAUDE.md 2>/dev/null
head -20 AGENTS.md
# Commit:
git add AGENTS.md CLAUDE.md
git status
```

## Refs

- AGENTS.md spec: `agents.md`
- Linux Foundation / AAIF announcement (Dec 2025): `agents.md/announcement`
- GitHub blog on 2,500+ adopting repos: `github.blog` (search "AGENTS.md")
- Pairs with: `/sdd`, `/constitution` (which lives at `.specify/memory/constitution.md` and is referenced from AGENTS.md's Architecture or Security section)
