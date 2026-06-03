<!-- Replace placeholders in <angle-brackets> when filling this in. One finding per item. -->

> **Finding #N** *(YYYY-MM-DD)* — <one-line summary>
>
> **Where:** <file:line or symbol under review>
> **Targets:** <one form: a comma-separated list of sub-rows like `§2.1, §2.3`, OR the bare section number `§2` for section-wide>
> **Severity:** blocker | major | minor | nit
> **Detail:** <what's wrong and why it matters — enough for the Coding Agent to act without a back-and-forth>
> **Suggested direction:** <optional — a way to address it; not binding on the Coding Agent>

<!--
The Review Agent appends the finding. The Coding Agent then appends its response block
directly beneath (shape in `reference/doc-ownership.md`). The Review Agent independently
verifies against the code and appends the resolution tag as the final line of the item:

> **Resolution:** fixed *(YYYY-MM-DD)*
> **Resolution:** fixed: <note> *(YYYY-MM-DD)*
> **Resolution:** wont-fix: <reason> *(YYYY-MM-DD)*
> **Resolution:** decision: <text> *(YYYY-MM-DD)*
> **Resolution:** spec-changed: <link> *(YYYY-MM-DD)*

`fixed` is bare by default. The optional `fixed: <note>` form annotates resolutions that
aren't an ordinary code change — e.g., the sentinel `fixed: no findings` written by the
Review Agent when a clean review produces zero real findings (see `procedures/code-review.md`
step 4), or any case where one short phrase clarifies what "fixed" means here. The note is
not a justification (that's `wont-fix:`) or a design call (that's `decision:`); reach for
those instead when applicable.

Finding numbers (`Finding #N`) are per-file monotonic — see `reference/sequence-rules.md`.
On file split they keep their original number. Numbers never reuse, even after archive.

`Targets:` drives the per-sub-row Review column in `build/PROGRESS.md`. List every §N.M
sub-item this finding touches (one or more). For findings that span the whole section
(architectural, naming-wide, cross-cutting), write `Targets: §N` — the bare section number
— which marks the finding section-wide and holds every sub-row at 🚧 until the finding is
tagged. The sentinel clean-review item (`fixed: no findings`) uses `Targets: §N` so its
already-written `Resolution: fixed: no findings` tag lets every sub-row flip ✅ on the
Initial trigger's return. Mapping: see `templates/progress-build.md` Review column rules.

The Coding Agent never writes a Resolution line. The Review Agent never edits the
Coding Agent's response block. See `reference/doc-ownership.md`.
-->
