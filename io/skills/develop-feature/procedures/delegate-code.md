# Delegate code (build phase — Coding Agent → Assistant)

The Coding Agent's procedure for handing a mechanical code-writing task to a Coding Agent
Assistant. The Assistant runs in a fresh subagent session, writes only inside a declared
file scope, and returns a structured summary. The Coding Agent retains all ceremony work
(`build/PROGRESS.md` column flips, `code-review/<section>.md` responses, `DECISIONS.md` /
`TECH-DEBT.md` / `OPEN-QUESTIONS.md` entries).

Role model: `SKILL.md` "Roles" and "Coding Agent delegates to Assistant". Ownership rules:
`reference/doc-ownership.md`. The Assistant's own entry: `procedures/assist.md`.

## Preconditions

- Build phase is active and a section is `🚧` in `build/PROGRESS.md`'s Active table.
- The work to delegate is "mechanically clear from spec" by the heuristic below. If it
  isn't, don't delegate — keep it in-context.
- For parallel spawns: every concurrent Assistant's `Scope:` must be disjoint from every
  other. The Coding Agent enforces no-overlap *before* dispatch.

If the section is `⏸`, refuse — paused work has to be resumed first.

## When to delegate (heuristic)

Delegation is discretionary; no auto-launch. Lean toward delegating when:

- A sub-row `§N.M` has a clear test target in `build/testing/<section>.md` and the spec
  leaves little judgement (signatures fixed, behaviour described, edge cases enumerated).
- The task is a large mechanical translation from `spec/*` or `testing/<section>.md` into
  code — function bodies, multi-file scaffolding, plumbing, boilerplate test setup.
- Two or more independent sub-rows can be implemented in parallel against disjoint files.

Lean toward keeping in-context when:

- The code is cited by an open Review finding (the response cycle is ceremony).
- The work spans integration glue between previously-delegated outputs (judgement on
  contracts, not mechanical translation).
- The spec has open ambiguity, an unresolved `Q<N>`, or the section is mid-review with
  untagged findings.
- The change is small enough that delegation overhead exceeds the context saved.

The heuristic is a recommendation, not a gate. The Coding Agent's judgement governs.

## Steps

### 1. Frame the delegation

Identify, before spawning:

- The section name (must match a `🚧` parent row in `build/PROGRESS.md`).
- The exact `Scope:` — repo-relative paths or globs the Assistant may write to. Be tight:
  only what the tasks need. Wider scope is harder to verify on return.
- The `Tasks:` — one bullet per concrete unit of work, each naming the target path.
- The `Spec refs:` — the spec headings and `build/testing/<section>.md` headings the
  Assistant needs to read to fulfil the contract.

Shape: `templates/delegation-prompt.md`.

### 2. Parallel dispatch — check scope disjointness

If dispatching two or more Assistants in one turn:

- Collect each Assistant's `Scope:`. Expand globs mentally (or with a quick `ls`) enough to
  rule out overlap.
- If two scopes overlap on any file path, the dispatch is invalid. Either:
  - Tighten the scopes so they're disjoint, or
  - Serialise — dispatch one Assistant, wait for return, then dispatch the next.
- Never spawn parallel Assistants with overlapping scopes. The Coding Agent owns this
  invariant; the Assistant has no view of its siblings.

### 3. Spawn

Spawn each Assistant via the Agent tool: `subagent_type: general-purpose`. The prompt
body is the filled-in template from `templates/delegation-prompt.md`. The first action of
the spawned subagent is `Skill(skill="io:develop-feature", args="assist")`, which lands in
`procedures/assist.md`.

For parallel dispatch, send a single message with multiple Agent tool calls. For serial
dispatch, one Agent call, wait for return, then the next.

Announce the spawn in one line per Assistant:
`Spawning Assistant for <section> (scope: <Scope:>, tasks: <count>).`

### 4. On return — verify and update ceremony

Run the four sub-steps below **per Assistant return**. With parallel dispatch, complete all
verify / test / flip / route work for one return before starting the next — never interleave.
Interleaving risks flipping `Code: ✅` on a test run that hasn't accounted for the other
Assistant's changes, or routing a `Needs Coding Agent attention:` item against a
half-updated ceremony state. Process returns in arrival order; if the order matters for
correctness (e.g., one Assistant's output depends on another's), the dispatch should have
been serial in the first place.

The Assistant returns the summary shape defined in `procedures/assist.md` step 6:

```
Section: <section-name>
Files changed:
  - <path> — <created | edited> — <one-line why>
Tests run: <command(s)> — <pass / fail / not run>
Needs Coding Agent attention:
  - <ambiguity or surprise the Assistant declined to resolve>
```

The Coding Agent then:

1. **Verify scope containment.** Run `git status` (or equivalent) and confirm every changed
   file in the summary sits inside the declared `Scope:`. If the Assistant reports an
   out-of-scope write in its summary, or one shows up in `git status` that the Assistant
   didn't list, treat the delegation as failed: revert the out-of-scope writes, surface to
   the user, do not flip `build/PROGRESS.md`.
2. **Run tests** for the section's test target — either the command the Assistant ran (to
   confirm) or the section's broader test suite. The Coding Agent's flip of `Tests` / `Code`
   columns rests on the Coding Agent's own observation of green, not the Assistant's claim.
3. **Flip `build/PROGRESS.md` columns** exactly as the Coding Agent would for in-house
   writes — `Tests` / `Code` on the parent and sub-rows per the signals in
   `templates/progress-build.md`. The flip path is unchanged by the delegation; only the
   author of the code differs.
4. **Route `Needs Coding Agent attention:` items** via `reference/issue-routing.md`. An
   ambiguity may become an `OPEN-QUESTIONS.md` entry; a smell may become a `TECH-DEBT.md`
   item; a spec-change need triggers `procedures/surgical-reopen.md`. The Assistant never
   self-logs these — that's why they come back in the summary.

After step 3's column flips, the existing `## Auto-launch protocol` (SKILL.md) takes over
just as it would for any green-after-code-change moment — the Review Agent auto-launches on
its normal triggers (Initial / Verify / Recheck). Delegation does not interact with the
Review Agent's protocol directly; it only changes who wrote the code.

## Errors

- **Section not `🚧`** → surface and stop. The Coding Agent doesn't delegate against a
  paused, pending, or archived section.
- **Scope overlap on parallel dispatch** → surface, do not spawn. Tighten or serialise.
- **Assistant writes outside `Scope:`** → revert the offending writes, do not flip
  PROGRESS columns, surface to the user. The Assistant should have refused; if it didn't,
  treat as an Assistant bug to flag.
- **Assistant writes anywhere under `docs/<feature>/`** → revert immediately, surface to the
  user. This is a hard boundary violation regardless of `Scope:`.
- **Assistant returns without running tests when tests exist** → the Coding Agent runs them
  itself before flipping `Code: ✅`. Not an error, but a missing column-flip signal.

## Postconditions

- The Coding Agent has verified every Assistant-changed file sits inside its declared `Scope:`.
- No ceremony file under `docs/<feature>/` was edited by the Assistant.
- `build/PROGRESS.md` columns reflect the Coding Agent's own observation of test/code state,
  not the Assistant's claim.
- Any `Needs Coding Agent attention:` items from the summary have been routed via
  `reference/issue-routing.md` (or explicitly noted as handled).
- The Review Agent auto-launch protocol picks up on its normal triggers — delegation does
  not bypass it.
