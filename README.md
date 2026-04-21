# Grimoire — Free Sample

Three free skills from [**The Claude Code Operator's Grimoire**](https://grimoire.example.com) — a curated bundle of fourteen skills that turns Claude Code into a disciplined senior operator.

## What's in this repo

- **`adr/`** — Write Architecture Decision Records in the Michael Nygard template (Context → Decision → Status → Consequences), numbered and committed in-tree.
- **`agents-md/`** — Scaffold the 2025 Linux Foundation `AGENTS.md` standard every coding agent reads first.
- **`constitution/`** — Author the project-wide contract that every agent and teammate follows.

These three skills ship free. The full bundle — spec-driven dev chain, orchestration topologies, evals harness, obsidian vault, claude-agent SDK scaffold, and nine more — is $99 one-time at **[gumroad.com/l/operators-grimoire](https://gumroad.com/l/operators-grimoire)**.

## Install

```bash
git clone https://github.com/oblivionlabz/grimoire-teaser.git
cp -r grimoire-teaser/skills/* ~/.claude/skills/
# restart Claude Code or /reload
```

Verify:

```bash
ls ~/.claude/skills/ | grep -E '^(adr|agents-md|constitution)$'
```

## Usage

```bash
/agents-md       # scaffold AGENTS.md in the current repo
/constitution    # author .specify/memory/constitution.md
/adr choose-postgres-over-sqlite   # write docs/adr/0001-choose-postgres-over-sqlite.md
```

## Why free?

Because curation beats completeness. These three establish the documentation lane. The paid eleven run the execution lane — spec-driven development, scaffolding, orchestration, evals, and the vault. If the free three earn a spot in your workflow, the rest will pay for themselves in the week you save.

## License

MIT. Use, modify, redistribute — your call. Attribution appreciated but not required.

## Full bundle

- **What:** 14 skills, one installer, a 5-page operator guide PDF.
- **Who:** Senior developers shipping real software with Claude Code.
- **Price:** $99 one-time. No subscription. No telemetry. No support tier gating.
- **Get it:** [gumroad.com/l/operators-grimoire](https://gumroad.com/l/operators-grimoire)

Built by an operator.
