# <feature>

<!--
Single working doc for a mini feature. Sections double as phase progress: whichever
section is being actively written is the current phase. No PROGRESS.md, no status icons.

Lift sources:
- `> [!NOTE] Review comment` callouts in `## Exploration notes` or `## Approach`.
- `decision:` tags in `## Review notes` (if review opted in but Review Agent off) or REVIEW.md (if Review Agent on).

At phase transitions, `procedures/advance.md` sweeps the relevant source against the lift
criteria (hard to reverse / surprising without context / real trade-off) and lifts qualifying
entries to DECISIONS.md via `procedures/lift.md`. DECISIONS.md is lazy — created only on first lift.
-->

## Exploration notes

<!--
Freeform. Replaces the four-topic split in io:develop-feature. Capture what you're figuring
out and any decisions you lock in along the way as `Review comment` callouts (shape from
io:develop-feature/templates/review-comment.md):

> **Review comment #N** *(YYYY-MM-DD)*
>
> **Decision:** <what was decided>
> **Why:** <rationale, including alternatives considered>

Numbers (#N) are per-feature monotonic across PLAN.md and REVIEW.md combined.
-->

## Approach

<!--
Distilled plan. Aim for ≤1 page of bullets — this is the "spec" equivalent. Locked-in
calls can carry callouts here too; sweep happens when advance flips Exploration → Approach
and again on Approach → Build.
-->

## Build checklist

<!--
- [ ] item 1
- [ ] item 2

Tick boxes as code lands. When the last box is ticked, advance.md offers the review opt-in.
-->

## Review notes

<!--
Only filled when review is opted in AND the Review Agent is off (inline self-review). When
the Review Agent is on, this section stays empty and REVIEW.md carries the findings.

Tags here use the same `decision:` shape as the Review Agent so the lift sweep catches them:

> **Resolution:** decision: <text> *(YYYY-MM-DD)*
-->
