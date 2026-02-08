# Codex Legacy Modernization Playbook

A structured approach for AI-assisted modernization of legacy codebases. Designed to be used as an instruction file for Codex (or similar AI coding agents) when migrating projects to modern frameworks and runtimes.

---

## Phase 1: Discovery

Before writing code, build a complete model of the project.

### 1.1 Identify project boundaries

- Map every project/module and their dependency graph.
- Identify which projects are in scope and which are shared dependencies.
- Note target frameworks, language versions, and package managers.

### 1.2 Inventory the stack

For each project, catalog:

- Framework and hosting model
- Dependency injection container and registrations
- Data access layer and migrations
- Configuration sources
- Static assets and bundling pipeline
- Legacy-only plumbing (global entry points, assembly info, XML configs)

### 1.3 Find a reference implementation

If a modernized or similar project exists:

- Read it fully and treat it as the source of truth for patterns.
- Note any gaps the reference does not cover.

### 1.4 Read every file you will touch

Do not propose changes to code you have not read end-to-end.

---

## Phase 2: Planning

Produce an explicit, ordered plan before executing.

### 2.1 Order of operations

A reliable default order:

1. Project file and build system
2. Entry point and hosting
3. Configuration
4. Data access layer
5. Data initialization and seeding
6. Service layer
7. Controllers and request handlers
8. Models and DTOs
9. Views and templates
10. Static assets
11. Remove legacy-only files
12. Build and verify

### 2.2 Specify file-level transforms

For every step, name the file and describe the exact transform.

### 2.3 Categorize dependencies and legacy files

- Keep: works on old and new
- Replace: modern equivalent exists
- Drop: no longer needed

---

## Phase 3: Execution

### 3.1 Follow the plan order

Do not jump ahead. Later steps depend on earlier ones compiling.

### 3.2 Preserve behavior, not structure

- Match original DI lifetimes
- Preserve routes and action names
- Preserve database schema and table names
- Preserve view rendering behavior

Do not refactor while migrating.

### 3.3 Track translation patterns

Record recurring API translations and reuse them consistently.

### 3.4 Handle static assets carefully

- Move, do not copy, to preserve git history
- Preserve directory structure unless required otherwise
- Update all references in views

### 3.5 Delete legacy files decisively

Remove files the modern framework no longer needs.

---

## Phase 4: Verification

### 4.1 Build gate (mandatory)

The project must compile with 0 errors.

### 4.2 Runtime gate (mandatory)

The application must start and return HTTP 200 on the root URL.

### 4.3 Smoke tests (recommended)

- Main page renders
- One CRUD action works
- API endpoints return valid responses

### 4.4 Diff review

Check for accidental inclusions and missing deletions.

---

## Phase 5: Delivery

### 5.1 Commit strategy

- Separate logical concerns
- Descriptive commit messages
- No unrelated changes

### 5.2 PR description

Include:

- What changed
- What was verified
- What was not verified

---

## Anti-patterns to avoid

- Changing code you have not read
- Mixing refactoring with migration
- Skipping build or runtime gates
- Reorganizing directories unnecessarily
- Leaving dead legacy files

---

## Checklist (copy into task tracker)

```
[ ] Discovery complete
[ ] Plan approved
[ ] Project file migrated
[ ] Entry point migrated
[ ] Configuration migrated
[ ] Data layer migrated
[ ] Service layer migrated
[ ] Controllers migrated
[ ] Views migrated
[ ] Static assets migrated
[ ] Legacy files removed
[ ] Build gate passes
[ ] Runtime gate passes
[ ] PR opened
```
