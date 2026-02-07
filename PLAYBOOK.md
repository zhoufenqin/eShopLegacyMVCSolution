# Legacy Framework Modernization Playbook

## Purpose
A repeatable playbook for modernizing legacy framework applications to current platform versions. Designed to be pointed at by an AI coding assistant (e.g., GitHub Copilot) to ensure consistent, high-quality migrations across projects.

---

## Core Principles

### 1. Surgical Changes Only
- Modify the **minimum number of lines** required to achieve compilation and runtime correctness.
- Preserve the original code structure, naming, and organization wherever possible.
- Do NOT refactor, restructure, or "improve" code beyond what is strictly needed for the migration.
- If a pattern works on the new platform (even if outdated style), leave it alone.

### 2. Preserve Behavioral Equivalence
- The migrated application must behave identically to the original from a user's perspective.
- All existing endpoints, routes, and page URLs must remain the same.
- All existing business logic must be untouched.
- Configuration values and defaults must carry over.

### 3. Incremental Verification
- Build after every logical group of changes. Never batch all changes and hope they compile.
- Run the application after each milestone to verify HTTP responses.
- Fix errors one at a time, from the bottom of the dependency graph up.

### 4. Dependency-Graph-First Ordering
- Always start with leaf dependencies (shared libraries, utilities) before dependent projects.
- Within a project, order work as: project file → models → data access → services → controllers → views → entry point.
- Never modify a file that depends on another file you haven't migrated yet.

---

## Migration Phases

### Phase 0: Deep Analysis
**Goal:** Understand the full scope before touching any code.

1. **Inventory all source files** — List every `.cs`, `.cshtml`, view, config, and static asset file.
2. **Map the dependency graph** — Which projects reference which? What NuGet packages are used and why?
3. **Identify framework-coupled APIs** — Find every usage of platform-specific namespaces (e.g., legacy web framework, legacy ORM, legacy DI container, legacy configuration system).
4. **Catalog breaking changes** — For each framework-coupled API, identify the modern equivalent.
5. **Assess static assets** — Identify CSS, JS, images, fonts that need to be relocated.
6. **Check for removed/obsolete APIs** — Serialization, security, hosting APIs that no longer exist.
7. **Create a written plan** — Document every file that needs to change and what the change is, before writing any code.

**Eval criteria:**
- [ ] Every source file is accounted for in the plan
- [ ] Every breaking API has a documented replacement
- [ ] No surprises should remain after this phase

### Phase 1: Project File & Configuration
**Goal:** Make the project restorable on the target platform.

1. Replace legacy project file format with modern SDK-style project file.
2. Replace package manager config with inline package references — only add packages that are not built into the new platform.
3. Migrate application configuration files to the modern config format.
4. Keep the same connection strings, app settings keys, and default values.

**Eval criteria:**
- [ ] `dotnet restore` (or platform equivalent) succeeds with 0 errors
- [ ] Package count is dramatically reduced (most legacy packages are now in-box)
- [ ] All original configuration values are preserved in the new format

### Phase 2: Data Layer
**Goal:** Migrate ORM/database code to the modern equivalent.

1. Update the DbContext to use the modern ORM's base class and constructor pattern.
2. Migrate fluent API configuration — translate each legacy API call to its modern equivalent.
3. Migrate raw SQL calls to the modern raw SQL API.
4. Migrate database initializers/seeders to the modern seeding pattern.
5. Preserve the exact same table names, column constraints, and relationships.

**Eval criteria:**
- [ ] DbContext compiles with 0 errors
- [ ] Entity configurations produce the same schema
- [ ] Raw SQL queries use the modern API without changing the SQL itself

### Phase 3: Service Layer
**Goal:** Migrate service classes with minimal changes.

1. Update ORM-specific using directives.
2. Replace any framework-coupled state management calls.
3. Business logic should require ZERO changes — if you're changing business logic, you've gone too far.

**Eval criteria:**
- [ ] Service classes compile with only using-directive changes
- [ ] No business logic was modified

### Phase 4: Controllers & API Layer
**Goal:** Migrate controllers from legacy web framework to modern web framework.

1. Replace controller base classes with their modern equivalents.
2. Replace return types and action results (e.g., legacy status code wrappers → modern helper methods).
3. Replace model binding syntax differences.
4. Replace any server-intrinsic calls (e.g., `Server.MapPath`) with modern dependency-injected equivalents (e.g., `IWebHostEnvironment`).
5. Replace legacy API controller base classes and return types.
6. For serialization changes, update call sites to use the new serialization API.

**Eval criteria:**
- [ ] All controllers compile with 0 errors
- [ ] Route attributes and URL patterns are identical to the original
- [ ] HTTP verbs and action names are preserved

### Phase 5: Views & Static Assets
**Goal:** Migrate views and reorganize static files.

1. Move static assets (CSS, JS, images, fonts) to the modern static file location.
2. Replace bundling/optimization helpers with direct file references.
3. Replace any server-side session or context access with modern equivalents.
4. Add a view imports file for shared namespaces and tag helpers.
5. Keep all existing view markup, HTML helpers, and model bindings intact.

**Eval criteria:**
- [ ] All views compile with 0 errors (at most warnings for deprecated helpers)
- [ ] Static assets are accessible via the modern static file middleware
- [ ] View layout renders the same visual structure

### Phase 6: Application Entry Point
**Goal:** Replace the legacy application lifecycle with the modern hosting model.

1. Create a modern entry point that configures the application.
2. Replace the legacy DI container with the built-in DI — register the same services with equivalent lifetimes (singleton, scoped, transient).
3. Configure middleware in the same order as the original pipeline.
4. Preserve the same default route.
5. Set mock/test-friendly defaults so the app can run without external dependencies (e.g., database).

**Eval criteria:**
- [ ] Application starts without exceptions
- [ ] Default route serves the expected page
- [ ] DI registrations match original lifetimes

### Phase 7: Cleanup
**Goal:** Remove obsolete files that are no longer needed.

1. Delete legacy entry point files (e.g., `Global.asax`).
2. Delete legacy configuration files (e.g., `Web.config` and transforms).
3. Delete legacy startup/routing/filter/bundle config classes.
4. Delete legacy DI module classes.
5. Delete legacy assembly info files.
6. Delete legacy package manager files.

**Eval criteria:**
- [ ] No legacy-only files remain
- [ ] Build still succeeds after deletion
- [ ] No orphaned files that aren't referenced by anything

### Phase 8: Final Verification
**Goal:** Confirm the migration is complete and correct.

1. Clean build from scratch — delete all intermediate/output directories first.
2. Run the application.
3. Verify every page/endpoint returns the expected HTTP status code.
4. Verify API endpoints return valid responses.
5. Spot-check that UI renders correctly (if applicable).

**Eval criteria:**
- [ ] `dotnet clean && dotnet build` succeeds with 0 errors
- [ ] Application starts and listens on expected port
- [ ] Root URL returns HTTP 200
- [ ] All API endpoints return HTTP 200
- [ ] No runtime exceptions in the console output

---

## Anti-Patterns to Avoid

1. **Big-bang rewrites** — Don't rewrite files from scratch. Edit the existing code in place.
2. **Gratuitous modernization** — Don't convert callback patterns to async/await, don't switch logging frameworks, don't adopt new architectural patterns. Migration ≠ modernization.
3. **Changing what works** — If a file doesn't need changes to compile and run on the new platform, don't touch it.
4. **Regenerating migrations** — Keep existing database migrations unless the schema is changing. Migration files are historical records.
5. **Adding infrastructure** — Don't add design-time factories, health checks, OpenAPI, or Docker support unless explicitly asked. Stay on task.
6. **Batch-and-pray** — Don't make 20 file changes before compiling. Build early and often.
7. **Renaming or moving code files** — Keep files in their original locations unless the platform requires relocation (e.g., static assets to `wwwroot`).

---

## Checklist Template

Copy this for each migration:

```
Project: _______________
Source framework: _______________
Target framework: _______________

[ ] Phase 0: Analysis complete, plan documented
[ ] Phase 1: Project file & config migrated, restore succeeds
[ ] Phase 2: Data layer compiles
[ ] Phase 3: Service layer compiles
[ ] Phase 4: Controllers compile
[ ] Phase 5: Views compile, static assets relocated
[ ] Phase 6: Entry point created, app starts
[ ] Phase 7: Legacy files removed
[ ] Phase 8: Clean build, app runs, all endpoints verified
```
