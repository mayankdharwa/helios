# Graduate to develop-feature

Convert a `docs/<feature>/` mini structure into the full `io:develop-feature` shape. One-way migration — there is no "demote to mini" path.

## Preconditions

- A feature dir is active under mini.
- At least one of the graduation triggers holds (see below). The user has confirmed they want to graduate.

## Graduation triggers

Any of these surfacing during normal work is a recommendation to graduate; surface to the user, do not auto-trigger.

- A 3rd distinct exploration area surfaces in `## Exploration notes` (e.g., schema + apis + jobs).
- `## Build checklist` exceeds ~10 items.
- A review pass surfaces more than ~5 untagged findings.
- A cross-feature decision is surfaced (would belong in repo-root `docs/DECISIONS.md`).
- The user explicitly asks.

## Steps

### 1. Confirm the conversion with the user

Show the proposed mapping (step 3 below) and confirm. Graduation is mechanical but irreversible.

### 2. Detect feature state

- Has the feature reached `## Approach`? (Yes if non-empty.)
- Has it reached `## Build checklist`? (Yes if any items exist.)
- Is review opted in? (Yes if the `<!-- Review: enabled -->` marker or `REVIEW.md` exists.)
- Does `DECISIONS.md` exist?

These determine which conversions run.

### 3. Apply conversions

| Mini source | Full skill destination | Notes |
|---|---|---|
| `OBJECTIVE.md` | `OBJECTIVE.md` | Unchanged. Append a `Systems in play` empty list if missing (full template includes it). |
| `OPEN-QUESTIONS.md` (if present) | `OPEN-QUESTIONS.md` | Unchanged. |
| `DECISIONS.md` (if present) | `DECISIONS.md` | Unchanged shape; `Source:` paths rewritten per the lift-backlink remap below. |
| `PLAN.md` `## Exploration notes` | `exploration/<topic>/<TOPIC>-EXPLORATION.md` | Ask user to split notes into the four standard topics (schema / apis / jobs / code-structure) — mini doesn't track that split, so the user must name it. Create topic `PROGRESS.md` per `io:develop-feature` `templates/progress-topic.md`. **Unit-row state must match the top-level exploration phase row** (set in the last row of this table): if exploration graduates as `✅` (because mini had already reached Approach), set every unit row to `✅` — the work is done. If exploration graduates as `🚧` (the user is graduating mid-exploration because a 3rd topic appeared), set units that correspond to already-captured callouts to `✅` and any new unit rows the user wants to add as `⏳`. Move callouts intact in either case. |
| `PLAN.md` `## Approach` | `spec/<file>.md` | Ask user how to split into spec files (often one for now). |
| `PLAN.md` `## Build checklist` | `build/PROGRESS.md` sections + per-section `testing/<section>.md` | Ask user how to group checklist items into sections. Each section gets a row in `build/PROGRESS.md` per `io:develop-feature` `templates/progress-build.md`. |
| `PLAN.md` `## Review notes` (if any) | discard or fold into `build/code-review/<section>.md` | Inline review notes that became `decision:` tags should already have been lifted at the review-close sweep. Anything remaining is informational. |
| `REVIEW.md` (if review was on) | `build/code-review/<section>.md` | Single section becomes one file. Findings keep their `Finding #N` numbers. The "Targets:" free-text becomes structured `§N.M` references per `io:develop-feature` `templates/code-review-finding.md`. |
| Top-level `PROGRESS.md` | created fresh from `io:develop-feature` `templates/progress-top-level.md` | Set phase rows based on detected state: exploration `✅` if `## Approach` was non-empty, `🚧` otherwise; spec `✅` if `## Build checklist` was non-empty, `🚧` otherwise; build `✅` if every checklist item was `[x]` and review (if opted in) is fully tagged, `🚧` otherwise. |

### 4. Rewrite DECISIONS.md backlinks

For each entry in `DECISIONS.md`, the `Source:` link was pointing into `PLAN.md` or `REVIEW.md`. Rewrite to the new home:

| Old `Source:` | New `Source:` |
|---|---|
| `PLAN.md §Exploration notes Review comment #M` | `exploration/<topic>/<TOPIC>-EXPLORATION.md §X Review comment #M` (topic determined by where the callout was split to in step 3) |
| `PLAN.md §Approach Review comment #M` | `spec/<file>.md §X Review comment #M` |
| `REVIEW.md item "..."` | `build/code-review/<section>.md item "..."` |

If review findings were split across more than one section in step 3 (the user grouped checklist items into multiple sections and the cited findings landed in different files), map each `DECISIONS.md` `Source:` entry per-finding: read its quoted finding text and locate it in the new `build/code-review/<section>.md` files to pick the right `<section>`. This is not a bulk rewrite.

Inline callouts themselves move with their hosting content — only the `Source:` line in `DECISIONS.md` changes.

### 5. Delete the mini-only files

After verifying the conversion:
- `git rm docs/<feature>/PLAN.md`
- `git rm docs/<feature>/REVIEW.md` (if it existed)

Do not delete `OBJECTIVE.md`, `OPEN-QUESTIONS.md`, or `DECISIONS.md` — those graduate in place.

### 6. Hand control to develop-feature

Surface: "Feature graduated. Next: invoke `io:develop-feature` to continue." Do not auto-route — the full skill's state detection will pick up the new layout on next invocation.

## Errors

- User can't decide which standard topic an exploration note belongs to: route the note to the optional `exploration/custom/` topic (see `io:develop-feature` `procedures/custom.md`).
- `DECISIONS.md` has entries whose inline `Review comment #N` can't be located: surface drift, ask user.
- Build checklist has items that don't map cleanly to sections: ask user; do not guess.

## Postconditions

- `docs/<feature>/PLAN.md` no longer exists.
- `docs/<feature>/REVIEW.md` no longer exists (if it had existed).
- `docs/<feature>/{exploration,spec,build}/` exist in the full skill's shape.
- `docs/<feature>/PROGRESS.md` exists with three phase rows.
- All `DECISIONS.md` `Source:` paths resolve.
- `io:develop-feature` would route this feature dir without surprise.
