# Advance to the next phase

The only state transition in mini. Moves PLAN.md from one section to the next and runs the lift sweep on the way.

## Preconditions

- A feature dir is active (`docs/<feature>/PLAN.md` exists).
- One of:
  - The user has signalled they're ready to move on ("approach is settled", "start building", "all done", etc.), **or**
  - `procedures/review.md` Path A step 6 is re-invoking this procedure because a review pass just closed (every finding in `REVIEW.md` carries a `> **Resolution:**` tag) — no user "ready" signal needed; the closed-review state is itself the trigger.

## Phases and transitions

The current phase is the last non-empty section in `PLAN.md`:
- Exploration notes filled, Approach empty → **Exploration phase**.
- Approach filled, Build checklist empty → **Approach phase**.
- Build checklist has at least one `[ ]` or `[x]` → **Build phase**.
- All checklist items `[x]` → **Build complete**.

## Steps

### 1. Confirm the transition

Communicate: which phase is ending, which begins, and that a lift sweep will run on the closing section. Wait for confirmation.

### 2. Run the lift sweep

Scan the closing section for lift sources (see SKILL.md "Decision lift"):

| Closing phase | Sweep target |
|---|---|
| Exploration → Approach | `## Exploration notes` — `Review comment` callouts |
| Approach → Build | `## Approach` — `Review comment` callouts (often empty) |
| Build complete (no review) | nothing |
| Build complete (review just closed) | `REVIEW.md` (Review Agent path) or `## Review notes` (inline path) — `decision:` tags |

For each candidate, apply the three lift criteria. Delegate each lift to `procedures/lift.md`. Lifts at one sweep point are batched — present the list to the user, get a per-item yes/no, then write.

Cross-feature decisions surfaced during the sweep are a strong **graduation signal** — route to `procedures/graduate.md` instead of proceeding.

### 3. Prompt for the next section

| New phase | Prompt |
|---|---|
| Approach | "Approach — distil the plan as bullets. Aim for ≤1 page." |
| Build | "Build checklist — what items do you want to track? Tick them as code lands." |
| Build complete | "All checklist items ticked. Want to do a review pass? Options: Review Agent subagent, inline self-review, or skip." Then `procedures/review.md` if opted in. |

### 4. Smell checks during sweep

- If the sweep finds 3+ qualifying lifts in a single phase, surface it: this often means the feature is bigger than mini and is a graduation signal.
- If `## Exploration notes` reads like 3+ distinct topics (schema + apis + jobs, etc.), surface this as a graduation signal *before* sweeping — the sweep itself is fine to run; the recommendation is to convert.

### 5. Summarise

Report: phase transition done, count of lifts (and which `#N` landed in `DECISIONS.md`), next concrete step.

## Errors

- Can't determine the current phase from `PLAN.md` shape (e.g., Build filled but Exploration empty): surface drift, ask user.
- Lift sweep finds a callout whose `Why:` is empty: surface, ask user to fill it before lifting or to drop the callout.

## Postconditions

- `PLAN.md`'s closing section is the same content it had before (sweeps don't edit — callouts stay in place).
- For each lift the user approved, a new `#N` entry exists in `DECISIONS.md` with a backlink.
- The next phase has been entered (a header prompt or first item).
