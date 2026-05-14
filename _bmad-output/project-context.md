---
project_name: 'Kanban-Workshop'
user_name: 'EWN Workshop'
date: '2026-05-14'
sections_completed: ['technology_stack', 'language_rules', 'framework_rules', 'soft_delete_rules', 'testing_rules', 'component_rules', 'anti_patterns']
status: 'complete'
rule_count: 28
optimized_for_llm: true
---

# Project Context for AI Agents

_This file contains critical rules and patterns that AI agents must follow when implementing code in this project. Focus on unobvious details that agents might otherwise miss._

---

## Technology Stack & Versions

- **Runtime:** .NET 10 LTS / C#
- **UI Framework:** Blazor Web App (server-side rendering, SignalR)
- **ORM:** Entity Framework Core (Code-First) + SQLite
- **Drag-and-Drop:** Blazor.DragDrop (NuGet — verify latest stable at scaffold time)
- **Testing:** NUnit + Moq (`KanbanWorkshop.Tests` project)

---

## Critical Implementation Rules

### Language & Naming Rules

- **Entity naming:** Use `TaskItem`, NEVER `Task` — `Task` conflicts with `System.Threading.Tasks.Task`
- **Classes/interfaces/methods/properties:** `PascalCase`
- **Parameters/local variables:** `camelCase`
- **Private fields:** `_camelCase` (e.g., `_projectRepository`)
- **Interfaces:** always prefixed with `I` (e.g., `IProjectRepository`)
- **DTOs:** `{Entity}Dto`, `Create{Entity}Request`, `Update{Entity}Request`, `Move{Entity}Request`

### Framework & Architecture Rules

- **Never inject `AppDbContext` into services** — inject repository interfaces only
- **Never add `WHERE DeletedOn IS NULL` manually** — EF Core global query filter in `AppDbContext` handles this automatically for all queries
- **Repository `Delete()` must set `DeletedOn = DateTime.UtcNow`** — never call `dbContext.Remove()`
- **Controllers never reference repositories or `AppDbContext`** — only service interfaces
- **Blazor components never reference repositories or `AppDbContext`** — only service interfaces
- **Controllers never serialize EF entities** — always map to DTOs before returning
- **Services never return `null`** — throw `NotFoundException` or return empty collections
- **All DI registrations use `Scoped` lifetime** (matches `DbContext` lifetime)
- **JSON serialization uses camelCase** — configured via `JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase` in `Program.cs`

### Soft Delete Rules

- All entities (`Project`, `TaskItem`) have `CreatedOn` (DateTime) and `DeletedOn` (DateTime?) properties
- `AppDbContext` registers a global query filter for each entity: `.HasQueryFilter(e => e.DeletedOn == null)`
- The UI "delete" action calls the service, which calls `repository.Delete()`, which sets `DeletedOn` — never hard deletes
- Soft-deleted records never appear in API responses — enforced by the global filter, not by callers

### Testing Rules

- **Test project:** `KanbanWorkshop.Tests` — mirrors source folder structure
- **Test class naming:** `{ClassName}Tests` (e.g., `ProjectServiceTests`)
- **Test method naming:** `{MethodName}_When{Condition}_Should{ExpectedResult}`
- **Service tests:** mock repository interfaces with Moq — never mock `AppDbContext` directly
- **Repository/integration tests:** use `DbContextFactory.cs` with in-memory SQLite provider
- **One `[Test]` or `[TestCase]` per behavior** — avoid multi-assertion tests that hide failures

### Blazor Component Rules

- Loading state: `private bool _isLoading;` — set `true` before async call, `false` in `finally`
- Error state: `private string? _errorMessage;` — display conditionally in markup
- All service calls in components wrapped in `try/catch`
- Empty state handled via `<EmptyState />` shared component (never inline ad-hoc messages)
- Destructive actions (delete project/task) always invoke `<ConfirmDialog />` first

### Critical Anti-Patterns

- **Never use `Task` as an entity name** — always `TaskItem`
- **Never filter soft-deleted records in service or repository code** — the global query filter makes this invisible; adding a manual filter creates a bug when the filter changes
- **Never expose EF navigation properties in DTOs** — map scalar fields only
- **Never register services as `Singleton`** — EF `DbContext` is `Scoped`; a `Singleton` service holding it causes lifetime conflicts

---

## Usage Guidelines

**For AI Agents:**

- Read this file before implementing any code
- Follow ALL rules exactly as documented
- When in doubt, prefer the more restrictive option
- Update this file if new patterns emerge during implementation

**For Humans:**

- Keep this file lean and focused on agent needs
- Update when technology stack changes
- Remove rules that become obvious over time

Last Updated: 2026-05-14
