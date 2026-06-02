name: team-workflow
description: Enforce feat/fix branches, Conventional Commits, and final-approve gate.

# Team Workflow Skill (Contest Mode)

## Purpose

Use this skill when implementing work under tight delivery windows where branch
discipline and merge readiness are mandatory.

## Required Rules

1. Branch must be `feat/*` or `fix/*`.
2. Never push directly to `main`; open an MR.
3. Commit message must follow Conventional Commits:
   `type(scope): subject`.
4. Do not push commits containing `WIP`.
5. Auto-push is allowed only if all required checks pass and `FINAL_APPROVE=1`.

## Execution Checklist

1. Confirm current branch pattern is valid.
2. Confirm task scope is clear and implementation is complete.
3. Run available checks in order:
   - lint + unit
   - integration
   - smoke
4. Confirm commit message is Conventional Commits compliant.
5. Push branch and create/update MR.
6. Record verification results in `verification_log`.

## Failure Handling

- If branch naming is invalid: create a compliant branch and continue.
- If checks fail: stop push, fix failures, rerun checks.
- If `FINAL_APPROVE` is missing for auto-push: stop and request approval.
