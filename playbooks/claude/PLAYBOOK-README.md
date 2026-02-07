# Legacy Modernization Playbook

A reusable methodology for AI-assisted codebase modernization. Use this playbook to give Claude Code (or any AI coding agent) a consistent, high-quality approach to migrating legacy projects.

## What this is

`PLAYBOOK.md` captures a structured 5-phase approach to legacy modernization:

1. **Discovery** — understand the codebase before touching it
2. **Planning** — produce an explicit, ordered migration plan
3. **Execution** — apply changes in dependency order
4. **Verification** — build gate, runtime gate, smoke tests
5. **Delivery** — clean commits and a descriptive PR

It is **technology-agnostic**. The phases, sequencing, and eval conditions apply whether you're migrating a .NET app, a Java monolith, a Python 2 codebase, or anything else.

## How to use it in a new project

### Step 1: Add the playbook to your repo

Copy `PLAYBOOK.md` into the root of the project you want to modernize. Or, if you prefer to keep it separate, place it in a shared location and reference it.

### Step 2: Point Claude Code at it

Add a `CLAUDE.md` file to the root of your project with the following:

```markdown
# Instructions

Before starting any modernization or migration work, read and follow
the methodology in PLAYBOOK.md.

When creating a migration plan, use the phase structure and step
ordering from the playbook. Do not skip the verification gates.
```

Alternatively, you can paste the playbook content directly into `CLAUDE.md` if you want it always loaded.

### Step 3: Provide the migration goal

When you start a conversation, tell Claude Code what the target state is. For example:

> Migrate this project from [old framework] to [new framework].
> Follow the playbook in PLAYBOOK.md.

Claude Code will then:

1. Read the codebase (Phase 1)
2. Produce a step-by-step plan for your approval (Phase 2)
3. Execute the plan (Phase 3)
4. Verify the build compiles and the app starts (Phase 4)
5. Commit and create a PR (Phase 5)

### Step 4: Supply a reference project (optional, but valuable)

If you have an existing project in the repo that already uses the target framework, tell Claude Code about it:

> Use `path/to/reference-project/` as the reference for modern patterns.

This dramatically improves quality — the agent will match the reference project's conventions for DI registration, hosting, configuration, and directory structure instead of guessing.

## Eval conditions

The playbook defines two mandatory gates. A migration is not considered complete unless both pass:

| Gate | Criteria | How to verify |
|---|---|---|
| **Build** | 0 compiler errors | `build-command` exits 0 |
| **Runtime** | App starts and serves requests | HTTP 200 on root URL |

These are non-negotiable. A migration that doesn't compile or doesn't start is not a migration — it's a broken codebase.

## Customizing the playbook

The playbook is a starting point. Adapt it to your team's needs:

- **Add domain-specific steps**: If your codebase has auth, background jobs, or message queues, add steps for those between the service layer and controller migration.
- **Add test gates**: If the project has tests, add a "tests pass" gate after the build gate.
- **Add lint/format gates**: If your team enforces style, add a formatting check.
- **Remove steps that don't apply**: If the project has no static assets, skip that step.

## What this is NOT

- **Not a framework migration guide.** It doesn't tell you which API replaces which. That's what the reference project and framework documentation are for.
- **Not a refactoring methodology.** The playbook explicitly separates migration from refactoring. Migrate first, refactor later.
- **Not a testing strategy.** It provides smoke-test gates, not a comprehensive test plan.
