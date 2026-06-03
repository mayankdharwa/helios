# Start a mini feature

Create `docs/<feature>/` scaffolding for a new mini feature.

## Preconditions

- User intends to start a small feature (≤ a day, 1–2 areas). If the scope feels larger, suggest `io:develop-feature` instead.
- Working directory has a `docs/` (or the user agrees to create it).
- No directory at `docs/<feature>/` already — collision is an error (see below).

## Steps

### 1. Confirm scope fits mini

One-line check before any writes. If the user describes:
- 3+ distinct exploration areas, or
- Multi-day work, or
- A cross-feature decision they already know is coming,

surface this and recommend `io:develop-feature` instead. Proceed only on user confirmation.

### 2. Ask for feature name

Validate: lowercase kebab-case, no spaces. Suggest a name based on what the user described.

### 3. Ask for OBJECTIVE.md content (substantive)

Walk the user through the four sections — do not write substance on their behalf:

- **What we're building** (1–3 sentences) — required.
- **Why** (1–3 sentences) — required.
- **In scope** (bulleted list) — may start empty.
- **Out of scope** (bulleted list) — may start empty.

Capture exact phrasing. Do not rewrite. Do not proceed past step 3 until What and Why have user-provided content — the rendered `OBJECTIVE.md` must never contain placeholder text. If the user isn't ready to write What or Why, stop and resume when they are.

### 4. Create the feature dir

Path: `docs/<feature>/`. Create silently after content is gathered:

- `OBJECTIVE.md` — from `templates/objective.md`, every `<...>` placeholder replaced (required sections) or an empty list (optional sections).
- `PLAN.md` — from `templates/plan.md`, with the feature name in the H1.

**Do NOT create:**
- `OPEN-QUESTIONS.md` — lazy; created on first question by the conversational flow (no procedure needed at mini scale).
- `DECISIONS.md` — lazy; created on first lift (see `procedures/lift.md`).
- `REVIEW.md` — lazy; created on review opt-in (see `procedures/review.md`).

### 5. Summarise and prompt for first exploration

Report: feature dir created, list the files, and prompt:

> Exploration notes — what do you want to figure out first?

The conversation continues by filling out `PLAN.md` `## Exploration notes` directly. No further procedure is invoked until the user is ready to advance.

## Errors

- Feature name already taken (dir exists at `docs/<feature>/`): surface, ask for a different name; do not overwrite.
- `docs/` doesn't exist: create it.
- `docs/<feature>/` already exists and contains `exploration/`, `spec/`, or `build/`: that's a full `develop-feature` feature — surface and route the user to `io:develop-feature` instead.

## Postconditions

- `docs/<feature>/OBJECTIVE.md` exists with no placeholder text in required sections.
- `docs/<feature>/PLAN.md` exists with four section headers (Exploration notes, Approach, Build checklist, Review notes) and no other files in `docs/<feature>/`.
