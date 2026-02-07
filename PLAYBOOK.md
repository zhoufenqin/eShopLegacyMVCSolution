# Legacy Modernization Playbook

A structured approach for AI-assisted modernization of legacy codebases. Designed to be used as a system prompt or instruction file for Claude Code (or similar AI coding agents) when migrating projects to modern frameworks and runtimes.

---

## Phase 1: Discovery

Before writing a single line of code, build a complete mental model of the project.

### 1.1 Identify the project boundary

- Map every project/module in the solution and their dependency graph.
- Identify which projects are in scope for migration and which are shared dependencies.
- Note the target framework, language version, and package manager for each project.

### 1.2 Inventory the tech stack

For each project, catalog:

- **Framework & hosting model** (e.g., how the app boots, where middleware lives, how routes are registered).
- **Dependency injection** — what container is used, how services are registered, what lifetimes are configured.
- **Data access** — ORM, connection string source, model configuration approach, seed/initialization strategy.
- **Configuration** — where settings live, how they're read at runtime.
- **Static assets** — where they're stored, how they're referenced in views/templates, any bundling/minification pipeline.
- **Legacy plumbing** — files that exist only because the old framework required them (manifests, XML configs, assembly info, global entry points).

### 1.3 Find a reference implementation (if one exists)

If a partially migrated or sister project exists in the repo that already targets the destination framework:

- Read it thoroughly. It is your primary source of truth for patterns, conventions, and DI registration style.
- Note any gaps — things the reference project didn't need to handle that the legacy project does.

### 1.4 Read every source file you will touch

Do not propose changes to code you haven't read. For each file in scope:

- Read the full file (not just a grep hit).
- Note which legacy APIs it uses that will need to change.
- Note which parts are already portable and can stay as-is.

---

## Phase 2: Planning

Produce an explicit, step-ordered migration plan before executing anything.

### 2.1 Sequence the work

Order steps so that **infrastructure changes land first** and **dependent changes follow**. A reliable default order:

1. **Project file / build system** — target framework, package references, SDK style.
2. **Entry point / hosting** — replace the legacy bootstrap with the modern hosting model.
3. **Configuration** — migrate settings to the modern config system.
4. **Data access layer** — ORM, context, model configuration, migrations tooling.
5. **Data initialization / seeding** — remove framework-coupled base classes, inject modern services.
6. **Service layer** — update namespace imports, adjust API surface as needed.
7. **Controllers / request handlers** — migrate base classes, return types, parameter binding, DI.
8. **Models / DTOs** — remove legacy namespace imports.
9. **Views / templates** — replace bundling helpers, add framework-required import files.
10. **Static assets** — move to the location the modern framework expects.
11. **Delete legacy files** — remove everything the modern framework no longer needs.
12. **Build & verify** — the plan isn't done until it compiles.

### 2.2 For each step, state the file and the transform

Be precise: "In file X, change A → B." This makes each step independently reviewable and reduces the chance of drift between the plan and the execution.

### 2.3 Decide what to keep, what to replace, and what to drop

Explicitly categorize every dependency and legacy file:

- **Keep**: Libraries that work on both old and new (e.g., logging, serialization).
- **Replace**: Framework-coupled libraries that have a modern equivalent (e.g., old ORM → new ORM, old DI container → built-in DI).
- **Drop**: Things that have no equivalent or are no longer needed (e.g., legacy telemetry, bundling frameworks, XML configs, assembly info).

---

## Phase 3: Execution

### 3.1 Work in the planned order

Execute each step from the plan sequentially. Do not jump ahead — later steps depend on earlier ones compiling.

### 3.2 Preserve behavior, not structure

The goal is **behavioral equivalence**, not a line-for-line translation.

- Match the original DI lifetimes (singleton, scoped, transient).
- Preserve the same routes, controller names, action signatures.
- Keep the same database schema and table names.
- Keep the same view rendering behavior.

Don't "improve" code during migration. Refactoring and migration are two different tasks — mixing them makes it impossible to verify the migration succeeded.

### 3.3 Track known patterns

As you work, you'll discover recurring API translations (e.g., `OldThing()` → `NewThing()`). Record them. They'll repeat across files and across projects. Refer to your memory/notes before each file to avoid re-discovering the same mapping.

### 3.4 Handle static assets carefully

- Move (not copy) files so git tracks them as renames.
- Preserve the original directory structure unless the team has explicitly decided to reorganize.
- Update all references in views/templates to match the new paths.

### 3.5 Delete decisively

After migration, delete every legacy file that the modern framework doesn't need. Don't leave dead files around "just in case." The old code is in git history.

---

## Phase 4: Verification

### 4.1 Build gate (mandatory)

```
The project MUST compile with 0 errors.
```

Run the build immediately after completing all steps. If it fails:

- Read every error.
- Fix the root cause (usually a missing `using`, a missing package, or an API that changed signature).
- Rebuild.
- Repeat until 0 errors.

Warnings are acceptable in this phase but should be noted for follow-up.

### 4.2 Runtime gate (mandatory)

```
The application MUST start and respond to a basic HTTP request.
```

- Start the app with mock/in-memory data (no external database dependency).
- Confirm the root URL returns HTTP 200.
- Confirm the app logs show successful startup (listening on a port, no exceptions).

### 4.3 Functional smoke test (recommended)

If time permits, verify:

- The main page renders (not a blank page or exception page).
- At least one CRUD operation works end-to-end.
- API endpoints return valid responses.

### 4.4 Diff review

Before committing, review the full diff:

- Are there files that were accidentally included (build output, IDE settings)?
- Are there files that should have been deleted but weren't?
- Do the git renames show as renames (not delete + add)?

---

## Phase 5: Commit & Deliver

### 5.1 Commit strategy

- Separate concerns into logical commits (e.g., "migrate project X" and "migrate project Y" as separate commits).
- Write commit messages that explain **what was migrated and why**, not just "updated files."
- Don't mix migration commits with unrelated changes.

### 5.2 PR description

Include:

- A summary of what was migrated (frameworks, libraries, patterns).
- What was verified (build, runtime, smoke tests).
- What was NOT verified and needs manual testing.

---

## Anti-patterns to avoid

| Anti-pattern | Why it's bad |
|---|---|
| Changing code you haven't read | You'll break things you don't understand |
| Mixing refactoring with migration | Impossible to verify behavioral equivalence |
| Copying static files instead of moving | Loses git history, creates duplicates |
| Reorganizing directory structure during migration | Adds unnecessary risk and review burden |
| Keeping dead legacy files "just in case" | Confuses future developers, pollutes the build |
| Skipping the build gate | Errors compound; catching them early is cheaper |
| Skipping the runtime gate | A project that compiles but won't start is not migrated |
| Guessing at API translations | Read the docs or the reference project; guessing causes subtle bugs |

---

## Checklist (copy into your task tracker)

```
[ ] Phase 1: Discovery
    [ ] Mapped project dependency graph
    [ ] Inventoried tech stack per project
    [ ] Read reference implementation (if available)
    [ ] Read every source file in scope

[ ] Phase 2: Planning
    [ ] Produced ordered step list
    [ ] Each step specifies file + transform
    [ ] Categorized all dependencies (keep / replace / drop)

[ ] Phase 3: Execution
    [ ] Completed steps in order
    [ ] Recorded recurring API translation patterns
    [ ] Moved (not copied) static assets
    [ ] Deleted all legacy files

[ ] Phase 4: Verification
    [ ] Build: 0 errors
    [ ] Runtime: app starts, HTTP 200 on root URL
    [ ] Diff review: no accidental inclusions or omissions

[ ] Phase 5: Delivery
    [ ] Logical commit separation
    [ ] Descriptive commit messages
    [ ] PR with summary and test status
```
