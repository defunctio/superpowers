# Codex Autonomous Brainstorming Fork Notes

This document tracks how the `codex-autonomous-brainstorming` branch differs from upstream `obra/superpowers`.

## Branch Purpose

This branch is intended for autonomous Codex agents that should continue through the design phase without waiting for routine human approval.

The goal is not to remove design discipline. The goal is to remove unnecessary human checkpoints while keeping:

- explicit design work before implementation
- documented assumptions
- review loops
- escalation only for genuinely high-risk ambiguity

## Operating Assumptions

This branch is optimized for GitHub/Codex agent workflows with these assumptions:

- the agent may start from an assigned issue rather than an interactive chat
- new issue comments should not be relied on as an in-flight control channel
- routine iteration happens after publication, typically through pull request comments
- the default "successful completion" path is to publish branch updates safely, not to ask what should happen next

Because of that, this branch prefers:

- assumptions over routine clarification prompts
- explicit blockers over conversational waiting states
- PR-first publishing behavior
- preserving recovery paths such as branches and worktrees

## Comparison Base

- Upstream repository: `obra/superpowers`
- Upstream branch: `main`
- Fork branch: `codex-autonomous-brainstorming`

At the time this document was last updated, this branch contains fork-specific changes for autonomous workflow execution on top of upstream `main`.

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

### 2. `skills/writing-plans/SKILL.md`

The planning workflow no longer asks the user to choose an execution mode.

- Upstream asks the user to choose between `subagent-driven-development` and `executing-plans`
- This fork defaults to `subagent-driven-development`
- `executing-plans` is treated as a fallback only when subagents are unavailable or a higher-priority instruction requires inline execution

### 3. `skills/using-git-worktrees/SKILL.md`

The worktree setup flow no longer blocks on routine location or baseline prompts.

- If no worktree directory convention exists, this fork defaults to `.worktrees/`
- If baseline tests fail, this fork requires the agent to assess whether the failures block the requested work instead of asking the user whether to continue

### 4. `skills/executing-plans/SKILL.md`

The inline execution flow no longer assumes a live human checkpoint.

- Low-risk concerns should be resolved autonomously
- High-risk blockers should be reported clearly rather than converted into synchronous clarification prompts

### 5. `skills/finishing-a-development-branch/SKILL.md`

The branch-completion workflow has been changed from an interactive integration menu to an autonomous publishing workflow.

- Upstream presents four choices: merge locally, create PR, keep branch, or discard
- This fork never merges locally by default
- This fork never discards work automatically
- This fork prefers pushing updates to an existing PR branch
- If no PR exists, this fork pushes the branch and creates a PR
- This fork preserves the branch and worktree by default for recoverability

### 6. `skills/subagent-driven-development/SKILL.md`

The subagent controller now resolves most missing context autonomously.

- Implementer questions are treated as context-resolution work for the controller
- The controller should resolve ambiguity from the plan, codebase, surrounding tasks, or explicit assumptions
- Human escalation is reserved for genuinely high-risk blockers that cannot be resolved safely

### 7. `skills/subagent-driven-development/implementer-prompt.md`

Implementer subagents are instructed to investigate and assume before escalating.

- Low-risk ambiguity should be handled with reasonable assumptions
- `NEEDS_CONTEXT` is reserved for genuinely blocking gaps
- `BLOCKED` is reserved for cases where safe progress is not possible

### 8. `skills/test-driven-development/SKILL.md`

The TDD workflow no longer assumes a synchronous human exception path.

- Exception cases now require a higher-priority instruction rather than an interactive permission check
- "Don't know how to test" now resolves toward simplification or an explicit blocker instead of a human prompt

### 9. `skills/systematic-debugging/SKILL.md`

The debugging workflow no longer assumes a human architecture checkpoint after repeated failed fixes.

- After 3 failed fixes, the branch requires either an explicit architectural redesign or a documented blocker
- It no longer instructs the agent to pause for discussion before taking either of those paths

## Why This Exists

This branch exists to support agents that are expected to work with minimal supervision inside automated or semi-automated Codex flows.

In that environment, the default upstream requirement for repeated user approvals and routine integration choices creates unnecessary stalls. The fork adjusts the workflow so the agent can continue unless there is a decision that is genuinely risky to make unilaterally.

## How To Review Future Fork Changes

To see the current fork delta against upstream:

```bash
git fetch upstream main
git log --oneline upstream/main..HEAD
git diff --stat upstream/main...HEAD
git diff upstream/main...HEAD
```

When adding new fork-specific behavior, update this document so it remains the authoritative summary of why the branch diverges from upstream.
