# Codex Modernization Playbook

A reusable methodology for AI-assisted codebase modernization with Codex. Use this playbook to give Codex a consistent, high-quality approach to migrating legacy projects.

## What this is

`PLAYBOOK.md` is a phased methodology that Codex can follow for migrations. It is technology-agnostic and focuses on ordering, verification gates, and delivery discipline.

## How to use it in a new project

### Step 1: Add the playbook

Copy `PLAYBOOK.md` into the root of the project you want to modernize, or keep it in a `playbooks/codex` folder and reference it.

### Step 2: Create `AGENTS.md`

Codex follows `AGENTS.md`. Add one at the project root with instructions to use the playbook. Example:

```markdown
# Codex Instructions

Before starting any modernization or migration work, read and follow
playbooks/codex/PLAYBOOK.md.

When creating a migration plan, use the phase structure and step
ordering from the playbook. Do not skip the verification gates.
```

### Step 3: Provide the migration goal

Start the request with a clear target, for example:

> Migrate this project from [old framework] to [new framework].
> Follow playbooks/codex/PLAYBOOK.md.

Codex will then:

1. Read the codebase (Discovery)
2. Produce a step-by-step plan (Planning)
3. Execute the plan (Execution)
4. Verify build and runtime gates (Verification)
5. Commit and open a PR (Delivery)

### Step 4: Supply a reference project (recommended)

If a modern reference project already exists in the repo, tell Codex to match it:

> Use `path/to/reference-project/` as the reference for DI, hosting, config, and folder structure.

## Eval gates (mandatory)

A migration is not complete unless both pass:

- Build gate: 0 compile errors.
- Runtime gate: app starts and returns HTTP 200 on the root URL.

## Customization

Add project-specific steps or gates if needed, for example:

- Tests pass
- Lint/format check
- API smoke tests

Keep the playbook focused on migration behavior, not refactoring.
