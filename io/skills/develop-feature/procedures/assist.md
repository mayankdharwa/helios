# Assist (build phase — Coding Agent Assistant entry)

The entry point when the skill is invoked in the **Coding Agent Assistant** role — selected
by the `assist` argument; see SKILL.md "Role declaration on invocation". The skill defaults
to the Coding Agent; this procedure runs only under an explicit Assistant selection.

The Assistant is spawned by the Coding Agent (`procedures/delegate-code.md`) in a fresh
subagent session. Its narrow surface is: write production and test code inside a declared
file scope; surface anything else back to the Coding Agent. It never touches ceremony.

Roles: SKILL.md "Roles". Ownership: `reference/doc-ownership.md`. Caller: `procedures/delegate-code.md`.

## Preconditions

- Invoked with skill arg `assist`.
- The invocation prompt carries the delegation block (shape in `templates/delegation-prompt.md`):
  - `Section: <section-name>` — required.
  - `Scope: <paths or globs>` — required.
  - `Tasks:` — at least one bullet — required.
  - `Spec refs:` — strongly recommended. Without at least one ref (or with every ref
    resolving to empty/irrelevant content), the Assistant has no contract to implement
    against and stops the whole delegation in step 3, returning an error summary. No task
    is attempted.
  - `Feature:` — required only when more than one feature dir under `docs/` has an active
    build phase.
- The named section must be `🚧` in `build/PROGRESS.md`'s Active table.

If any required field is missing, or `Section:` doesn't match a `🚧` row, stop and return an
error summary — do not improvise or guess.

## Hard write boundary

The Assistant **may write only** to paths inside the declared `Scope:`. It **may never write**:

- Anything under `docs/<feature>/` — the ceremony tree. Explicitly: `PROGRESS.md`,
  `DECISIONS.md`, `OPEN-QUESTIONS.md`, `TECH-DEBT.md`, `MIGRATION-CHECKLIST.md`,
  `OBJECTIVE.md`, `spec/*`, `build/testing/<section>.md`, `build/code-review/<section>.md`,
  `references/*`, `_archive/*`. These are Coding-Agent-only regardless of what `Scope:`
  appears to allow.
- Any path outside `Scope:`, even within the source tree.

If `Scope:` and the ceremony exclusion ever conflict (e.g., someone wrote `Scope: docs/**`),
the ceremony exclusion wins — refuse and return an error. The Assistant does not negotiate
the boundary; the Coding Agent fixes the delegation prompt.

Reads are unconstrained — read whatever's needed to understand the contract.

## Steps

### 1. Parse the delegation prompt

Extract `Section:`, `Scope:`, `Tasks:`, `Spec refs:`, optional `Feature:`. Validate per
preconditions. Stop on any failure.

### 2. Locate the section

Run `ls docs/` (or use `Feature:` if provided) to find the active feature dir. Read
`docs/<feature>/build/PROGRESS.md` and confirm `Section:` matches a `🚧` parent row in the
Active table.

If multiple feature dirs have an active build phase and `Feature:` is missing from the
invocation prompt, stop and return an error summary — the Assistant does not guess which
feature it's writing code for.

If the row is `⏸`, `⏳`, `✅`, or absent — drift; stop and return an error summary.

### 3. Read the contract

Read each `Spec refs:` target — only the headings cited, not whole files unless needed.
Read `build/testing/<section>.md` for the test contract. Read existing code in `Scope:`
paths to understand the surrounding style and patterns.

If `Spec refs:` is missing or every ref turns out empty/irrelevant, stop and return: there's
no contract to implement against.

### 4. Implement each task

For each bullet in `Tasks:`:

1. Implement against the spec/testing contract. Use existing utilities and patterns from
   the surrounding code; don't introduce new abstractions unless the task demands it.
2. Every write must land inside `Scope:`. Before any write, confirm the target path matches
   one of the `Scope:` entries (or expands to one via glob). If not, refuse the write and
   record the task as blocked in the return summary.
3. Follow project conventions visible in the code (naming, error handling, logging, file
   organisation). Don't import patterns that aren't already in use.
4. **On ambiguity, missing dependency, or unexpected state:** STOP that task. Do not
   improvise. Do not log an entry to `OPEN-QUESTIONS.md`, `TECH-DEBT.md`, `DECISIONS.md`,
   or any other ceremony file — that's the Coding Agent's call. Record it as a
   `Needs Coding Agent attention:` item in the return summary and move to the next task.

### 5. Run tests

If the project exposes a test command (visible from `package.json`, `Makefile`, CI config,
or convention) and the section has tests against the code being written, run the relevant
subset. Capture pass/fail.

Tests are run for the return summary's `Tests run:` line — the Coding Agent re-runs them on
its end before flipping `Code: ✅` in `build/PROGRESS.md`. The Assistant's run is a signal,
not a gate.

### 6. Verify and return

Before returning:

- List every file changed during this session.
- Confirm each one is inside `Scope:` and not under `docs/<feature>/`. If any write
  escaped — this is an Assistant bug; do **not** silently revert. Flag it explicitly in
  the return summary so the Coding Agent can revert and audit.

Return the summary in this exact shape:

```
Section: <section-name>
Files changed:
  - <path> — <created | edited> — <one-line why>
Tests run: <command(s)> — <pass / fail / not run>
Needs Coding Agent attention:
  - <ambiguity or surprise the Assistant declined to resolve>
```

If no `Needs Coding Agent attention:` items exist, omit the section. If no tests were run
(no test command, or no tests apply), write `Tests run: not run — <one-line why>`.

The Coding Agent reads this summary and runs the verify-then-flip-PROGRESS path in
`procedures/delegate-code.md` step 4.

## Errors

- Required field missing from invocation prompt → stop, return error summary.
- `Section:` doesn't match a `🚧` row → drift; stop, return error summary.
- `Scope:` contains a `docs/<feature>/` path → refuse the whole delegation; stop, return
  error. The Coding Agent fixes the prompt and retries.
- An attempted write resolves outside `Scope:` → refuse the write; record the task as
  blocked in the return summary; continue with remaining tasks.
- An attempted write resolves under `docs/<feature>/` → refuse the write; flag prominently
  in the return summary as a boundary violation attempt. Continue with remaining tasks.

## Postconditions

- Every file written sits inside the declared `Scope:` and is not under `docs/<feature>/`.
- No file under `docs/<feature>/` was modified.
- The return summary follows the shape in step 6 and accurately lists changed files,
  tests run, and any `Needs Coding Agent attention:` items.
- The Coding Agent has everything it needs to verify scope-containment, run tests, flip
  `build/PROGRESS.md` columns, and route any deferred items.
