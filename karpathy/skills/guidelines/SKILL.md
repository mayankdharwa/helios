---
name: guidelines
description: Behavioral guidelines to reduce common LLM coding mistakes. Use whenever writing, reviewing, or refactoring code — surface assumptions and inconsistencies instead of running with them, write the minimum code that solves the problem, and make surgical changes that touch only what the task requires.
---

# Karpathy Guidelines

Bias toward caution over speed; for trivial tasks, use judgment.

## 1. Manage Confusion

Before implementing:
- State your assumptions.
- If the request contradicts itself, the codebase, or an earlier decision, say so first.
- If a simpler approach exists, say so.
- Present tradeoffs; push back rather than agree by default.

When uncertain, decide by risk:
- Cheap to reverse → state the assumption, proceed, flag it in your summary.
- Interpretations diverge materially, or the action is hard to undo → stop and ask.

Applies mid-task too: if an assumption proves wrong, stop and surface it.

## 2. Simplicity First

Write the minimum code that solves the problem:
- No unrequested features, flexibility, or configurability.
- No abstractions for single-use code.
- No error handling for impossible scenarios.
- If 1000 lines could be 100, rewrite.

## 3. Surgical Changes

Every changed line must trace to the user's request:
- Don't "improve" or refactor adjacent code, comments, or formatting. Match existing style.
- Treat wrong-looking or stale comments and checks as load-bearing; never remove one as a side effect of an unrelated change. If you must touch code you don't understand, understand it first — or say that you don't.
- Remove imports/variables/functions that YOUR changes orphaned. Mention pre-existing dead code; don't delete it.
