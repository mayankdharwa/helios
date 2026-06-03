# OPEN-QUESTIONS lifecycle

Add, resolve, or mark irrelevant entries in `OPEN-QUESTIONS.md`.

## Modes

This procedure handles three actions:
- **Add** — create a new entry.
- **Resolve** — close an entry via one of four resolution paths.
- **Irrelevant** — mark an entry as no-longer-relevant due to scope change.

## Preconditions (all modes)

- Feature dir is active.
- `OPEN-QUESTIONS.md` exists (was planted at bootstrap).

---

## Mode: Add

### Steps

1. Determine next Q-number. Read `OPEN-QUESTIONS.md`, scan both indices (active + resolved) for highest Q<N>, use N+1. **Numbers never reuse.**
2. Ask user for substantive content:
   - Question text (one sentence as title).
   - Context (what prompted the question).
   - Options considered (if any).
   - Why deferred (what made it not-answerable-now).
   - Blocking (what task/unit/section depends on this).
3. Append to `## Index (active)`:
   ```
   - **Q<N>** — <title> (raised: YYYY-MM-DD, blocks: <pointer>)
   ```
4. Append to `## Details`:
   ```markdown
   ### Q<N> — <title>

   Context: <text>
   Options considered: <text or "none yet">
   Why deferred: <text>
   Blocking: <pointer>
   ```

5. **If `blocks:` names the currently 🚧 build section**, pause it in `build/PROGRESS.md` (pause mechanics — kept colocated here because raising a blocking Q is the only trigger):
   - Flip the parent row's three triple-columns 🚧 → ⏸ in the Active table.
   - Flip any 🚧 column on the currently in-flight sub-row to ⏸. Untouched columns stay ⏳; finished columns stay ✅.
   - Append ` — BLOCKED by Q<N>` to the section name in the `Section` column.
   - Update `## Current focus` for the partial-empty Active edge state (no section is 🚧 now — `templates/progress-build.md` carries the full spec):
     - Replace the active-section block (the `**<active-section-name>**` header + state summary + `Next up:` line) with the single-line placeholder:
       > `_No section actively driven — §<section> paused on Q<N>. Resolve Q<N> to resume, or start the next ⏳ section._`
     - Add or extend the `Paused:` line with `§<section> — BLOCKED by Q<N>`.

   If the named section is **already ⏸** (a second blocker stacking on existing pause), don't flip state — extend the existing marker to list both, e.g. `— BLOCKED by Q5, Q7`. Likewise extend the existing `Paused:` entry rather than duplicating it.

   If the named section is `⏳` (not started) or already in Pending / Done (archived), no PROGRESS change — the Q simply blocks future work and the marker isn't materialised on a row yet. (If a `⏳` section later promotes into Active while Q<N> is still open, the promote step in `section-archive.md` 5b should carry the marker over — but the current procedure assumes blockers raise after work starts; revisit if this case appears in practice.)

   The user decides what's next: start a different section through the normal flow, resume an earlier ⏸ section whose blocker has cleared (via the section-archive 5b path when the active section closes, or a manual swap), or stop and answer Q<N>. The Add flow does **not** auto-promote a replacement.

If the trigger was `procedures/defer.md`, return the Q-number for use in the deferred row marker.

### Summary

Report: the new Q-number and what it blocks. If step 5 paused (or extended the marker on) a build section, name the section and its new state.

---

## Mode: Resolve

### Steps

1. Identify Q<N> the user is resolving.
2. Ask which of the four resolution paths applies:

   - **DECISIONS lift** — answered and the decision is course-altering.
   - **Review comment** — answered, captured as a `Review comment` callout in an exploration doc (locks a specific unit).
   - **Discussion only** — answered in conversation; no artifact warranted (didn't lock a specific unit).
   - **No longer relevant** — scope changed; supersede with a pointer.

3. Based on the path:

   **DECISIONS lift:**
   - Route to `unit-lock` or `surgical-reopen` flow that creates the relevant Review comment + DECISIONS entry first.
   - Then update the active-index line to resolved-index:
     ```
     - **Q<N>** — ✅ Resolved → see DECISIONS.md #M
     ```

   **Review comment:**
   - Confirm the Review comment exists (path + number).
   - Move to resolved-index:
     ```
     - **Q<N>** — ✅ Resolved → see exploration/<topic>/<DOC>.md §X Review comment #M
     ```

   **Discussion only:**
   - Ask user for the one-line answer summary.
   - Move to resolved-index:
     ```
     - **Q<N>** — ✅ Resolved by discussion (no artifact — answer: <one-line summary>)
     ```

   **No longer relevant:**
   - Ask for the pointer (DECISIONS entry, exploration callout, or other artifact that supersedes).
   - Move to resolved-index:
     ```
     - **Q<N>** — ⊘ No longer relevant (scope changed; see <pointer>)
     ```

4. **Remove the `## Details` section for Q<N>**. The resolved-index line carries the title and pointer; details that mattered are in the resolution target. For `⊘ No longer relevant` resolutions, the parenthetical pointer on the index line itself carries the context (e.g., `(scope changed; see DECISIONS.md #7)`) — no separate inline note is written or preserved.

5. Check whether any `PROGRESS.md` rows reference this Q<N> via `BLOCKED by Q<N>`. If yes, surface to the user that the row may now be unblocked. Do not auto-unblock.

### Summary

Report: which Q-number resolved via which path, that details have been removed, and list any PROGRESS rows that referenced this Q<N> (so the user knows they may now be unblocked).

---

## Mode: Irrelevant (shortcut to "No longer relevant" path)

Identical to the "No longer relevant" resolution path above.

---

## Errors

- Q<N> doesn't exist: surface and list active questions.
- Q<N> is already resolved: surface; ask if user wants to amend the resolution.
- User wants to resolve but the resolution-path target doesn't exist (e.g., references a DECISIONS entry that hasn't been written): refuse; ask user to create the target first or pick a different path.

## Postconditions

**Add:** `OPEN-QUESTIONS.md` `## Index (active)` contains a `Q<N>` line where `<N>` is one greater than any pre-existing question number, and `## Details` contains a matching detail block. If `blocks:` named the 🚧 build section, that section is now ⏸ in `build/PROGRESS.md`'s Active table with the `— BLOCKED by Q<N>` marker on its `Section` cell; if it named an already-⏸ section, the marker now lists Q<N> alongside prior blockers; otherwise `build/PROGRESS.md` is unchanged.

**Resolve:** the entry no longer appears under `## Index (active)`; it appears under `## Index (resolved)` with one of the four resolution forms; the `## Details` block for this entry is removed (except for the `⊘ No longer relevant` form, which retains a one-line context).

**Irrelevant:** entry under `## Index (resolved)` carries `⊘ No longer relevant` and a pointer; one-line context retained.

