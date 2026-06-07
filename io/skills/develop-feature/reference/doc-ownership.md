# Doc ownership

Read before editing any tracked file. Edits land per the role and the path.

**Ownership applies to content edits, not lifecycle moves.** Archive moves (`git mv <file> _archive/...`), file splits, and renames are lifecycle transitions executed by whichever procedure owns the transition. They do not require the file-owner role to perform them. The relevant procedure gates the move with its own preconditions (e.g., section-archive requires every item Review-Agent-tagged before moving `code-review/<section>.md`).

- `exploration/<topic>/*-EXPLORATION.md`, topic `PROGRESS.md` — edit freely.
- `exploration/custom/INDEX.md`, `exploration/custom/PROGRESS.md`, `exploration/custom/<sub-exploration>/*.md` — edit freely (same posture as standard exploration topics). The skill handles bookkeeping: directory creation, appending INDEX.md hook lines, adding/flipping PROGRESS.md rows. The authoring loop inside a sub-exploration folder is described in `reference/custom-shape.md`; lifecycle in `procedures/custom.md`.
- `build/testing/<section>.md`, `build/PROGRESS.md` — edit freely.
- `build/code-review/<section>.md` — Review Agent owns the file. Review Agent appends findings (shape in `templates/code-review-finding.md`; entry + loop in `procedures/code-review.md`) and, after independently verifying the Coding Agent's work, writes the resolution tag (`fixed` / `fixed: <note>` / `wont-fix: <reason>` / `decision: <text>` / `spec-changed: <link>`). The `fixed: <note>` form is optional and reserved for resolutions that aren't an ordinary code change (e.g., the clean-review sentinel `fixed: no findings`) — see `templates/code-review-finding.md`. Coding Agent appends a `> **Coding Agent response**` block (shape below) under each item — one line describing the action taken or, for a refusal, the reason — but does not write tag lines. Neither role edits the other's lines. Pure mechanical fixes made by the Coding Agent during its own work (not surfaced by a Review Agent) need no entry.

  Response block shape:
  > **Coding Agent response** *(YYYY-MM-DD)*
  >
  > **Action:** fixed | refused | deferred | spec-change-needed
  > **Detail:** <one line — what was changed, or why refused, or pointer to OPEN-QUESTIONS / surgical-reopen>

  Review Agent reads the response, verifies against the code, and writes the tag. If the Review Agent disagrees with a refusal, it leaves the item untagged and appends a follow-up finding — the section cannot archive until every item carries a tag, which forces resolution.
- `OPEN-QUESTIONS.md` — add entries freely. Resolving requires user confirmation.
- `spec/*`, `OBJECTIVE.md`, `DECISIONS.md`, `MIGRATION-CHECKLIST.md`, `TECH-DEBT.md`, `references/*` — confirm with user before any edit. For `DECISIONS.md`: existing entries are otherwise append-only — the only permitted in-place edits to a prior entry are the supersession markers added by `procedures/supersede.md` (the `*Superseded by #N — YYYY-MM-DD*` line on the older entry and the `**Supersedes:** #M[, …]` line on the newer one); `Decision:` / `Why:` / `Implications:` / `Source:` on every touched entry stay intact.
- `_archive/*` — never edit. Only move files in.

## Source code (outside `docs/<feature>/`)

Writeable by the Coding Agent freely. Writeable by a **Coding Agent Assistant** only within the `Scope:` paths declared at its invocation (`templates/delegation-prompt.md`, entry procedure `procedures/assist.md`). The Assistant has no write access anywhere under `docs/<feature>/` — every file listed above (`OBJECTIVE.md`, `DECISIONS.md`, `MIGRATION-CHECKLIST.md`, `TECH-DEBT.md`, `OPEN-QUESTIONS.md`, `PROGRESS.md` at any depth, `spec/*`, `build/testing/<section>.md`, `build/code-review/<section>.md`, `references/*`, `_archive/*`, and all exploration files) is Coding-Agent-only regardless of what an Assistant's `Scope:` appears to allow. The ceremony exclusion always wins over a permissive `Scope:`.

Spec ambiguities, tech-debt smells, missing-dependency surprises, or anything else the Assistant encounters but can't resolve from spec alone are returned in its summary as `Needs Coding Agent attention:` items — never self-logged to OPEN-QUESTIONS, TECH-DEBT, DECISIONS, or routed to surgical-reopen by the Assistant. The Coding Agent reads the summary and routes via `reference/issue-routing.md`.
