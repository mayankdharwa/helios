# Section-archive (build phase)

Close a build section when every item in its `code-review/<section>.md` is tagged. Triggers a DECISIONS sweep on `decision:` tags, then archives the file.

## Preconditions

- Build phase is active (`build/` exists; `build/PROGRESS.md`'s Active table shows the section's parent row with `Tests: ✅`, `Code: ✅`, `Review: ✅` — all three already flipped by the Coding Agent as their respective signals fired (tests written + green, code complete against green tests, review → fix → verify loop closed; the `Review: ✅` flip path is SKILL.md "Auto-launch protocol"). A ⏸ section can't archive — it has to be resumed first so the triple-columns can land on ✅. A parent row still showing any 🚧 means the corresponding signal hasn't fired yet — refuse and surface which column is not yet ✅.
- `build/code-review/<section>.md` exists with at least one item. (Sections with no real findings still satisfy this via the clean-review sentinel — see `procedures/code-review.md` step 4.)
- Every item carries a resolution tag on its final `> **Resolution:**` line (canonical shape in `templates/code-review-finding.md`): `fixed` (optionally `fixed: <note>`), `wont-fix: <reason>`, `decision: <text>`, or `spec-changed: <link>`. Tags are written by the **Review Agent** after independently verifying the Coding Agent's response and the code (see `reference/doc-ownership.md`).

If any item is untagged, surface and refuse to archive. The Coding Agent does **not** self-tag to clear the block — surface the untagged items, identify whether each is awaiting a Coding Agent response or awaiting Review Agent verification, and stop. Resolution lands through the normal review cycle (or user tie-break).

Lift criteria + immediate-lift mechanics: `reference/lift-criteria.md`.

## Steps

### 1. Verify all items tagged

Read `build/code-review/<section>.md`. Check every item. If any are untagged, surface the list with the current state of each:
- **No Coding Agent response yet** — awaiting Coding Agent action.
- **Response present, no Review Agent tag** — awaiting Review Agent verification.
- **Response and follow-up finding, no tag** — Review Agent disagreed; needs another response cycle or user tie-break.

Stop. Do not proceed. The Coding Agent does not write tags to unblock archive.

### 2. DECISIONS sweep on `decision:` tags

Enumerate items tagged `decision: ...`. Skip any already lifted via immediate-lift (check `DECISIONS.md` `Source:` lines). For each remaining, apply the three lift criteria from `reference/lift-criteria.md`.

> Item: "`<one-line summary of the item>`"
> Decision tag: "`<decision text>`"
> - Hard-to-reverse: Y / N
> - Surprising: Y / N
> - Real-trade-off: Y / N
> Recommend: **LIFT to DECISIONS.md #N | LEAVE inline**.
> Confirm / flip / re-discuss?

For confirmed LIFTs: append a new entry to `DECISIONS.md` using the canonical shape in `templates/decisions.md`. Determine `#N` per `reference/sequence-rules.md`. `Source:` is `[build/code-review/<section>.md item "<one-line summary>"]` — the code-review item stays in place (the lifecycle move at step 3 archives the file, but the entry's source link captures the live-path identity at lift time).

Cross-feature check: if the decision affects code outside this feature, route to `procedures/cross-feature-lift.md`.

### 3. Archive the file

Determine archive filename:
- Path: `build/_archive/code-review/<section>_<YYYY-MM-DD>_<seq>.md`
- `<seq>` per `reference/sequence-rules.md` (per-section monotonic across time).

Move: `git mv build/code-review/<section>.md build/_archive/code-review/<section>_<YYYY-MM-DD>_<seq>.md`.

The file is no longer at the live path — `ls build/code-review/` then shows only sections with open items.

### 4. `build/testing/<section>.md` stays in place

Do NOT archive or move the testing file. It's permanent reference material once code exists.

### 5. Update `build/PROGRESS.md`

Build PROGRESS has two tables — **Active** (sections that are 🚧 or ⏸) and **Pending / Done** (sections that are ⏳ or ✅). Section close moves one row between tables and promotes the next section into Active. Shape and column semantics: `templates/progress-build.md`.

**5a. Move the closing section from Active to Pending / Done.**

- Remove the parent row AND all its sub-rows (§N.1, §N.2, ...) from the Active table.
- Insert a flat parent row (no sub-rows) into the Pending / Done table at its `§` slot — not at the end. § order is preserved across the table regardless of completion order, so a section that closes out-of-order lands between its neighbours, not after the last row. All three triple-columns are already `✅` (precondition) and carry over unchanged. This may place a ✅ row between ⏳ rows, which is fine; the triple encodes which is which.

The `Review: ✅` flip happened *before* archive — when the verify loop returned all-tagged, the Coding Agent flipped the column per SKILL.md "Auto-launch protocol" and only then invoked this procedure. Step 1 has already re-verified that every `code-review/<section>.md` item carries a Review Agent tag; the column flip and the tag invariant are two sides of the same gate.

**5b. Promote the next section into Active.**

Pick the next section in this order:

1. **Resume a paused section whose blockers have all cleared.** Markers stack (a section can be `— BLOCKED by Q5, Q7` if two Qs blocked it at different times — see `procedures/open-questions.md` step 5), so a section is eligible to resume only when *every* Q in its marker is resolved.

   For each ⏸ row in the Active table:
   - Parse the `— BLOCKED by Q<a>, Q<b>, ...` marker on the parent row into a list of Q-numbers.
   - For each Q-number, check whether it's in `OPEN-QUESTIONS.md`'s resolved-index.
   - **All Qs resolved** → row is eligible. Don't mutate yet — collect across all ⏸ rows first.
   - **Some Qs resolved, some still active** → prune the resolved Q-numbers from the marker (rewrite the comma-separated list with only the still-active ones); the row stays ⏸ and is not eligible. The prune keeps the marker truthful so future archive passes don't re-check resolved Qs.
   - **No Qs resolved** → leave the row unchanged.

   From the collected eligibles, promote one:
   - **Zero eligible** → fall through to 5b.2.
   - **Exactly one eligible** → resume it: flip the parent row's three triple-columns ⏸ → 🚧, flip any ⏸ column on the previously-frozen sub-row back to 🚧 (untouched columns stay ⏳; finished columns stay ✅), and drop the **entire** `— BLOCKED by Q<list>` marker from the section name. No row movement — the section was already in Active.
   - **Multiple eligible** → surface them and ask which to resume; only the chosen one promotes (drop its marker per the single-eligible case). The 1 🚧 cap holds; the others stay ⏸ in Active with their now-fully-resolved markers intact (the next archive pass will re-pick from them, or the user can resume one manually). Flag them in the report so the user knows the choice is revisitable.

2. **Otherwise pick the next `⏳` section by `§` order.** Remove its row from Pending / Done, add a parent row to Active with `Tests` = 🚧, `Code` = 🚧, `Review` = ⏳, and expand its sub-rows from the top-level headings in `testing/<section>.md` — one sub-row per heading; cells start at `Tests` = ⏳, `Code` = ⏳, `Review` = ⏳, `Pointer` = —. The `Review` column flips on the first auto-launch return (SKILL.md "Auto-launch protocol").

**5c. Update `## Current focus`.** First, invoke `procedures/checkpoint.md` on `build/PROGRESS.md` (per its "Auto-trigger from `procedures/section-archive.md`" section — steps 2 + 3 only, no rewrite, no table-drift). Anything liftable in the about-to-be-erased focus block gets routed to its canonical home before the rewrite below overwrites it. Then branch on what 5b actually did — three cases:

- **5b promoted a section** (resumed a fully-cleared ⏸ via 5b.1, or started a fresh ⏳ via 5b.2):
  - Rewrite the active-section block (name + short qualifier + 2–3 line state summary + next intra-section step) to reflect the newly-promoted section.
  - Update the `Paused:` line — remove the just-resumed section if 5b.1 resumed one; keep the rest. Omit the whole line when no sections are paused.

- **5b couldn't promote, but ⏸ rows remain in Active** (no fully-cleared ⏸, no ⏳ left — only sections with at-least-one-active-blocker remain): partial-empty Active edge state (see `templates/progress-build.md` edge-states block).
  - Leave the Active table as-is. ⏸ rows remain; no new 🚧 row.
  - Replace the active-section block in `## Current focus` with the single-line placeholder:
    > `_No section actively driven — §<X> paused on Q<N>. Resolve Q<N> to resume, or start the next ⏳ section._`
    
    Use any ⏸ section from the `Paused:` line as `§<X>` for orientation — the most recently parked (last entry in `Paused:`) is the natural pick, since the user was working closest to it. Reference one of its blockers as `Q<N>`; the marker on the row carries the canonical list.
  - Leave the `Paused:` line intact (still the canonical roster of parked work).
  - Do **not** phase-lock. Do **not** archive `build/PROGRESS.md`. The build phase is parked, not done — work resumes when a blocking Q resolves and the next archive (or a manual resume) picks one of the ⏸ rows up.

- **Build phase-lock** (no ⏳ left to promote, no ⏸ rows in Active — every section is ✅): Active becomes empty.
  - Replace the table body with the pre-archive placeholder row:
    > `_no active section — all sections archived_`

  - Set the Current focus narrative to:
    > Build locked. All sections ✅ — archiving build/PROGRESS.md.

  - Then archive `build/PROGRESS.md` (path: `build/_archive/PROGRESS_<YYYY-MM-DD>_<seq>.md`, per-directory monotonic across time — counter does not reset per day).

  - Update top-level `PROGRESS.md`: `🚧 build → ✅`.

### 6. Summarise

Report: section archived to its destination path, sweep results (count of `decision:` tags considered and how many lifted to DECISIONS.md), that `build/testing/<section>.md` stays in place, and what 5b/5c produced — one of: section §X promoted into Active (which 5b path: resumed-⏸ or new-⏳); partial-empty Active state (work parked behind blockers — name the remaining ⏸ sections); or build phase-lock (`build/PROGRESS.md` archived, top-level flipped). If 5b.1 pruned resolved Q-numbers from any non-promoted ⏸ markers, list those too.

## Post-archive findings

If new findings surface for this section after archive, **create a new** `build/code-review/<section>.md` (do NOT edit the archive). When that new file closes, archive with the next per-section sequence number. The new file is independent; cross-references happen via git history if needed.

## Errors

- Untagged items present: surface, stop.
- Sequence number collision (shouldn't happen with correct read-then-write): surface and ask.
- Section name doesn't match any `code-review/*.md`: drift — surface and ask.
- Section row missing from `build/PROGRESS.md`'s Active table (or present but ⏸): drift — surface and ask. A ⏸ section needs to be resumed before archive; a missing row means the parent/sub-rows weren't materialised when the section flipped ⏳ → 🚧.
- A ⏸ section's blocker tag (`Q<N>`) doesn't resolve to a question in `OPEN-QUESTIONS.md`: drift — surface and ask.

## Postconditions

- `build/code-review/<section>.md` no longer exists at the live path.
- `build/_archive/code-review/<section>_<YYYY-MM-DD>_<seq>.md` exists with `<seq>` one greater than any pre-existing archive of THIS section.
- `build/testing/<section>.md` is untouched.
- The closing section's row is in `build/PROGRESS.md`'s **Pending / Done** table at its `§` slot with `Tests: ✅`, `Code: ✅`, `Review: ✅`; its sub-rows are gone; it is no longer in the Active table.
- 5b/5c left Active in exactly one of three states:
  - **Promoted**: a section is now 🚧 in Active — either an ⏸ row with all blockers cleared resumed (marker dropped, in-flight sub-row columns flipped ⏸ → 🚧) or a fresh ⏳ row moved in with sub-rows materialised from `testing/<section>.md`. `## Current focus` names it.
  - **Partial-empty (parked)**: no eligible ⏸ and no ⏳ remained, but ⏸ rows still exist in Active. The Active table retains those ⏸ rows; the active-section block in `## Current focus` is replaced with the partial-empty placeholder line; the `Paused:` line is intact; `build/PROGRESS.md` is **not** archived; top-level `PROGRESS.md` still shows `🚧 build`.
  - **Phase-lock**: every section is ✅. Active shows the pre-archive placeholder; `build/PROGRESS.md` is archived to `build/_archive/PROGRESS_<YYYY-MM-DD>_<seq>.md`; top-level `PROGRESS.md` shows `build: ✅`.
- Any partially-resolved ⏸ markers in Active have been pruned (resolved Q-numbers removed from the comma-list; row stays ⏸ if any active Q remained).
- For each confirmed lift: `DECISIONS.md` has a new entry whose `Source:` backlinks to the code-review item.
