---
name: adr
description: Write an Architecture Decision Record into docs/adr/NNNN-<slug>.md following the Michael Nygard template (Context, Decision, Status, Consequences). Pass a decision title + optional status. Auto-increments NNNN.
allowed-tools: Bash Read Write Glob
argument-hint: <title> [status]
---

ADR for: **$ARGUMENTS**

```!
ARGS=($ARGUMENTS)
# Status is optional trailing single word; everything before it is the title.
LAST="${ARGS[-1]}"
case "$LAST" in
  Proposed|Accepted|Deprecated|Superseded)
    STATUS="$LAST"
    TITLE="${ARGS[@]:0:${#ARGS[@]}-1}"
    ;;
  *)
    STATUS="Proposed"
    TITLE="$ARGUMENTS"
    ;;
esac

if [ -z "$TITLE" ]; then
  echo "usage: /adr <title> [Proposed|Accepted|Deprecated|Superseded]"
  exit 1
fi

DIR="docs/adr"
mkdir -p "$DIR"

# Find highest NNNN already used
LAST_NUM=$(ls "$DIR" 2>/dev/null | grep -oE '^[0-9]{4}' | sort -n | tail -1)
if [ -z "$LAST_NUM" ]; then
  NEXT_NUM="0001"
else
  NEXT_NUM=$(printf "%04d" $((10#$LAST_NUM + 1)))
fi

# slugify: lowercase, spaces → hyphens, strip non-alnum-hyphen
SLUG=$(echo "$TITLE" | tr '[:upper:]' '[:lower:]' \
       | sed -E 's/[^a-z0-9]+/-/g; s/^-+|-+$//g')

FILE="$DIR/${NEXT_NUM}-${SLUG}.md"
echo "writing $FILE (status: $STATUS)"
```

## WRITE THE ADR

```bash
# Recompute — bash blocks don't persist state across Claude Code sessions
DIR="docs/adr"
LAST_NUM=$(ls "$DIR" 2>/dev/null | grep -oE '^[0-9]{4}' | sort -n | tail -1)
NEXT_NUM=$([ -z "$LAST_NUM" ] && echo "0001" || printf "%04d" $((10#$LAST_NUM + 1)))

ARGS=($ARGUMENTS)
LAST="${ARGS[-1]}"
case "$LAST" in
  Proposed|Accepted|Deprecated|Superseded) STATUS="$LAST"; TITLE="${ARGS[@]:0:${#ARGS[@]}-1}";;
  *) STATUS="Proposed"; TITLE="$ARGUMENTS";;
esac

SLUG=$(echo "$TITLE" | tr '[:upper:]' '[:lower:]' | sed -E 's/[^a-z0-9]+/-/g; s/^-+|-+$//g')
FILE="$DIR/${NEXT_NUM}-${SLUG}.md"

cat > "$FILE" <<EOF
# ${NEXT_NUM}. $(echo "$TITLE" | sed -E 's/\b(.)/\U\1/g')

**Date:** $(date +%Y-%m-%d)
**Status:** $STATUS

## Context

<What problem forces a decision? What constraints, observations, prior
commitments are in play? State the facts — no recommendations yet.>

## Decision

<The decision, stated in the active voice: "We will …". One paragraph.
Be specific enough that someone reading this in a year can audit whether
the decision was followed.>

## Consequences

<What becomes easier, what becomes harder, what's now locked in. List
positive, negative, and neutral consequences. This is the section future
readers care about most — be honest about tradeoffs.>

## References

- <Links to prior ADRs this supersedes or relates to>
- <External docs, RFCs, benchmark data>
- Michael Nygard's original ADR template: https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions
- https://adr.github.io — index of ADR patterns and tooling
EOF

echo "created: $FILE"
echo ""
echo "next steps:"
echo "  - fill in Context / Decision / Consequences"
echo "  - \$EDITOR $FILE"
echo "  - git add $FILE && git commit -m \"adr: ${NEXT_NUM} $TITLE\""
```

## PHASE 2 — OPEN (optional)

```bash
if [ -n "$EDITOR" ] && [ -t 0 ]; then
  DIR="docs/adr"
  LATEST=$(ls -t "$DIR"/*.md 2>/dev/null | head -1)
  echo "open in \$EDITOR? file: $LATEST"
  echo "  (skip — running in non-interactive harness)"
fi
```

## NOTES

- Template: **Michael Nygard** (2011) — Context / Decision / Status / Consequences. Still the default in 2026 because it's short enough to actually get written.
- Status values: `Proposed` (under discussion), `Accepted` (decision is live), `Deprecated` (no longer applies but left for history), `Superseded` (replaced by ADR NNNN — link forward in the References section of the old one, backward in the new one).
- Numbering is monotonic — never renumber, never reuse, even when deprecating. ADR history is append-only.
- Keep ADRs short (<300 lines). If it's getting long, the decision isn't made yet — split it.
- Reference: **adr.github.io** — ADR patterns, GitHub's own ADR process, and tooling like `adr-tools` if you want CLI automation.
- Alternatives to Nygard: MADR (Markdown ADR, more structured), Y-statements (one-liner). Stick with Nygard unless the team already uses one of the others.
