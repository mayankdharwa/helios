---
name: develop-feature
description: Use when bootstrapping a new feature under `docs/<feature>/`, locking units, deferring questions, distilling spec, archiving build sections, surgically reopening locked phases, or checking project-done. Operates a three-phase exploration → spec → build process for medium-to-large backend features. Invoke with arg `review` to act as the Review Agent during build.
---

# develop-feature

Drive the multi-phase development of a feature. The user contributes substance; the skill handles ceremony — dirs, files, callout formats, archives, sequence numbers, lifecycle transitions.

This skill is suggest-only. Wait for explicit invocation; the agent may recommend it when the conversation enters lifecycle territory.

Do not engage for bugfixes, small refactors, single-file edits, or tasks expected to wrap in under a few hours.

For features that fit in about a day and touch 1–2 areas, use `io:develop-mini-feature` instead — same `docs/<feature>/` root, lighter scaffolding, graduates to this skill if scope grows (a 3rd exploration topic, an oversized build, a cross-feature decision). Wrong-skill guard: if `PLAN.md` exists at `docs/<feature>/` and no `build/` subdir, the feature is tracked by mini — route there.

## Roles

- **Coding Agent** — default role; drives work across phases and retains all ceremony ownership. In code-review, appends the response block under each finding (shape: `reference/doc-ownership.md`); never writes resolution tags. May delegate code-writing to an Assistant in build phase (see "Coding Agent delegates to Assistant" below).
- **Review Agent** — owns `build/code-review/<section>.md`: appends findings, verifies Coding Agent responses against the code, writes resolution tags (`fixed` / `wont-fix:` / `decision:` / `spec-changed:`). Entered by `review` arg. Procedure: `procedures/code-review.md`.
- **Coding Agent Assistant** — writes production and test code inside a Coding-Agent-declared file scope; never touches `docs/<feature>/`. Entered by `assist` arg. Procedures: `procedures/assist.md` (Assistant side), `procedures/delegate-code.md` (Coding Agent side).
- **User** — locks decisions, confirms protected-doc edits, answers open questions, breaks ties when Review Agent and Coding Agent disagree.

### Role declaration on invocation

The skill operates as **one** role per session — binding for the whole session. Role is selected by the skill argument:

- no arg → **Coding Agent** (default)
- `review` → **Review Agent** (prose declaration "You are the Code Review Agent" / "You are the Review Agent" is accepted as a synonym)
- `assist` → **Coding Agent Assistant**

Invocation: `Skill(skill="io:develop-feature", args="review")` or `args="assist"`. Start fresh sessions for the Review Agent (independent code read) and Assistant (context isolation, parallelism).

Run state detection (below). Then route by role: Review Agent with build phase active → `procedures/code-review.md`. Coding Agent Assistant → `procedures/assist.md`. Coding Agent → route by the action (see "Action routing"). No role edits another's lines in `build/code-review/<section>.md` (`reference/doc-ownership.md`); the Assistant has zero write access to anything under `docs/<feature>/`.

In the build phase, the cycle is TDD: write tests → write code → review → respond → independently verify and tag → repeat. Each section in `build/` carries triple test+code+review status (see `templates/progress-build.md`).

## Auto-launch protocol

Build phase only — exploration and spec have no Review Agent. The "suggest-only" rule (top of this file) governs the top-level Coding Agent invocation; once the Coding Agent is running, internal Review Agent spawns are mechanical and do not need a fresh user invocation.

The Coding Agent spawns the Review Agent itself — fresh subagent via the Agent tool (`subagent_type: general-purpose`) — at three moments, so review keeps pace with code:

- **Initial.** The first time every test in `build/testing/<section>.md` is green against new section code, with no further code changes planned for the section this turn. Fires *once per section lifecycle*: at first-ever pass, and again on the first all-green pass after a surgical reopen of the section (`procedures/surgical-reopen.md`). Any later green-after-red transition within the same section (rewrites, refactors that temporarily broke tests) is **Recheck**, not Initial.
- **Verify.** At end of turn, if the Coding Agent has appended one or more `> **Coding Agent response**` blocks during the turn. One launch per section per turn, regardless of how many responses. Refusal responses (no code change) still trigger verify — the Review Agent judges whether the refusal holds, see `procedures/code-review.md` step 4.
- **Recheck.** After section code changes (whether written in-house by the Coding Agent or by a delegated Assistant — `procedures/delegate-code.md`) where the change isn't addressing an open finding (research-driven update, refactor, tangent fix, green-after-red rewrite). The in-the-loop test is *intent, not file identity* — a refactor in a helper that resolves finding #N is in-the-loop even if the helper's file wasn't cited in the finding; an unrelated tangent in a cited file is Recheck. Applies only while the section row in `build/PROGRESS.md` is 🚧 — once the section is archived, code changes to cited files no longer trigger review. One launch per section per turn.

Batching is mandatory, not advisory. Each auto-launch is a fresh subagent that re-reads SKILL.md, `procedures/code-review.md`, `reference/doc-ownership.md`, the testing doc, the code-review doc, and the code itself. Coalescing five responses into one Verify launch saves four full re-reads.

Spawn the subagent via the Agent tool (`subagent_type: general-purpose`). Its first action is to call `Skill(skill="io:develop-feature", args="review")`; the Agent prompt body itself just identifies the section:

```
Section: <section-name>
[Feature: <feature-name>]
[Recheck: <one-line — what changed and why>]
```

`Feature:` is required when more than one feature dir under `docs/` has an active build phase; with a single active feature, the skill picks it. `Recheck:` is required only for the recheck trigger — `procedures/code-review.md` uses it to branch when there are pending responses *and/or* fresh recheck code. For initial and verify, the skill derives the branch from `build/code-review/<section>.md` state. (The label is `Recheck:` not `Drift:` — "drift" is reserved for state-inconsistency surfacing, see `procedures/code-review.md` Errors.)

Announce the launch in one line, shape: `Spawning Review Agent for <section> (trigger: initial | verify | recheck).` Do not gate on user approval — review is part of the TDD cycle, not a phase transition. Surface the subagent's summary verbatim; it replaces the Coding Agent's end-of-turn summary (Operating posture point 8) for the auto-launched cycle. If new untagged findings come back, respond inline and the verify trigger fires again at end of turn. The loop terminates when verify returns no new untagged findings — and `procedures/code-review.md` step 4 caps it at two response rounds per finding before surfacing to the user. If the subagent fails or returns without writing findings, surface and ask — do not retry silently.

**After the subagent returns, the Coding Agent updates the section's `Review` column in `build/PROGRESS.md`** — this is the only path the column flips on. Both the parent row and every sub-row §N.M are recomputed on the same return; sub-row state is derived from each finding's `Targets:` field per `templates/progress-build.md` Review column rules. The Review Agent never edits `build/PROGRESS.md` (see `procedures/code-review.md` postconditions). Three cases for the **parent row**, based on the state of `build/code-review/<section>.md` after the subagent's run:

- **First finding(s) just landed, row currently shows `Review: ⏳`** → flip parent to `Review: 🚧`. Normally fires on the Initial trigger's return; the rule is keyed on the current cell value, so any later path that resets the column to ⏳ and then gets findings is handled by the same edge.
- **Verify (or Recheck) returned and every item in `build/code-review/<section>.md` now carries a Resolution tag, with no untagged follow-ups outstanding** → flip parent `Review: 🚧 → ✅`. This is the Coding Agent's affirmation that the review → fix → verify loop is complete for this section. Do this **before** invoking `procedures/section-archive.md`; archive's preconditions expect `Review: ✅` already set. May fire on the same return as the case above for a clean review — the Initial trigger returns with a sentinel finding (`Targets: §N`) already tagged `fixed: no findings`, so the parent walks ⏳ → 🚧 → ✅ in one step.
- **Otherwise** (findings still untagged, response cycle still in flight, or no state change) → leave the parent column unchanged.

**Sub-row recompute (same return, after the parent flip).** Walk each sub-row §N.M and reset its `Review` cell per `templates/progress-build.md` Review (sub-row) rules — derived from each finding's `Targets:` field. The Initial-trigger return is the moment most sub-rows resolve: those with zero citing findings flip ⏳ → ✅ alongside the parent's ⏳ → 🚧 flip. Subsequent Verify/Recheck returns flip sub-rows 🚧 → ✅ as their citing findings get tagged, or ✅ → 🚧 if a new untagged finding lands on a previously-clean sub-row.

The `Review: 🚧 → ✅` flip and the subsequent `section-archive` invocation are two distinct actions: flip first, archive second. If the user redirects or pauses between them, the column accurately reflects "review done, archive pending."

## Coding Agent delegates to Assistant

Build phase only. The Coding Agent may spawn a **Coding Agent Assistant** in a fresh subagent to write code inside a declared file `Scope:`. **Discretionary** (Coding Agent's call) — unlike the Review Agent's auto-launch, the Assistant never spawns mechanically. The Review Agent's value is structural independence; the Assistant's is context conservation and parallelism. The Assistant never touches `docs/<feature>/`; the Coding Agent retains all column flips and routing.

- **Procedure (when to delegate, spawn, on-return verify + column flips + issue routing):** `procedures/delegate-code.md`.
- **Prompt shape** (filled-in body passed to the Agent tool; subagent's first action is `Skill(skill="io:develop-feature", args="assist")`): `templates/delegation-prompt.md`.
- **Hard write boundary:** Assistant writes only inside `Scope:`, and never under `docs/<feature>/` regardless of what `Scope:` declares (`reference/doc-ownership.md`).
- **Parallel dispatch — disjoint scopes only.** Multiple Assistants in one turn is allowed only when every Assistant's `Scope:` is disjoint from every other's. The Coding Agent enforces no-overlap *before* dispatching; the Assistant has no view of its siblings. Send parallel spawns as multiple Agent tool calls in a single message. Overlap is a Coding Agent error — tighten scopes or serialise.

Announce each spawn: `Spawning Assistant for <section> (scope: <Scope:>, tasks: <count>).` Surface the Assistant's return summary verbatim before the column-flip step.

Delegation does not interact with the Review Agent's auto-launch protocol — Initial / Verify / Recheck fire on their normal signals exactly as for in-house code.

## Operating posture

1. **Tiered autonomy.** Execute pure mechanics silently after the higher-level action is approved. Checkpoint once at phase/scope transitions (sweep, archive, distillation, surgical reopen entry). Prompt the user for substantive content (Review comment text, OBJECTIVE body, OPEN-QUESTIONS phrasing, cleanup-vs-meaning judgment).
2. **Read all state on every invocation.** Run `ls docs/`. Read the active feature's top-level `PROGRESS.md`, any active topic `PROGRESS.md`, `OPEN-QUESTIONS.md` index. State files are the cursor; never maintain a sidecar.
3. **Surface smells; do not block.** See `reference/smells.md`.
4. **Surface inconsistencies; never auto-sync.** When a state file drifts from reality, ask the user what to do.
5. **Bootstrap missing cross-feature scaffolding.** If `docs/DECISIONS.md` is missing when the user starts work, offer to plant a stub before creating the feature dir. Bootstrap touches only `docs/`; anything at repo root is out of scope.
6. **Enumerate multi-question turns.** When surfacing three or more decisions in a single turn, number them so the user can answer by index.
7. **Confirm before moving to the next step.** Once all open items in a phase or section resolve, confirm before proceeding.
8. **End with a tight summary.** Three to six lines: what changed, the next concrete step, any smells or deferred items.

## State detection on invocation

1. `ls docs/` — locate feature dirs (dirs containing `OBJECTIVE.md`). 0 → bootstrap mode. 1 → auto-pick. 2+ → ask which one; remember in conversation context.
2. Read `docs/<feature>/PROGRESS.md`. Identify which of the three phase rows (`exploration`, `spec`, `build`) is `🚧`. `references/` is a sidecar — no phase row.
3. If a phase row is `🚧`, read the pointer it carries (e.g., `→ exploration/schema/PROGRESS.md`) for the active topic / section.
4. Read `OPEN-QUESTIONS.md` index to know what's blocked.
5. Route by role and action (see below). Under a Review Agent selection (`review` arg) with build phase active, route to `procedures/code-review.md`.

If `PROGRESS.md` is malformed or unreadable, surface and ask. Do not guess.

## Action routing

- Start a new feature / no feature dir exists → `procedures/bootstrap.md`
- Lock a unit / approve a unit → `procedures/unit-lock.md`
- Defer a unit / can't lock yet → `procedures/defer.md`
- Add a custom sub-exploration / lock one → `procedures/custom.md`
- Topic done / all rows `✅` in topic → `procedures/topic-lock.md`
- All exploration topics locked / ready to distill spec → `procedures/distill.md`
- Invoked as Review Agent (to review the active build section) → `procedures/code-review.md`
- Invoked as Coding Agent Assistant (delegated code-writing in a fresh subagent) → `procedures/assist.md`
- Coding Agent decides to delegate a mechanical code-writing task → `procedures/delegate-code.md`
- Findings awaiting a Coding Agent response → respond inline per `reference/doc-ownership.md` (Coding Agent appends the `> **Coding Agent response**` block; does not tag). End-of-turn: see auto-launch line below.
- Section tests first all-green / responses posted this turn / code changed in a file cited by `build/testing/<section>.md` → auto-launch Review Agent (see "Coding Agent auto-launches Review Agent")
- Verify loop just returned all-tagged → flip `Review: 🚧 → ✅` (SKILL.md "Auto-launch protocol"), then `procedures/section-archive.md`
- Section blocked on a question / can't proceed mid-work → `procedures/open-questions.md` Add (also pauses the 🚧 row in `build/PROGRESS.md` and adds the `— BLOCKED by Q<N>` marker — see step 5)
- Surgically reopen / proposed edit to `spec/*` → `procedures/surgical-reopen.md`
- Add open question / resolve `Q<N>` → `procedures/open-questions.md`
- Add a reference / new doc in `references/` → `procedures/references.md`
- Checkpoint the progress / consolidate `## Current focus` (also auto-invoked by `procedures/section-archive.md` step 5c) → `procedures/checkpoint.md`
- Add migration item / close `<id>` → `procedures/migration-checklist.md`
- Add tech-debt item / close `TD<N>` → `procedures/tech-debt.md`
- Decision affects multiple features → `procedures/cross-feature-lift.md`
- Newer decision replaces an older one / mark `#M` superseded by `#N` → `procedures/supersede.md`
- Is the project done / final gate check → `procedures/project-done-check.md`
- An issue surfaced mid-work / unclear where to log it → `reference/issue-routing.md`

Each procedure file is self-contained: preconditions, steps, errors, postconditions. Read the procedure when routing to it.

## Reference files

Read on-demand when the topic comes up:

- `reference/sequence-rules.md` — archive sequences (`<seq>` resolution), numbered artifacts (`#N`, `Q<N>`, `Review comment #N`, `Finding #N`, migration IDs).
- `reference/smells.md` — state-shape problems to surface.
- `reference/section-split.md` — when a `build/` or exploration file needs splitting; also the positive definition of a section.
- `reference/doc-ownership.md` — edit rules per path.
- `reference/lift-criteria.md` — the three lift criteria, sweep cadences, immediate-lift mechanics.
- `reference/issue-routing.md` — the "Where does this go?" decision rule for mid-work issues.
- `reference/lock-semantics.md` — the three lock scopes (unit / topic / phase) and the phase-termination gates.
- `reference/migration-checklist-shape.md` — file-level structure, item ID, tags, and item shape for `MIGRATION-CHECKLIST.md`.
- `reference/tech-debt-shape.md` — file-level structure, item ID, and item shape for `TECH-DEBT.md` (non-blocking own-codebase follow-ups).
- `reference/custom-shape.md` — file-level structure and distillation fold rules for the optional `exploration/custom/` topic (sub-explorations that don't fit the standard four).

## File templates

Read the template before writing a new artifact:

- `templates/objective.md` — feature `OBJECTIVE.md`.
- `templates/progress-top-level.md` — feature top-level `PROGRESS.md`.
- `templates/progress-topic.md` — exploration topic `PROGRESS.md` and `references/PROGRESS.md` (with `## Current focus`).
- `templates/progress-build.md` — `build/PROGRESS.md`. Two tables: **Active** (sections that are 🚧 or ⏸, with §N.M sub-rows under each parent) and **Pending / Done** (sections that are ⏳ or ✅, flat — ordered by §). Each row carries `Tests` / `Code` / `Review` columns; sub-rows track all three (sub-row `Review` is derived from each finding's `Targets:`) and leave `Pointer` as `—`.
- `templates/open-questions.md` — feature `OPEN-QUESTIONS.md`.
- `templates/decisions.md` — feature `DECISIONS.md`.
- `templates/review-comment.md` — inline `Review comment` callout.
- `templates/code-review-finding.md` — `build/code-review/<section>.md` finding item (Review Agent), with the response-block and resolution-tag shapes.
- `templates/delegation-prompt.md` — Coding Agent → Assistant invocation prompt body (`Section:` / `Scope:` / `Tasks:` / `Spec refs:`).
