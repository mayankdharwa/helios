# Supersede a prior decision

Mark an older `DECISIONS.md` entry as superseded by a newer one. Mini's shape and procedure are identical to the full skill's — `io:develop-feature/procedures/supersede.md` is canonical. This file captures only the mini-specific deltas; read the full procedure for the steps.

## Mini-specific deltas

- `Source:` paths on mini entries point to `PLAN.md` (`## Exploration notes` / `## Approach`) or `REVIEW.md`, not `exploration/<topic>/*-EXPLORATION.md` or `build/code-review/<section>.md`. The supersession marker and `Supersedes:` line make no reference to source paths, so this detail does not affect the procedure shape — only the surrounding entries.
- Inline `Review comment` callouts in `PLAN.md` and `decision:` tags in `REVIEW.md` / `## Review notes` stay untouched, same rule as the full skill.
- Cross-feature ripple (full procedure step 5) applies only if mini has previously lifted a decision to repo-root `docs/DECISIONS.md` via `io:develop-feature/procedures/cross-feature-lift.md`. Mini doesn't have its own cross-feature mechanic — cross-feature scope is itself a graduation signal (SKILL.md "When to use this skill vs `io:develop-feature`"). If supersession would force a repo-root edit, surface graduation instead of doing a one-off cross-file write.

## Preconditions, steps, errors, postconditions

Read `io:develop-feature/procedures/supersede.md` verbatim. Substitute mini's `Source:` paths where the full procedure refers to exploration / code-review source docs.

## When to invoke

- The user is recording that an older `#M` no longer holds because of a newer `#N`. Confirm-before-edit; one supersession action per invocation (multiple older entries against one newer entry is fine — confirm each).
- After a fresh lift via `procedures/lift.md` when the new entry replaces a prior one. Run `lift.md` first; invoke `supersede.md` immediately after.

## Graduation signal

A pattern of repeated supersessions (3+ in a single mini lifecycle) usually means the feature's design is still in flux at a scale mini's flat `DECISIONS.md` isn't built to track. Surface as a graduation candidate.
