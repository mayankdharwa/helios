<!-- Replace placeholders in <angle-brackets> when filling this in. -->

## Current focus

**<active-section-name>** — <short qualifier>

<2–3 line state summary of the actively-worked section: which test/code aspect is open, what was last touched. Points at testing/code-review docs; does not restate substance.>

Paused: <§X — BLOCKED by Q<N>>, <§Y — BLOCKED by Q<M>>  <!-- omit the whole line when nothing is paused -->

Next up: <next concrete intra-section step>.

## Active

| § | Section | Tests | Code | Review | Pointer |
|---|---|---|---|---|---|
|   | _no active section — starting with §1_ |   |   |   |   |

## Pending / Done

| § | Section | Tests | Code | Review | Pointer |
|---|---|---|---|---|---|
| <N> | <section-name> | ⏳ | ⏳ | ⏳ | testing/<section>.md |
| <N> | <section-name> | ⏳ | ⏳ | ⏳ | testing/<section>.md |

<!--
Two tables, one source of truth per row. Every section lives in exactly one table at a time.

  - Active = sections that have started but aren't done — any row whose state is 🚧 or ⏸.
  - Pending / Done = sections that haven't started yet (⏳) or are fully archived (✅). Ordered by § (mixes ⏳ and ✅; the triple encodes which is which).

Section states (the parent row's lifecycle):

  - ⏳ pending — not started. Lives in Pending / Done.
  - 🚧 in flight — actively being worked. Lives in Active. Cap: exactly one 🚧 at a time.
  - ⏸ paused — started, halted on a blocker, will resume. Lives in Active. No cap on count.
  - ✅ done — archived. Lives in Pending / Done.
  - **Review-closed, archive-pending** (transient) — all three triple-columns are ✅ but the row is still in Active. The window between the Coding Agent's `Review: 🚧 → ✅` flip and the `section-archive` call (SKILL.md "Auto-launch protocol"). The two actions are intentionally separable so the column accurately reflects "review done, archive pending" if the user pauses or redirects between them. Counts as 🚧 for the 1-at-a-time cap until archive moves it out.

Row discipline: rows carry status icons + pointer + the inline `— BLOCKED by Q<N>` marker only. No rationale, no findings, no "noticed that X" notes. Substantive discussion lives in testing/<section>.md, code-review/<section>.md, OPEN-QUESTIONS, DECISIONS.

The five glyphs used in the triple:

  - ⏳ pending  · 🚧 in flight  · ⏸ paused (frozen mid-work)  · ✅ done  · — N/A (only in `Pointer` on sub-rows)

The three triple-columns move on different signals:

  - `Tests` — ⏳ until the test plan starts landing; 🚧 while red/incomplete; ✅ when the section's tests are written and green. The Coding Agent flips it.
  - `Code` — ⏳ until production code starts landing; 🚧 while in flux; ✅ when tests are green against the final implementation. The Coding Agent flips it.
  - `Review` (parent row) — ⏳ until the Review Agent has run for this section; 🚧 once the first Review Agent finding lands in `code-review/<section>.md` (the flip is keyed on the current cell value ⏳, not on whether the finding is already tagged); ✅ when every code-review item carries a Review-Agent tag (`fixed` / `wont-fix:` / `decision:` / `spec-changed:`). The Coding Agent flips it on the return of each auto-launched Review Agent subagent — 🚧 when the first findings land, ✅ when verify returns all-tagged. Both flips happen *before* `section-archive` is invoked; archive only verifies the column, it does not flip it. See SKILL.md "Auto-launch protocol" for the exact cases.
  - `Review` (sub-row §N.M) — derived from the section's findings, each of which carries a `Targets:` field (`templates/code-review-finding.md`). At rest, before any Review Agent has run, the cell is ⏳ (set by `procedures/section-archive.md` 5b.2 when the section enters Active). On each Review Agent auto-launch return, the Coding Agent recomputes the cell from the current findings:
    - 🚧 if any *untagged* finding cites this sub-row in `Targets:` — either directly (`§N.M`) or section-wide (`§N`, which holds every sub-row at 🚧).
    - ✅ otherwise (every finding citing it is tagged, or no finding cites it).
    A sub-row with zero findings citing it flips ⏳ → ✅ on the first Review Agent return, in the same step the parent flips ⏳ → 🚧 (or ⏳ → ✅ for a clean review).
    - Clean review (Review Agent finds nothing) is not a separate path: the Review Agent still creates `code-review/<section>.md` with a single sentinel item targeting `§N` and already tagged `fixed: no findings` — so the file-exists + all-tagged invariants hold uniformly. `Review` walks ⏳ → 🚧 → ✅ on the same edges; for a clean review both parent flips fire on the Initial trigger's return, and every sub-row walks ⏳ → ✅ in the same step.

Sub-row decomposition (Active table only): when a section flips ⏳ → 🚧, expand it with status-only sub-rows enumerating its constituent items. Anchor: one sub-row per top-level heading in `testing/<section>.md` (the test plan's own structure decides the count — no ad-hoc grouping). Sub-rows:

  - Use §N.M numbering in the § column, and a leading `↳ ` on the name to make hierarchy scannable.
  - Track `Tests`, `Code`, and `Review`. Sub-row `Review` follows the per-sub-row rules above — derived from each finding's `Targets:` field. Before any Review Agent has run, sub-row `Review` is ⏳ (matches the parent).
  - Omit the pointer (`Pointer` column is `—`). The parent's pointer already locates them.

Example mid-walk (one section actively driven, none paused; Review Agent has not yet run):

| § | Section | Tests | Code | Review | Pointer |
|---|---|---|---|---|---|
| 2 | section-B | 🚧 | 🚧 | ⏳ | testing/B.md |
| 2.1 | ↳ sub-item-A | ✅ | ✅ | ⏳ | — |
| 2.2 | ↳ sub-item-B | 🚧 | ⏳ | ⏳ | — |
| 2.3 | ↳ sub-item-C | ⏳ | ⏳ | ⏳ | — |

Example post-Initial (findings landed; §2.1 has an open finding, §2.3 has none):

| § | Section | Tests | Code | Review | Pointer |
|---|---|---|---|---|---|
| 2 | section-B | ✅ | ✅ | 🚧 | testing/B.md |
| 2.1 | ↳ sub-item-A | ✅ | ✅ | 🚧 | — |
| 2.2 | ↳ sub-item-B | ✅ | ✅ | ✅ | — |
| 2.3 | ↳ sub-item-C | ✅ | ✅ | ✅ | — |

Pause / resume (1 🚧 + N ⏸):

A section pauses when a blocker hits mid-work and we need to start another section to keep moving. Mechanics:

  - Parent row: in each triple-column whose value is 🚧, flip 🚧 → ⏸. Columns already at ⏳ or ✅ are unchanged (a column that hasn't been touched yet has nothing to freeze). Append ` — BLOCKED by Q<N>` to the section name in the `Section` column (pointer into OPEN-QUESTIONS — no rationale on the row).
  - On every sub-row: apply the same column-wise rule — any 🚧 cell flips to ⏸; ⏳ and ✅ are unchanged. The in-flight sub-row freezes mid-state and is identifiable at a glance because it's the only one carrying ⏸ in the columns that were in flight.
  - The newly-started section enters Active as 🚧 in the normal way (one 🚧 at a time invariant holds — the paused section is no longer 🚧).
  - Resume: in each parent or sub-row cell currently at ⏸, flip ⏸ → 🚧. Drop the `— BLOCKED by Q<N>` marker. No row movement. The previously-active section pauses (or archives, if it's done first).

Example mid-block (§3 paused on Q7 while §4 is actively driven):

| § | Section | Tests | Code | Review | Pointer |
|---|---|---|---|---|---|
| 3 | section-C — BLOCKED by Q7 | ⏸ | ⏸ | ⏳ | testing/C.md |
| 3.1 | ↳ sub-item-A | ✅ | ✅ | ⏳ | — |
| 3.2 | ↳ sub-item-B | ⏸ | ⏳ | ⏳ | — |
| 3.3 | ↳ sub-item-C | ⏳ | ⏳ | ⏳ | — |
| 4 | section-D | 🚧 | 🚧 | ⏳ | testing/D.md |
| 4.1 | ↳ sub-item-A | ✅ | ✅ | ⏳ | — |
| 4.2 | ↳ sub-item-B | 🚧 | ⏳ | ⏳ | — |

Section-archive (parent finishes; details in procedures/section-archive.md):

  1. Confirm all three triple-columns on the parent are already ✅ (the Coding Agent flipped each as its signal fired — Tests/Code on completion of work, Review on the verify loop closing) and every item in `code-review/<section>.md` carries a Review-Agent tag.
  2. Remove the parent row and its sub-rows from the Active table.
  3. Insert a flat parent row (no sub-rows) into the Pending / Done table at its § slot — not at the end. § order preserves the spec walk regardless of completion order.
  4. Promote the next section into Active: pick the next ⏳ row (or resume an ⏸ row whose blocker has cleared), flip its parent to 🚧, and expand its sub-rows from `testing/<section>.md`.

Edge states:

  Active empty (no 🚧 or ⏸ rows) — kickoff and pre-archive:

  - Kickoff (nothing started yet) and pre-archive (all sections done) both leave Active empty. Keep the `## Active` header and the table header row; put a single italic placeholder row directly under the header:
    - Kickoff: `_no active section — starting with §1_`
    - Pre-archive: `_no active section — all sections archived_`

  Partial-empty Active (⏸ rows but no 🚧) — paused-without-replacement:

  - Triggered by `procedures/open-questions.md` Add pausing the 🚧 section when the user hasn't yet started a replacement. The table is NOT empty (⏸ rows still hold the parked work) — only the actively-driven slot is unfilled.
  - The Active table renders normally — no placeholder row needed; the ⏸ rows speak for themselves.
  - The `## Current focus` block has no section to name. Replace its `**<active-section-name>** ... Next up:` body with a single line:
    - `_No section actively driven — §X paused on Q<N>. Resolve Q<N> to resume, or start the next ⏳ section._`
  - The `Paused:` line still appears below, listing all paused sections (including §X). It is allowed to repeat §X here — the Current focus line names it as the most recently paused; the `Paused:` line is the canonical roster.

  Goal: structure stays predictable across the lifecycle; readers always find the same two tables in the same place, and `## Current focus` always tells them where attention is (or isn't).
-->
