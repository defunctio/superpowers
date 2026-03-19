# Codex Autonomous Brainstorming Fork Notes

This document tracks how the `codex-autonomous-brainstorming` branch differs from upstream `obra/superpowers`.

## Branch Purpose

This branch is intended for autonomous Codex agents that should continue through the design phase without waiting for routine human approval.

The goal is not to remove design discipline. The goal is to remove unnecessary human checkpoints while keeping:

- explicit design work before implementation
- documented assumptions
- review loops
- escalation only for genuinely high-risk ambiguity

## Comparison Base

- Upstream repository: `obra/superpowers`
- Upstream branch: `main`
- Fork branch: `codex-autonomous-brainstorming`

At the time this document was written, the diff from upstream `main` to this branch consists of a single commit:

- `79147e2` `feat: make brainstorming autonomous by default`

## Current Differences From Upstream

### 1. `skills/brainstorming/SKILL.md`

The `brainstorming` skill has been changed from an interactive approval workflow to an autonomous default workflow.

#### Upstream behavior

Upstream `brainstorming` assumes a human-in-the-loop process:

- asks clarifying questions one at a time
- proposes options and waits for user choice or approval
- requires user approval before moving past design
- requires user review of the written spec before moving to planning
- proactively offers the visual companion when relevant

#### Fork behavior

This fork changes the default behavior to:

- infer requirements from the request and codebase where possible
- make reasonable assumptions for low-risk ambiguity
- ask the user a question only when the missing information is truly blocking and would materially affect architecture, scope, or safety
- generate 2-3 approaches, choose the recommended option automatically, and document why
- write the spec without waiting for routine user approval
- proceed to `writing-plans` after the spec review loop passes
- avoid proactively offering the visual companion in this branch

#### Important constraint retained

The branch still preserves a design gate before implementation:

- implementation must not start before a design pass is completed
- a spec still needs to be written
- the spec review loop is still required

This is an autonomous design workflow, not a "skip design" workflow.

## Why This Exists

This branch exists to support agents that are expected to work with minimal supervision inside automated or semi-automated Codex flows.

In that environment, the default upstream requirement for repeated user approvals creates unnecessary stalls. The fork adjusts the workflow so the agent can continue unless there is a decision that is genuinely risky to make unilaterally.

## How To Review Future Fork Changes

To see the current fork delta against upstream:

```bash
git fetch upstream main
git log --oneline upstream/main..HEAD
git diff --stat upstream/main...HEAD
git diff upstream/main...HEAD
```

When adding new fork-specific behavior, update this document so it remains the authoritative summary of why the branch diverges from upstream.
