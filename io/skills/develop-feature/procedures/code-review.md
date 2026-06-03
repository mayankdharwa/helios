# Code review (build phase — Review Agent entry)

The entry point when the skill is invoked in the **Review Agent** role — selected by the
`review` argument (or the prose synonym "You are the Code Review Agent"); see SKILL.md
"Role declaration on invocation". The skill defaults to the Coding Agent; this procedure
runs only under an explicit Review Agent selection.

Governs the *ceremony* of reviewing: where to look, where findings land, the finding and
resolution shapes, and the verify-then-tag loop. The *substance* of the review — what to
flag — is the agent's judgement. The checklist in step 3 is a prompt, not a gate.

## Preconditions

Two entry shapes:

**Shape A — `io:develop-feature` (default).** Build phase is active: `build/` exists and `build/PROGRESS.md` has at least one section row across its Active and Pending / Done tables. At least one section in the **Active** table has code to review — a row whose parent shows `Code: 🚧` or `Code: ✅` with `Review: ⏳` or `🚧`. A section whose parent is ⏸ has nothing new to review until it resumes; a section in Pending / Done is either not started (⏳, no code) or already archived (✅, review complete).

**Shape B — `io:develop-mini-feature` delegation.** The invocation prompt carries `Findings-doc: <path>` and `Code anchor: <text>` lines (and a `Feature: <name>` line). The findings-doc is a single review file (typically `docs/<feature>/REVIEW.md`); the code anchor names what to review (a checklist section, a file list). The downstream review loop is identical to Shape A — finding shape, verify-then-tag, two-round cap, clean-review sentinel — but anchored on a single findings-doc instead of one per build section. Mini delegates here from `io:develop-mini-feature/procedures/review.md` Path B.

If neither shape resolves — build phase not active *and* no `Findings-doc:` in the prompt — surface and stop. If invoked in the Review Agent role but no code exists to review, say so and stop.

Roles, ownership, and the tag set: `reference/doc-ownership.md`. Lock/archive flow:
`reference/lock-semantics.md`, `procedures/section-archive.md`. Lock/archive applies only to Shape A — Shape B has no `build/PROGRESS.md` to flip and no `_archive/`; mini handles its own post-review sweep via `io:develop-mini-feature/procedures/advance.md`.

## Steps

### 1. Detect the review surface

**Shape B (mini delegation).** If the invocation prompt carries `Findings-doc:` and `Code anchor:`, use them directly. The findings-doc replaces `build/code-review/<section>.md` for the rest of this procedure; the code anchor replaces the testing-doc + PROGRESS.md pointer chain. There is no `build/PROGRESS.md` to consult and no testing-doc to read — the Coding Agent's `## Build checklist` items in the named `PLAN.md` substitute for `testing/<section>.md` as the statement of what was built. Skip the rest of this step.

**Shape A (full skill).** Standard state detection has already run (SKILL.md). From `build/PROGRESS.md`:

- One section with reviewable code → that section.
- Several → list them with their `Tests` / `Code` / `Review` status from the Active table
  and ask which to review. Remember the choice in conversation context.

If the invocation prompt carries `Section: <name>` (auto-launch from the Coding Agent —
SKILL.md "Coding Agent auto-launches Review Agent"), use it directly — do not ask. A
`Recheck: <note>` line on the same prompt signals the recheck branch in step 3.

Read the section's `build/testing/<section>.md` (what the tests assert, what they don't)
and `build/code-review/<section>.md` if it already exists (prior findings, which carry
Coding Agent responses awaiting verification — step 4).

**Wherever this procedure says `build/code-review/<section>.md` below, read it as "the findings-doc"** — that's the file shared between Shapes A and B. Shape B has one findings-doc; Shape A has one per section. The verify-then-tag loop, finding numbering, and clean-review sentinel are identical across both.

### 2. Read the code under review

Identify the code for this section from the testing doc and `build/PROGRESS.md` pointers.
Read it. This is a fresh, independent read — do not assume the Coding Agent's framing.

### 3. Conduct the review

**Recheck branch.** If the invocation prompt carried `Recheck: <note>`, run step 4 as usual
on any pending Coding Agent responses, and *additionally* post findings on the changed
code. The two are independent — pending verify work doesn't preempt the recheck pass, and
fresh recheck findings don't preempt verification of the prior round. If no responses are
pending and all existing findings are tagged, the recheck note is still not "nothing to do":
treat it as a seed, re-read the changed code and the files cited in `testing/<section>.md`,
and post findings on the new code. Previously-tagged findings stand unless the new code
regresses them.

What to flag is your judgement. The following dimensions are a non-binding prompt — skip
any that don't apply, add any the code warrants:

- **Correctness** — does it do what the testing doc and spec say? Edge cases, off-by-one,
  null/empty, concurrency.
- **Error handling** — structured errors with error codes; failures surfaced, not swallowed.
- **Logging / observability** — structured JSON logging, correlation IDs on the request path.
- **Project conventions** — feature-based organisation, naming, and the patterns recorded in any
  `.claude/memory/patterns-learned.md` or equivalent project notes.
- **Test coverage** — do the tests in `testing/<section>.md` (Shape A) or the tests pointed to by the invocation prompt's `Tests:` line (Shape B, if provided) actually cover the code's branches? Gaps are findings too. Under Shape B, mini does not enforce TDD: if no `Tests:` pointer is provided, treat test coverage as a soft prompt — note absent tests as a finding but do not block on intent-vs-assertion mismatches that a testing doc would catch.
- **Security** — input validation, authz checks, no secrets in code/logs, injection surfaces.
- **Clarity / maintainability** — naming, dead code, duplication, comment density matching
  the surrounding file.

Pure-mechanical nits the Coding Agent would obviously fix need not become findings unless
they recur or matter; prefer signal over volume.

### 4. Append findings and run the verify-then-tag loop

The Review Agent owns `build/code-review/<section>.md`. If it doesn't exist, create it
(`# Code review — <section>` heading, then findings). Append each finding using
`templates/code-review-finding.md`. Number per-file monotonic (`Finding #N`) —
`reference/sequence-rules.md`.

Every finding must carry a `Targets:` line.

**Shape A.** Cite the §N.M sub-rows the finding touches (one or more), or `§N` for findings that span the whole section. Read the §N.M numbers from `build/PROGRESS.md`'s Active table — the section's sub-rows are the authoritative numbering; do not invent numbers from the testing doc's headings directly. The Coding Agent uses `Targets:` to recompute per-sub-row `Review` cells in `build/PROGRESS.md` on each subagent return — see SKILL.md "Sub-row recompute". Pick the narrowest accurate set; reach for the section-wide `§N` only when the finding is genuinely cross-cutting (architectural, naming-wide, an invariant the whole section breaks).

**Shape B.** `Targets:` is free-text — a `## Build checklist` item from `PLAN.md` (quote the item text) and/or a `file:line` reference. There is no `build/PROGRESS.md` to anchor against and no sub-row recompute downstream. Example: `Targets: Build checklist "implement /reviews endpoint", src/api/reviews.py:42`.

Clean review (zero findings): still create `build/code-review/<section>.md` and append a
single sentinel item — `Finding #1` with body "No findings — clean review of <section>",
`Targets: §N` (section-wide), and an already-written `> **Resolution:** fixed: no findings`
tag. This keeps the file-exists + every-item-tagged invariants uniform across clean and
non-clean sections, so `section-archive` and the `review:` status walk the same path either
way; with the sentinel already tagged, every sub-row flips ⏳ → ✅ in the same step the
parent flips ⏳ → 🚧 → ✅.

The verify-then-tag loop crosses contexts — the Coding Agent and Review Agent live in
separate subagent sessions, and each handoff is the file on disk, not a continued
conversation. `build/code-review/<section>.md` is the shared medium; do not assume any
prior turn's reasoning carries over implicitly.

For each finding, the cycle is:

1. **You append the finding.** Stop there for new findings — the Coding Agent responds next.
2. **Coding Agent appends a response block** under the item — `> **Coding Agent response**`,
   shape in `reference/doc-ownership.md`. You do not write that block.
3. **You independently verify against the code** — re-read the relevant code; do not trust the
   response's claim. For action-taken responses, check the change resolves the finding. For
   refusal responses (`wont-fix` / `decision` / `spec-changed` — no code diff expected),
   re-read the code as-is and judge whether the refusal reason holds against current code.
   Then append the `> **Resolution:**` tag line (shape in `templates/code-review-finding.md`)
   as the item's final line:
   - `fixed` — verified the change resolves it.
   - `wont-fix: <reason>` — agreed it should not change.
   - `decision: <text>` — resolved by a recorded user/judgement call (record the call).
   - `spec-changed: <link>` — needs a locked-spec change; link the surgical-reopen/OPEN-QUESTIONS entry.
4. **Agree-but-out-of-section-scope.** When the Coding Agent's response is "valid
   observation, out of section scope," do not auto-tag. Surface to the user — they decide
   whether to log it as a TD entry (`procedures/tech-debt.md` Add mode → then tag the
   finding `wont-fix: see TD<N>`) or close without one (`wont-fix: <reason>`). The user's
   call, not the Review Agent's. The Review Agent never invokes `procedures/tech-debt.md`
   itself.
5. **Disagree with a refusal?** Leave the item **untagged** and append a follow-up finding
   under it. An untagged item blocks `section-archive` (`procedures/section-archive.md`),
   which forces another response cycle or a user tie-break. Do not tag to unblock.
6. **Cycle limit — two response rounds per finding.** Count Coding Agent response blocks on
   a single finding: if a third response is being verified (response → follow-up → response →
   follow-up → response), do not append yet another follow-up. Surface the disagreement to
   the user for tie-break and leave the item untagged. Two response rounds is the cap;
   beyond that, the cycle is not converging on its own.

Never edit the Coding Agent's response lines. Never write a `> **Coding Agent response**` block.

### 5. Route issues that aren't plain code findings

If something you find isn't a straightforward code fix — needs user input, needs a locked
spec changed, or belongs elsewhere — route via `reference/issue-routing.md` instead of (or
in addition to) a finding. One issue lands in exactly one place.

### 6. Hand back

Once you've posted findings (or verified responses and tagged), summarise per SKILL.md:
what you reviewed, how many findings at each severity, how many tagged vs awaiting Coding
Agent response, and whether the section is now fully tagged (ready for `section-archive`).

Do not archive from this procedure unless explicitly asked — archive is a separate lifecycle
action (`procedures/section-archive.md`) and the Coding Agent or user typically triggers it.

## Errors

- Build phase not active → surface, stop.
- No section has reviewable code → surface, stop.
- Section name doesn't match any `build/PROGRESS.md` row → drift; surface and ask.
- `code-review/<section>.md` exists but for a section not in `build/PROGRESS.md` → drift; surface and ask.

## Postconditions

- `build/code-review/<section>.md` exists with each new finding appended in the canonical shape, numbered per-file.
- For every finding that had a Coding Agent response, you have either appended a verified Resolution tag or left it untagged with a follow-up finding.
- No Coding Agent response lines or tags were written by anyone but their owner.
- `build/PROGRESS.md` and `build/testing/<section>.md` are untouched by this procedure. The Review Agent never edits `build/PROGRESS.md` — Review-column flips are the Coding Agent's, on each auto-launch return (SKILL.md "Auto-launch protocol"); testing-doc edits are the Coding Agent's.
