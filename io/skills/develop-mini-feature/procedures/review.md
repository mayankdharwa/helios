# Review (opt-in)

Run a code-review pass on a completed mini feature's build. Opt-in only; default is to skip review at mini scale.

This procedure is the entry point for both:
- The **Coding Agent** opting in to review at end of build (creates REVIEW.md and spawns the Review Agent subagent).
- The **Review Agent** running the actual review (delegates to `io:develop-feature` `procedures/code-review.md`).

## Preconditions

- A feature dir is active.
- All `## Build checklist` items are `[x]`.
- The user has not declined review for this feature.

## Two paths

### Path A: Coding Agent — opt user in and spawn Review Agent

Trigger: `advance.md` reached Build complete and offered the review opt-in; user chose **Review Agent subagent**.

1. Append a one-line marker to `PLAN.md` (anywhere above `## Review notes`, e.g., right after the H1):
   ```
   <!-- Review: enabled (Review Agent) -->
   ```
   `## Review notes` stays empty when the Review Agent is on — findings land in `REVIEW.md`.

2. Create `REVIEW.md` at `docs/<feature>/REVIEW.md`:
   ```markdown
   # Code review — <feature>
   ```

3. Spawn the Review Agent subagent via the Agent tool (`subagent_type: general-purpose`). The agent's first action is `Skill(skill="io:develop-mini-feature", args="review")`. Prompt body:
   ```
   Feature: <feature>
   Findings-doc: docs/<feature>/REVIEW.md
   Code anchor: PLAN.md ## Build checklist
   Tests: <optional — path or glob to test files, e.g., tests/api/test_reviews.py>
   ```

   The `Findings-doc:` and `Code anchor:` lines are the contract with `io:develop-feature` `procedures/code-review.md` — see step 1 of that procedure (which accepts these from the invocation prompt instead of inferring them from `build/PROGRESS.md`). The optional `Tests:` line points the Review Agent at where tests live so it can evaluate the **Test coverage** dimension in step 3 of `code-review.md`; if omitted, that dimension degrades to noting absent tests rather than gap-checking against a test spec.

4. Announce: `Spawning Review Agent for <feature> (mini review).` Do not gate on extra user approval — opt-in already happened.

5. When the subagent returns, surface its summary verbatim. If new untagged findings landed, respond inline (Coding Agent response blocks under each finding, shape in `io:develop-feature` `reference/doc-ownership.md`) and re-spawn the Review Agent for verification. Cap at two response rounds per finding before surfacing for tie-break.

6. Once every finding in `REVIEW.md` carries a `> **Resolution:**` tag, re-invoke `advance.md` so it runs the post-review lift sweep on `REVIEW.md` `decision:` tags.

### Path B: Review Agent — run the review

Trigger: skill invoked with `args="review"`.

1. Read the invocation prompt for `Feature:`, `Findings-doc:`, and `Code anchor:` lines. If absent, surface and stop — mini's Review Agent never infers these from feature dir shape.

2. Delegate to `io:develop-feature` `procedures/code-review.md` with the findings-doc anchor. The full skill's procedure carries the actual review machinery: finding shape, verify-then-tag loop, two-round cap, sentinel "no findings" for clean reviews. This procedure trusts that one.

3. The only mini-specific adjustment: the "Targets:" field on each finding (which in full skill maps to `build/PROGRESS.md` sub-rows) is replaced with a free-text reference to a `## Build checklist` item (quoted by its text — positional references break when the user reorders the list) and/or a file/line. Example:
   ```
   > **Targets:** Build checklist "implement /reviews endpoint", src/api/reviews.py:42
   ```

4. Hand back per the full procedure's "Hand back" step.

## Inline review (no Review Agent) — not a procedure path

If the user opts in to review but picks **inline self-review** instead of the Review Agent subagent, no spawn happens. The Coding Agent fills `## Review notes` in `PLAN.md` directly with prose findings, and any `> **Resolution:** decision: <text>` tags become lift candidates on the next `advance.md` sweep. This path is intentionally informal — there is no procedure for it.

## Errors

- Review opted in but no `## Build checklist` items: surface, ask whether to skip review.
- Review Agent invocation prompt missing `Findings-doc:` or `Code anchor:`: surface and stop.
- `REVIEW.md` already exists with findings from a previous review: route to the verify-then-tag branch of `io:develop-feature` `procedures/code-review.md` step 4, not a fresh review.

## Postconditions

- Path A: `REVIEW.md` exists with at least one finding (or the clean-review sentinel `Finding #1 — fixed: no findings`), and the `<!-- Review: enabled -->` marker is in `PLAN.md`.
- Path B: findings appended to the findings-doc per `io:develop-feature` `procedures/code-review.md` postconditions.
- Neither path edits `DECISIONS.md` — that's `advance.md`'s post-review sweep via `lift.md`.
