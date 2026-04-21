---
name: constitution
description: Author or revise .specify/memory/constitution.md — the project-wide constraints agents follow. Uses Addy Osmani's three-tier shape — ✅ Always / ⚠️ Ask-first / 🚫 Never. Pass a project brief or 'update' to revise.
allowed-tools: Read Write Edit
argument-hint: <project-brief|update>
---

Write / revise project constitution from: **$ARGUMENTS**

## Phase 1 — Detect mode

Check for an existing constitution:

```
.specify/memory/constitution.md
```

- If it exists **and** `$ARGUMENTS` == `update` → **revise mode**: read the current file, ask the user which rules to add/drop/promote/demote, then Edit in place.
- If it exists and `$ARGUMENTS` != `update` → warn; either the user wants a fresh rewrite (confirm first) or they meant to pass `update`.
- If it's missing → **greenfield mode**: scaffold from scratch using the brief.

If `.specify/memory/` does not exist, stop and direct the user to `/spec-kit-init` first.

## Phase 2 — Probe the user (greenfield)

Before writing anything, ask the user a terse checklist (one question per line, wait for answers):

1. **Stack** — primary language(s), framework(s), runtime targets?
2. **Testing posture** — TDD, test-after, integration-heavy, what's the minimum bar before merge?
3. **Security constraints** — secrets handling, authz model, data classification, any compliance regime (PCI / HIPAA / SOC2 / IL state records)?
4. **Deploy gates** — who approves prod? CI required green? Manual smoke? Canary?
5. **AI guardrails** — things you've seen agents do wrong on this codebase that must be ruled out (e.g., "no mocking DB in integration tests", "never disable auth middleware even in dev", "no auto-generated migrations without review")?
6. **Style** — formatter / linter authority? Commit conventions? PR size ceiling?

Take their answers as source material. Do **not** invent rules they didn't confirm.

## Phase 3 — Why, not just what (Anthropic Claude Constitution pattern)

Anthropic's Jan 2026 Claude Constitution (`anthropic.com/news/claude-new-constitution`) moved from flat rule lists to *explanations for each rule*. Mirror that:

- Each rule gets a one-line **rationale** — the failure mode it prevents.
- The agent obeys better when it understands *why*, and a reviewer can tell when a rule has outlived its context.

## Phase 4 — Write in the 3-tier shape

Template (Write to `.specify/memory/constitution.md`):

```markdown
# <Project Name> — Constitution

**Version:** 1.0 · **Last updated:** <YYYY-MM-DD>

The rules every agent and contributor on this project follows. Three tiers:
- ✅ **Always** — non-negotiable defaults. Do these without asking.
- ⚠️ **Ask-first** — gray zones. Pause and confirm with a human.
- 🚫 **Never** — hard stops. Do not cross these, even if a ticket asks you to.

Each rule has a one-line rationale. When a rule no longer protects what it claims to, amend it.

---

## ✅ Always

- **<rule>.** <rationale — the failure mode this prevents.>
- …

## ⚠️ Ask-first

- **<rule>.** <rationale — why a human needs to weigh in.>
- …

## 🚫 Never

- **<rule>.** <rationale — the blast radius if this is crossed.>
- …

---

## Amendments

Any amendment must cite the incident / PR / decision that motivated the change. Log amendments at the bottom with date + author.

- <YYYY-MM-DD> — initial constitution authored.
```

## Phase 5 — Exemplar (Next.js + FastAPI dashboard with LLM agents)

Use this only as a shape reference. Tailor to the user's actual answers.

```markdown
## ✅ Always

- **Every route handler validates input with Zod (frontend) or Pydantic (backend).** Unvalidated input is the #1 entry point for XSS/SSRF/SQLi — never trust shape.
- **Every DB write runs in a transaction.** Partial writes on crash are worse than no write.
- **LLM calls log prompt + response + token counts to our observability sink.** Without this, cost + correctness regressions are invisible.
- **Every PR ships with at least one integration test touching a real Postgres (testcontainers).** Unit tests with mocked DBs lie.
- **Secrets load from OpenBao at runtime, never from .env in repo.** A single leaked .env is a breach.

## ⚠️ Ask-first

- **New third-party dependency.** Supply-chain risk; a human decides if the value beats the audit cost.
- **Schema migration.** Production data model changes need a rollback plan reviewed by a second pair of eyes.
- **New LLM tool/MCP with network egress.** Expands the agent's blast radius.
- **Public API surface change (added/removed/renamed endpoint).** Downstream consumers need a deprecation window.

## 🚫 Never

- **Never mock the database in an integration test.** If it runs against a mock, it's a unit test — rename it.
- **Never disable auth middleware, CSRF, or rate limits "just for dev".** Dev flags migrate to prod more often than anyone admits.
- **Never commit an API key, private key, token, or plaintext credential.** Rotate immediately if one lands; do not "fix in next commit".
- **Never let an LLM agent run `rm -rf`, `git push --force`, `DROP TABLE`, or send outbound messages unattended.** Destructive blast radius + no undo.
- **Never ship a migration without a reverse migration.** `ALTER TABLE … DROP COLUMN` on prod without a rollback path is how Fridays get ruined.
```

## Phase 6 — Final checks before handoff

- Every rule has a rationale. No bare imperatives.
- Each rule lives in exactly one tier. If it feels like it belongs in two, split it.
- The 🚫 list is short. If it's longer than ⚠️, you've mis-categorized — most rules are gray-zone.
- Nothing contradicts `~/.claude/CLAUDE.md` destructive-action guardrails. This file *extends* them, not replaces them.

## Refs

- Addy Osmani, *What makes a good spec?*: `addyosmani.com/blog/good-spec/` (origin of the 3-tier shape for agent constraints)
- Anthropic, *Claude's new constitution* (Jan 2026): `anthropic.com/news/claude-new-constitution` (the "explain the why" pattern)
- Pairs with: `/sdd`, `/spec-kit-init`, `/agents-md`
