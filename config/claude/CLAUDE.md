# CLAUDE.md

## Core Rule

This project uses forge-lite as the primary engineering workflow.

The only primary artifacts are:

1. proposal
2. design
3. tasks

Do not create extra primary planning documents unless explicitly requested.

## Workflow

Follow this order:

1. proposal
2. design
3. tasks
4. implementation
5. verification

Implementation must not begin before proposal, design, and tasks exist or the user explicitly allows skipping them.

## Fast Track

For small, low-risk changes, the user may request to skip proposal, design, and tasks.

Qualifying changes:

- typo or wording fix
- config value adjustment
- single-line bug fix
- rename or formatting change
- adding or removing a simple dependency

How to use:

- The user says "fast track", "quick fix", or "直接改"
- Or the change is obviously trivial (1-3 lines, no logic change)

What is skipped:

- proposal
- design
- tasks

What is kept:

- inspect existing code before changing
- make minimal necessary changes
- verify the change works after applying

## Artifact Ownership

Primary artifacts are the source of truth:

- proposal defines what to build and what not to build
- design defines how to build it
- tasks define executable implementation steps

Skills and agents may review, challenge, or improve artifacts, but must not replace the forge-lite artifact structure.

## Skills

Use skills only when relevant.

Default gstack skills:

- office-hours
- review
- investigate
- qa-only

Default superpowers skills:

- test-driven-development
- systematic-debugging
- verification-before-completion
- requesting-code-review

Optional skill:

- brainstorming

Do not use brainstorming during tasks or implementation unless explicitly requested.

If a listed skill is not available, skip it silently and proceed without it. Do not block work because a skill is missing. Apply the skill's intent manually if possible (e.g., if test-driven-development is unavailable, still follow TDD discipline yourself).

## Role Behavior

Agents act as reviewers and specialists.

Agents must:

- point out unclear assumptions
- identify missing edge cases
- check implementation risks
- produce concrete changes
- avoid vague advice

Agents must not:

- invent requirements
- expand scope without approval
- create competing proposal/design/tasks documents
- overwrite existing artifacts without user approval

## Engineering Rules

Before changing code:

1. inspect existing files
2. understand current conventions
3. make minimal necessary changes
4. preserve existing style
5. avoid broad rewrites

Before marking work complete:

1. run relevant tests if available
2. explain what was changed
3. list verification performed
4. list remaining risks if any

## Git Workflow Guardrails

For team delivery and contests, follow these rules:

- Work only on `feat/*` or `fix/*` branches.
- Never push directly to `main`; use MR only.
- Commit messages must follow Conventional Commits:
  `type(scope): subject`
- Do not push `WIP` commits to remote.
- Before push, run available checks in this order:
  lint + unit, then integration, then smoke.
- Auto-push/auto-commit is allowed only when all required checks pass and
  `FINAL_APPROVE=1` is set.

## Output Style

Be concise.
Be direct.
Prefer code, commands, diffs, and concrete file paths.
Do not over-explain obvious steps.
