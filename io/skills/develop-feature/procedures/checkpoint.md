# Checkpoint

Sweep a `PROGRESS.md`'s `## Current focus` block — lift content that has a canonical home elsewhere, drop stale residue, condense the remainder to the template shape. Hygiene pass; no lifecycle transition.

Current focus is intentionally freeform so the user can park transient context that has no other home. Over time it accretes findings, decisions, and questions that belong in `code-review/`, `DECISIONS.md`, or `OPEN-QUESTIONS.md`. This procedure is the periodic compaction that keeps the freeform shape honest.

Two entry points:

- **User-invoked.** Asked for explicitly when the focus block has grown ("checkpoint the progress", "consolidate the current focus"). Runs end-to-end.
- **Auto-triggered.** Invoked by `procedures/section-archive.md` step 5c **before** the active-section block is rewritten — see "Auto-trigger from section-archive" below.

## Preconditions

- A target `PROGRESS.md` with a `## Current focus` block exists. Resolve target per state detection:
  - Build phase active → `build/PROGRESS.md`.
  - Topic active → `exploration/<topic>/PROGRESS.md` (including `exploration/custom/PROGRESS.md`).
  - References sidecar in flight → `references/PROGRESS.md`.
  - Top-level `docs/<feature>/PROGRESS.md` has no `## Current focus` — refuse.
- Caller (user or `section-archive`) names the target when more than one PROGRESS.md is active.

## Steps

### 1. Read the target

Read the full `PROGRESS.md`. Locate `## Current focus`. The template shape it must collapse to is fixed by the file:

- `build/PROGRESS.md` → `templates/progress-build.md`: `**<section-name>** — <qualifier>` + 2–3 line state summary + optional `Paused:` line + `Next up:` line.
- Topic / references `PROGRESS.md` → `templates/progress-topic.md`: `**<unit-name>** — <qualifier>` + 2–3 line state summary + `Next up:` line.

### 2. Classify the focus block

Walk every sentence / bullet beyond the prescribed shape. Bucket each:

- **Has a canonical home.** Route to its destination:
  - Concrete finding about section code → `build/code-review/<section>.md`. The Coding Agent **cannot** create findings (Review-Agent-owned per `reference/doc-ownership.md`). Surface to the user; options are (a) leave in place and let the next Review Agent auto-launch pick it up, (b) drop the line if the finding is obsolete. Do not write to the code-review file from this procedure.
  - Locked decision → `DECISIONS.md`. Apply the three lift criteria (`reference/lift-criteria.md`); confirm with user before appending (DECISIONS is confirm-before-edit per `reference/doc-ownership.md`). Use `templates/decisions.md` shape; `Source:` is `[<PROGRESS path> Current focus, <YYYY-MM-DD>]`.
  - Open question or blocker → `OPEN-QUESTIONS.md` via `procedures/open-questions.md` Add. OPEN-QUESTIONS adds don't need confirmation.
  - Pointer at a doc/file that already captures the substance → drop (the pointer remains in the residual summary if the user still wants it visible).
- **Pure scratch, still load-bearing.** Short context the user consciously parked here (e.g., "ping infra about retention policy before PR"). Keep — but trim to a single line.
- **Pure scratch, stale.** Superseded by later work. Drop.

Surface the classification as a numbered list before applying. The user breaks ties on borderline cases and confirms each lift.

### 3. Apply confirmed lifts

For each confirmed liftable line:

- `DECISIONS.md` → append per `templates/decisions.md`; number per `reference/sequence-rules.md`.
- `OPEN-QUESTIONS.md` → route through `procedures/open-questions.md` Add.
- Code-review-bound → no write (Review-Agent-owned); the line either stays in Current focus until the next auto-launch surfaces it as a real finding, or gets dropped.

Strip lifted lines from Current focus as each lift lands.

### 4. Rewrite Current focus to template shape

Collapse the residual to its template's prescribed shape (step 1). Keep load-bearing pure-scratch lines as one or two single-line bullets under the state summary, each prefixed `Scratch:` — explicit so the next checkpoint can re-classify them. Omit the `Scratch:` block entirely when empty.

### 5. Table-drift pass

Scan the `Active` / `Pending / Done` tables (build) or `## Progress` rows (topic) for row-discipline violations — full rules in `templates/progress-build.md` and `templates/progress-topic.md`:

- Text in the `Section` column beyond `<name>` + the inline `— BLOCKED by Q<N>` or `— deferred: <pointer>` marker.
- "Noticed that X" / rationale / finding text anywhere in a row.
- Pointer columns whose paths no longer exist at the live location.

Surface each violation with the row and proposed clean form. Do not auto-rewrite — table drift usually indicates a real state question that needs the user.

### 6. Summarise

Three to five lines: lift count by destination, scratch lines kept vs dropped, focus-block line count before → after, and any table drift surfaced (with row pointers).

## Auto-trigger from `procedures/section-archive.md`

Invoked at step 5c **before** the active-section block is rewritten. Target: `build/PROGRESS.md`.

Scope reduces to steps 2 + 3 + 5 only: section-archive's 5c immediately rewrites the active-section block per its own three-case branch, so step 4's rewrite is skipped. The lift-and-classify pass exists precisely to rescue liftable content from the rewrite — anything not lifted is lost when 5c overwrites.

The Coding-Agent-can't-create-findings constraint (step 2) is tighter at this trigger: by definition the section is about to leave Active, so the next Review Agent auto-launch won't fire on it. A code-review-bound line at archive-time is functionally a drop unless the user wants to spawn a final Review Agent before archive — surface the choice.

## Errors

- No `## Current focus` block in the target file: drift — surface and ask.
- Multiple active `PROGRESS.md` files and caller didn't specify: ask which.
- A line classified as code-review-bound but no Review Agent is active and the user declines a spawn: leave or drop per user; do not write.

## Postconditions

- `## Current focus` conforms to its template's shape (length, ordering, markers, optional `Scratch:` bullets).
- Each confirmed lift is appended to its destination file with a `Source:` backlink where the destination's template carries one.
- Table contents are unchanged unless the user explicitly accepted a row-discipline cleanup surfaced in step 5.
- For the auto-trigger entry: control returns to `procedures/section-archive.md` step 5c with the focus block stripped of all liftable lines — 5c then proceeds with its three-case rewrite.
