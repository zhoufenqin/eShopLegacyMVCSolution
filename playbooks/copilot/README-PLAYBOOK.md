# How to Use the Modernization Playbook

## What Is This?

`PLAYBOOK.md` is a structured guide for migrating legacy framework applications to modern platform versions. It captures a proven approach — surgical, incremental, verification-driven — that produces clean migrations with minimal risk.

It is designed to be used as a **prompt/context document** for AI coding assistants, ensuring they follow the same disciplined approach every time.

## Quick Start

### 1. Point your AI assistant at the playbook

When starting a migration task, include the playbook as context. For example:

```
Follow the approach described in PLAYBOOK.md to migrate this project
from [source framework] to [target framework].
```

The assistant will use the phased approach, eval criteria, and anti-patterns as guardrails.

### 2. Start with Phase 0 (Analysis)

Always ask the assistant to analyze before implementing:

```
Analyze this project following Phase 0 of the playbook. 
Create a migration plan before making any changes.
```

Review the plan. Confirm the scope. Then proceed:

```
Implement the migration following the playbook phases.
```

### 3. Verify at each phase

The playbook defines **eval criteria** for every phase — these are the conditions that must be true before moving on. The assistant should check these automatically, but you can enforce them:

```
Before moving to the next phase, verify all eval criteria are met.
```

## When to Use This

- Migrating a web application from a legacy framework to a modern one
- Upgrading a project to a new major platform version with breaking changes
- Porting a project that uses a legacy ORM, DI container, or hosting model
- Any migration where behavioral equivalence must be maintained

## When NOT to Use This

- Greenfield rewrites (building from scratch, not migrating)
- Minor version upgrades with no breaking changes
- Projects that need architectural redesign (the playbook preserves existing architecture)

## Customizing the Playbook

The playbook is intentionally **framework-agnostic**. The phases and principles apply regardless of:

- Programming language
- Web framework (MVC, API, SPA backend)
- ORM / data access technology
- DI container
- Configuration system

To adapt it for a specific technology stack, you can add a companion document with stack-specific mappings:

```markdown
## Stack-Specific Mappings

| Legacy API | Modern Equivalent |
|---|---|
| (legacy controller base) | (modern controller base) |
| (legacy ORM context) | (modern ORM context) |
| (legacy DI registration) | (modern DI registration) |
| ... | ... |
```

## Key Behaviors the Playbook Enforces

1. **Minimum viable changes** — Only touch what must change for the new platform. Don't refactor, don't improve, don't modernize beyond what's required.

2. **Build-verify-proceed** — Compile after each phase. Run the app at key milestones. Never assume changes work.

3. **Bottom-up dependency order** — Migrate shared libraries first, then dependent projects. Within a project, migrate models → services → controllers → views.

4. **Preserve everything possible** — Same file locations, same routes, same config keys, same defaults, same business logic. The user should not notice the migration.

5. **No scope creep** — Don't add features, don't fix pre-existing bugs, don't adopt new patterns. A migration PR should contain only migration changes.

## Eval Criteria Summary

The playbook succeeds when ALL of these are true:

- [ ] Project builds from clean state with 0 errors
- [ ] Application starts without runtime exceptions  
- [ ] All original pages/endpoints return expected HTTP status codes
- [ ] All original API endpoints return valid responses
- [ ] No legacy-only files remain in the project
- [ ] No business logic was modified
- [ ] Original routes and URLs are preserved
- [ ] Configuration values and defaults are preserved
