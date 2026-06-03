<!-- Replace placeholders in <angle-brackets> when filling this in. This is the prompt body the
Coding Agent passes to the Agent tool when spawning a Coding Agent Assistant. The spawned
subagent's first action is `Skill(skill="io:develop-feature", args="assist")`. -->

```
Section: <section-name>
[Feature: <feature-name>]
Scope: <comma-separated paths or globs the Assistant may write to>
Tasks:
  - <one delegated task — function/class/module to implement, with target path>
  - <another delegated task>
Spec refs:
  - <pointer to spec doc — e.g., docs/<feature>/spec/<file>.md#<heading>>
  - <pointer to build/testing/<section>.md and the relevant heading(s)>
```

<!--
Required fields: Section, Scope, Tasks. Spec refs is strongly recommended; without at least
one ref the Assistant has no contract to implement against and will stop on the first task.

Feature: is required when more than one feature dir under `docs/` has an active build phase;
with a single active feature the skill picks it.

Scope: rules:

  - Comma-separated repo-relative paths or globs (e.g., `src/foo/bar/*.ts, src/foo/bar/__tests__/*.test.ts`).
  - The Assistant may write only inside these paths. Reads are unconstrained.
  - Anything under `docs/<feature>/` is implicitly excluded — the Assistant refuses such writes
    regardless of what Scope: declares. The ceremony tree is Coding-Agent-only (see
    `reference/doc-ownership.md`).
  - For parallel spawns in one turn, every Assistant's Scope: must be disjoint from every
    other concurrent Assistant's Scope:. Overlap is a Coding Agent error; surface and stop
    before spawning.

Tasks: rules:

  - One task per bullet. Each task should name the target file path(s) (within Scope) and
    the concrete unit of work (function, class, module, test file).
  - Avoid bundling unrelated work — separate Assistants for separate concerns keeps return
    summaries readable and parallelism viable.

Spec refs: rules:

  - Point at specific headings, not whole files, when the doc is long. The Assistant reads
    only what's needed to fulfil the contract.
  - Include `build/testing/<section>.md` heading(s) so the Assistant knows what the tests
    assert (or will assert) about the code it's writing.

The Assistant returns a structured summary (shape in `procedures/assist.md` step 6) that the
Coding Agent reads, verifies for scope-containment, and then uses to update build/PROGRESS.md
columns. The Assistant never updates ceremony files itself.
-->
