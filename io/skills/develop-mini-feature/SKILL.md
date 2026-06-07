---
name: develop-mini-feature
description: Use for small features (up to a day, 1–2 areas) under `docs/<feature>/` — bootstrapping, advancing phases, lifting decisions, optional review. Mirrors `io:develop-feature` shape but drops phase/topic/section ceremony. Graduates to `io:develop-feature` when scope grows. Invoke with arg `review` to act as the Review Agent.
---

# develop-mini-feature

Drive a lightweight explore → approach → build cycle for a small feature. The user contributes substance; the skill handles ceremony — directory creation, lift bookkeeping, optional Review Agent spawning, and graduation to `io:develop-feature` when the scope outgrows mini.

This skill is suggest-only. Wait for explicit invocation; the agent may recommend it when the conversation enters lifecycle territory.

Do not engage for: typo fixes, single-line changes, bugfixes with a clear root cause (use `practo:fix-sentry` or `practo:debug` instead), pure refactors (use `practo:refactor`), or tasks that finish in under an hour.

## When to use this skill vs `io:develop-feature`

Use **`io:develop-mini-feature`** (this skill) when:
- The work fits in roughly a day.
- It touches 1–2 areas (one schema tweak, one API change — not all three).
- You don't yet know whether spec needs a separate doc.

Use **`io:develop-feature`** when:
- Multi-day work.
- 3+ distinct exploration topics (schema + apis + jobs, etc.).
- You know up front you'll need locked spec docs, multiple build sections, and a Review Agent loop.

If a mini feature grows past the ceiling, route to `procedures/graduate.md` — one-way migration into the full skill's shape.

## Roles

- **Coding Agent** — drives work. Skill default. Writes `OBJECTIVE.md`, `PLAN.md`, `DECISIONS.md` entries (via `lift.md`), and code. In review, appends `> **Coding Agent response**` blocks under each finding in `REVIEW.md`.
- **Review Agent** — owns `REVIEW.md`. Entered by `args="review"`. Delegates the review machinery to `io:develop-feature/procedures/code-review.md`.
- **User** — confirms lift candidates, opts in or out of review, breaks ties when responses and findings disagree.

### Role declaration on invocation

One role per session, selected by the skill argument:

- no arg → **Coding Agent** (default).
- `review` → **Review Agent**.

Invocation: `Skill(skill="io:develop-mini-feature", args="review")`. The Review Agent is auto-spawned by `procedures/review.md` Path A; users normally never invoke `review` directly.

## State detection on invocation

Run on every invocation:

1. `ls docs/` — locate feature dirs (dirs containing `OBJECTIVE.md`). 0 → bootstrap mode. 1 → auto-pick. 2+ → ask which one.
2. **Wrong-skill guard.** Read the feature dir contents. If `exploration/`, `spec/`, or `build/` subdirs exist, this feature is tracked by `io:develop-feature`. Surface: "This feature is on the full skill. Invoke `io:develop-feature` instead." Stop.
3. Read `PLAN.md`:
   - All sections empty → between `start` and first exploration content.
   - `## Exploration notes` filled, `## Approach` empty → exploration phase.
   - `## Approach` filled, `## Build checklist` empty → approach phase.
   - Build checklist exists with `[ ]` items → build phase.
   - All checklist items `[x]` → build complete.
4. Note opt-in markers:
   - `<!-- Review: enabled (Review Agent) -->` → `REVIEW.md` exists and is the review surface.
   - Otherwise review is either off or inline (look at `## Review notes`).
5. Route by role + action.

If `PLAN.md` is malformed or unreadable, surface and ask. Do not guess.

## Action routing

- Start a new mini feature / no feature dir exists → `procedures/start.md`
- User says "approach is settled" / "start building" / "all done" → `procedures/advance.md`
- Inline decision feels course-altering and the user agrees → `procedures/lift.md` (immediate-lift path)
- Newer decision replaces an older one in `DECISIONS.md` / mark `#M` superseded by `#N` → `procedures/supersede.md`
- Build checklist fully ticked, user opts into review → `procedures/review.md` (Path A: Coding Agent)
- Invoked with `args="review"` → `procedures/review.md` (Path B: Review Agent)
- Graduation trigger fires (3rd exploration topic, oversized checklist, cross-feature decision, oversized review) → `procedures/graduate.md`
- Open question surfaces → append to `docs/<feature>/OPEN-QUESTIONS.md` (create on first one); no procedure needed — the conversational flow handles it.
- Anything mid-work that isn't a plain code fix → see "Where does this go?" below.

## Decision lift (short form)

A decision in `## Exploration notes`, `## Approach`, `## Review notes`, or `REVIEW.md` qualifies for `DECISIONS.md` if **all three** hold:

1. **Hard to reverse** — reversing it would touch multiple unrelated files or phases.
2. **Surprising without context** — a future reader will ask "why?" without the rationale.
3. **Real trade-off** — there were genuine alternatives.

Sweep happens at every `advance.md` phase transition; immediate lift can fire any turn. `DECISIONS.md` is lazy — created on first lift. Per-feature monotonic `#N`, numbers never reuse. The inline callout/tag stays in place; the `DECISIONS.md` entry carries the distilled "decision + why" plus a backlink (`Source:` line). One-way pointer: central → inline.

Canonical reference: `io:develop-feature/reference/lift-criteria.md`. Detailed procedure: `procedures/lift.md`.

## Where does this go? (issue routing, short form)

Mid-work issue, unclear destination:

1. **Plain code fix, no user input needed** → fix it.
2. **Needs user input now and user is available** → discuss → record as a `Review comment` callout in the active `PLAN.md` section (or `decision:` tag in review).
3. **Needs user input later** → append to `OPEN-QUESTIONS.md` (create on first one).
4. **Requires modifying locked content** → mini has no locks per se; if the user wants this kind of friction, it's a graduation signal.

If the issue is owned by another team / another codebase, or is a "do later" cleanup, mini has no equivalent of `MIGRATION-CHECKLIST.md` or `TECH-DEBT.md`. Either fold into `OPEN-QUESTIONS.md` or, if persistent tracking matters, graduate.

Canonical reference: `io:develop-feature/reference/issue-routing.md`.

## Operating posture

1. **Tiered autonomy.** Execute mechanics silently once the higher-level action is approved. Checkpoint at phase transitions (advance), DECISIONS lifts, and review opt-in. Prompt for substantive content (OBJECTIVE body, callout text).
2. **Read all state on every invocation.** `ls docs/`, read `OBJECTIVE.md` and `PLAN.md`. State files are the cursor; never maintain a sidecar.
3. **Surface graduation signals; do not auto-graduate.** A 3rd topic, an oversized checklist, or a cross-feature decision is a recommendation, not a trigger. Always ask before invoking `procedures/graduate.md`.
4. **Confirm before moving to the next step.** Especially before lifts and phase transitions.
5. **End with a tight summary.** Three to six lines: what changed, the next concrete step, any graduation signals.

## File ownership (short form)

- `OBJECTIVE.md` — confirm with user before any edit after `start.md`.
- `PLAN.md` — Coding Agent edits freely.
- `OPEN-QUESTIONS.md` — Coding Agent adds entries freely; resolving requires user confirmation. Shape on creation: `io:develop-feature/templates/open-questions.md`.
- `DECISIONS.md` — Coding Agent appends new entries via `lift.md` only; never rewrites existing entries. The single permitted edits to existing entries are the supersession markers added by `procedures/supersede.md` (the `*Superseded by #N — YYYY-MM-DD*` line on the older entry and the `**Supersedes:** #M[, …]` line on the newer one); the original `Decision:` / `Why:` / `Implications:` / `Source:` of every touched entry stay intact.
- `REVIEW.md` — Review Agent owns the file. Coding Agent appends only `> **Coding Agent response**` blocks under findings (shape: `io:develop-feature/reference/doc-ownership.md`).

Lifecycle moves (graduate's `git rm` of `PLAN.md` and `REVIEW.md`) are exempt from ownership rules — `procedures/graduate.md` performs them.

## Files in this skill

```
procedures/
  start.md     — bootstrap a feature dir
  advance.md   — phase transition + lift sweep
  lift.md      — append one decision to DECISIONS.md
  supersede.md — mark an older DECISIONS entry superseded by a newer one
  review.md    — opt-in review (Path A: Coding Agent; Path B: Review Agent)
  graduate.md  — migrate to io:develop-feature shape
templates/
  objective.md — same shape as io:develop-feature's, minus "Systems in play"
  plan.md      — Exploration notes / Approach / Build checklist / Review notes
```

Each procedure is self-contained: preconditions, steps, errors, postconditions. Read the procedure when routing to it. For finding/callout shapes, read directly from `io:develop-feature/templates/` — mini does not duplicate them.
