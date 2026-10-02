---
description: "Rules for maintaining the '## Branches' table under '## Modernisation notes' in README.md — consistent table formatting, rows sourced only from real remote branches, never fabricated"
---

# README Branches Table — Agent Rules

## Overview

Repos in this collection document their modernisation history in `README.md`
via a `## Branches` table placed under the `## Modernisation notes` heading —
one row per development branch, describing the effort and linking to the
Copilot instruction file(s) that guided it.

The **formatting** of this section must stay consistent across projects.
The **contents** must always be regenerated from real, verifiable data for the
specific repo — never copied, guessed, or extrapolated from another project or
from a prior draft.

Token counts, model names, session names, session summaries, and interaction
mode guidance must **never** appear in a README. The README is for developers
reading the demo, not for the maintainer. See
`readme.structure.instructions.md` for the full README specification.

---

## Rule: every row must be sourced, not remembered

Before writing or updating the table:

1. Enumerate the actual branches on the remote (`git fetch && git branch -r`) —
   do not rely on memory of what branches "should" exist. A session's local
   working branch may have been renamed before push.
2. Confirm each instruction file referenced actually exists at
   `.github/instructions/<file>`.
3. If a branch cannot be confirmed on the remote, do not list it.

**Fabricated history is worse than no history.** It looks authoritative,
erodes trust in the README, and cannot be reproduced or audited.

---

## Table format

Keep the intro sentence and header exactly as below; only the body rows change
per project:

```markdown
## Modernisation notes

Each branch represents a distinct modernisation effort, guided by a corresponding Copilot instruction file.

| Branch | Description | Instructions |
|--------|-------------|--------------|
| [`<branch-name>`](../../tree/<branch-name>) | <one-line description of the effort> | [<file>.instructions.md](.github/instructions/<file>.instructions.md) |
```

Rules:

- One row per feature/modernisation branch, in **chronological order**
  (oldest first) by branch creation date.
- Do not include transient branches (README-only edits, sync/merge branches,
  housekeeping) unless the project intentionally documents all dev branches
  rather than curated feature branches — confirm scope with the user if
  ambiguous.
- Description is a single sentence stating what changed and why, not an
  implementation log.
- If a branch's work is guided by more than one instruction file, list all of
  them as comma-separated links in the same cell.
- Link format is fixed: ``[`branch-name`](../../tree/branch-name)`` for
  branches, `[filename](.github/instructions/filename)` for instructions.

---

## When the user supplies explicit content

If the user provides exact branch names, descriptions, or table rows to insert,
use them as given — do not re-verify against the remote. The verification
workflow below applies only when regenerating from scratch without
user-supplied data.

---

## Workflow for regenerating the table

1. `git fetch --all` and list the real remote branches.
2. For each branch, determine what it changed (diff against `main`, or the
   merged PR title/body) and write a one-sentence description.
3. Identify the instruction file(s) that governed that change.
4. Rebuild the table from that data, preserving the exact intro text, header
   row, and link formats above.
5. Do not touch any other section of the README.
6. Commit with a message describing what was corrected or updated.
