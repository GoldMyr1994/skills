---
name: docs-workflow
description: >
  Three-command documentation skill for a before/after workflow.
  Use this skill whenever the user wants to read, summarize, or get caught up
  on a docs folder — even if they don't say "brief me" explicitly. Trigger
  phrases include "brief me", "brief-me", "read the docs in", "load context
  from", "catch me up", "what's in these docs", "summarize the docs",
  "what do we have".
  Also use when the user wants to write or update documentation after finishing
  work: "update-docs", "update the docs in", "write docs after my changes",
  "document what I did", "write up what I did".
  Also use for code standards: "update-standards", "update the standards in",
  "new coding standard", "add a standard".
  Append "with issues" to brief-me or update-docs to include GitHub issue work.
---

# docs-workflow

Three commands for a before/after documentation workflow.

---

## Bare invocation

**When**: the skill is called without a recognized command keyword.

Present the user with selectable options using an interactive selection tool (e.g., `vscode_askQuestions`) if available. If no interactive tool is available, fall back to a text prompt.

Options to present:
- **brief-me** — load context from a docs folder
- **update-docs** — update docs after completed work
- **update-standards** — update code standards files
- **Include GitHub issues?** — yes / no

Wait for the user to pick before proceeding.

---

## GitHub CLI rules

Only apply when the user appends **"with issues"** to a brief-me or update-docs command.

- Read issues with `gh issue view <number>` or `gh issue view <url>`.
- List issues with `gh issue list` only if the user asks to find related issues without specifying numbers.
- Edit issues with `gh issue edit`: supported flags are `--title`, `--body`, `--add-label`, `--remove-label`, `--add-assignee`, `--remove-assignee`, `--milestone`, `--remove-milestone`.
- Close issues with `gh issue close`: supported flags are `--reason {completed|not planned|duplicate}`, `--comment`, `--duplicate-of`.
- Do not use `--web`, authentication commands, or pre-flight auth checks.
- If a `gh` command fails, report it and continue with remaining work.

---

## Doc writing rules

Apply to all docs created or updated by this skill.

- **Progressive disclosure**: lead with the essential fact; add detail only if needed to avoid ambiguity. (Readers scan first, read second — front-load what matters.)
- **Minimal**: no filler words, no introductory phrases, no summaries of what you just said. (Saves context window and reader attention.)
- **No code**: prose only — no code blocks, inline code, or command snippets. Exception: files written by `update-standards`. (Keeps docs tool-agnostic and readable by non-developers.)
- **500-word cap**: each file must not exceed 500 words. Split into multiple files if needed. (Forces atomic files — easier to navigate and update independently.)

---

## brief-me

**Use when**: the user is about to start work and wants to load context from an existing docs folder.

**Steps**:

1. Read all files in the specified path recursively.
2. If invoked **with issues**: read any issues referenced by number or URL; if the user asks to find related issues, use `gh issue list`.
3. Synthesize what exists: key topics, how things relate, what is complete vs. incomplete.
4. List explicit TODOs, open questions, or pending work found in the docs (and issues, if loaded).
5. Note anything incomplete, missing, or unclear.
6. End with: "Context loaded. Ready when you are."

**Output format**:

## Summary
...

## Key Topics
- ...

## Gaps
- ...

## TODOs
- ...

If the path doesn't exist or has no readable files, say so and stop.

---

## update-docs

**Use when**: the user has finished work and wants to create or update docs to reflect what was done.

### Phase 1 — Plan

1. Identify what changed from the conversation. If the user mentions branches or commits, run targeted git commands to refine your understanding.
2. Read existing docs in the specified path. If the path is empty or doesn't exist, ask the user to confirm the path or whether to create it before continuing.
3. Note the conventions of existing docs — naming patterns, heading style, tone, file organization — and carry them into Phase 2.
4. If invoked **with issues**: include issue updates or closures in the plan. Use `ISSUE-UPDATE <number>` or `ISSUE-CLOSE <number>` with the exact fields to change.
5. Produce a plan. For each change include: action (`CREATE`, `UPDATE`, `ISSUE-UPDATE`, or `ISSUE-CLOSE`), target, one-line reason.

**Plan format**:

## Doc Update Plan

- UPDATE path/to/existing.md — reason
- CREATE path/to/new.md — reason
- ISSUE-UPDATE #123 — add-label "done", remove-label "in-progress"
- ISSUE-CLOSE #456 — reason: completed

Proceed with writing? (yes / no)

Stop and wait for approval.

### Phase 2 — Write

6. Execute the approved plan. Match the conventions identified in step 3. Apply doc writing rules above. For issue actions use `gh issue edit` and `gh issue close`. Report each as SUCCEEDED or FAILED.
7. List what was created, updated, or changed.

If you can't determine what changed, ask the user before producing the plan.

---

## update-standards

**Use when**: the user says "update-standards", "update the standards in", "new coding standard", or "add a standard" followed by a path.

**Purpose**: create or update code standards files. Code is allowed here.

**Scope**: standards define *how* code should be written — naming conventions, formatting rules, architectural patterns, decision records. Regular docs (handled by `update-docs`) describe *what* exists and *what* was done. If unsure whether something is a standard or a doc, ask the user.

### Phase 1 — Plan

1. Read existing standards files in the specified path.
2. Identify what standards to add, change, or remove based on the conversation.
3. Produce a plan. For each change include: action (`CREATE` or `UPDATE`), target path, one-line reason.

**Plan format**:

## Standards Update Plan

- UPDATE path/to/standards.md — reason
- CREATE path/to/standards.md — reason

Proceed with writing? (yes / no)

Stop and wait for approval.

### Phase 2 — Write

4. Execute the approved plan. Each standards file may contain code examples. Keep prose minimal; let the examples speak.
5. List what was created or updated.
