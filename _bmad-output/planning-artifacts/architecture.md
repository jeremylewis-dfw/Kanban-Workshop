---
stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8]
inputDocuments: ['prd.md']
workflowType: 'architecture'
project_name: 'Kanban-Workshop'
user_name: 'EWN Workshop'
date: '2026-05-14'
lastStep: 8
status: 'complete'
completedAt: '2026-05-14'
---

# Architecture Decision Document

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

## Project Context Analysis

### Requirements Overview

**Functional Requirements:** 19 FRs across Project Management (FR1–4), Task Management (FR5–8), Board Interaction (FR9–11), Navigation (FR12–13), Data Persistence (FR14–15), and Data Integrity (FR16–19). Two core entities: `Project` and `Task`. Relationships are simple — one project has many tasks; each task belongs to one of three fixed columns.

**Non-Functional Requirements:** CRUD operations must complete within 1 second; board load within 2 seconds for up to 100 tasks; drag-and-drop must provide immediate visual feedback. Basic semantic HTML accessibility required (keyboard navigation, labeled inputs).

**Scale & Complexity:**

- Primary domain: Full-stack web (Blazor Server + .NET REST API)
- Complexity level: Low
- Estimated architectural components: 5 (Domain, Data/Repository, Service, API, UI)

### Technical Constraints & Dependencies

- Blazor Server rendering model (SignalR) — state lives on server, no client-side serialization concerns
- .NET 8+ / EF Core / SQLite — single-file database, no external DB server required
- No authentication, no multi-tenancy — single-user local application
- Drag-and-drop requires JS interop or a Blazor-compatible component library

### Cross-Cutting Concerns Identified

- **Drag-and-drop interaction:** Requires JS interop or third-party Blazor component — affects UI layer and potentially project dependencies
- **Confirmation dialogs:** Reusable modal/dialog pattern needed across project and task deletion (FR17, FR18)
- **Empty state handling:** Reusable pattern for boards with no tasks (FR19)
- **Clean layer separation:** All business logic must reside in the service layer — controllers and components are thin orchestrators only

## Starter Template Evaluation

### Primary Technology Domain

.NET full-stack web application — Blazor Web App (`dotnet new blazorweb`) with internal folder-based layer separation, SQLite via EF Core.

### Selected Starter: Blazor Web App (Single Project)

**Rationale:** Single deployable unit, minimal scaffolding overhead. Layer separation enforced by folder structure and coding conventions rather than project boundaries. Appropriate for a workshop codebase where simplicity of setup matters.

**Initialization Command:**

```bash
dotnet new blazorweb -n KanbanWorkshop -o KanbanWorkshop
```

**Folder Structure (Layer Separation by Convention):**

```text
KanbanWorkshop/
├── Domain/          # Entities: Project, Task, ColumnStatus enum
├── Data/            # EF Core DbContext, migrations, repository implementations
├── Services/        # Business logic — IProjectService, ITaskService + implementations
├── Api/             # Minimal API or controller endpoints
├── Components/      # Blazor UI components and pages
└── Tests/           # NUnit test project (separate project, references KanbanWorkshop)
```

**Architectural Decisions Established:**

- **Language & Runtime:** C# / .NET 10 LTS
- **Rendering:** Blazor Web App (server-side rendering)
- **Layer discipline:** Convention-based — no cross-layer imports by rule
- **Testing:** Separate `nunit` test project references the main project
- **Build tooling:** Single `dotnet run` entry point

**Note:** Project initialization is the first implementation story.

## Core Architectural Decisions

### Decision Priority Analysis

**Critical Decisions (Block Implementation):**

- Repository pattern for data access — required for testable service layer
- Soft delete strategy — affects all entity definitions and query behavior
- Controller-based API — required for unit testing workshop surface area

**Important Decisions (Shape Architecture):**

- Drag-and-drop library selection
- Error handling approach

**Deferred Decisions (Post-MVP):**

- CI/CD pipeline
- Deployment strategy

### Data Architecture

- **ORM:** Entity Framework Core (Code-First) with SQLite
- **Pattern:** Repository pattern — each aggregate root (`Project`, `Task`) has a corresponding repository interface and implementation
- **Soft Deletes:** All entities include `CreatedOn` (DateTime) and `DeletedOn` (DateTime?) properties. EF Core Global Query Filters apply `WHERE DeletedOn IS NULL` at the `DbContext` level — services and repositories never manually filter deleted records
- **Delete behavior:** Repository `Delete()` methods set `DeletedOn = DateTime.UtcNow` rather than calling `dbContext.Remove()` — confirmation dialogs in UI trigger soft delete
- **Migrations:** EF Core migrations, applied on startup in development

### Authentication & Security

None required. Single-user local application — no login, no roles, no authorization middleware.

### API & Communication Patterns

- **Style:** Controller-based Web API (not Minimal API) — provides more testable surface area for workshop exercises
- **Error handling:** Try/catch in service layer; services return result objects or throw typed exceptions; controllers translate to appropriate HTTP status codes
- **Documentation:** Swagger/OpenAPI enabled in development (included by default in `dotnet new blazorweb`)

### Frontend Architecture

- **Rendering:** Blazor Web App server-side rendering
- **State management:** Component-scoped state via injected services — no Flux/Redux pattern needed
- **Drag-and-drop:** `Blazor.DragDrop` NuGet package — pure C# integration, no JS interop required
- **Reusable patterns:** Shared confirmation dialog component, shared empty state component

### Infrastructure & Deployment

- **Environment:** Local development only (workshop tool)
- **Logging:** Built-in `ILogger<T>` via .NET dependency injection
- **No CI/CD, no cloud deployment** required for this scope

### Decision Impact Analysis

**Implementation Sequence:**

1. Project scaffold (`dotnet new blazorweb`)
2. Domain entities with `CreatedOn`/`DeletedOn`
3. EF Core DbContext with global query filters + SQLite
4. Repository interfaces and implementations
5. Service layer (`IProjectService`, `ITaskService`)
6. API controllers
7. Blazor components and pages
8. `Blazor.DragDrop` integration

**Cross-Component Dependencies:**

- Global query filter in `DbContext` is the single source of truth for soft-delete behavior — repositories must not duplicate this filter
- Service layer depends on repository interfaces only — never on `DbContext` directly
- Blazor components depend on service interfaces only — never on repositories or `DbContext`

## Implementation Patterns & Consistency Rules

### Naming Patterns

**C# Code Conventions:**

- Classes, interfaces, methods, properties: `PascalCase`
- Parameters, local variables: `camelCase`
- Interfaces prefixed with `I`: `IProjectRepository`, `ITaskService`
- Private fields: `_camelCase` (e.g., `_projectRepository`)

**Database Naming (EF Core):**

- Table names: plural PascalCase — `Projects`, `Tasks` (EF Core default via `DbSet<T>` property name)
- Column names: match C# property names exactly (PascalCase) — EF Core default
- Foreign keys: `{EntityName}Id` — e.g., `ProjectId`

**API Endpoint Naming:**

- Plural, lowercase, kebab-case: `/api/projects`, `/api/tasks`
- Resource-scoped routes: `/api/projects/{id}/tasks`
- Route parameters: `{id}` (not `:id`)

**JSON Serialization:**

- camelCase field names in all API responses (configured via `JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase`)

### Structure Patterns

**Folder Organization:**

```text
Domain/
  Entities/         # Project.cs, TaskItem.cs (Task conflicts with System.Threading.Tasks)
  Enums/            # ColumnStatus.cs
Data/
  AppDbContext.cs
  Repositories/
    Interfaces/     # IProjectRepository.cs, ITaskRepository.cs
    ProjectRepository.cs
    TaskRepository.cs
Services/
  Interfaces/       # IProjectService.cs, ITaskService.cs
  ProjectService.cs
  TaskService.cs
Api/
  Controllers/      # ProjectsController.cs, TasksController.cs
Components/
  Pages/            # Board.razor, ProjectList.razor
  Shared/           # ConfirmDialog.razor, EmptyState.razor
```

**Test Project:**

- Single `KanbanWorkshop.Tests` NUnit project alongside main project
- Mirror source structure: `Services/ProjectServiceTests.cs`, `Repositories/ProjectRepositoryTests.cs`
- Test class naming: `{ClassName}Tests`
- Test method naming: `{MethodName}_When{Condition}_Should{ExpectedResult}`

### Format Patterns

**API Response Format:**

- Success: return the resource directly (no wrapper) — `200 OK` with `ProjectDto`
- Not found: `404 NotFound()` with no body
- Validation error: `400 BadRequest(new { message = "..." })`
- Server error: `500` with no sensitive detail exposed

**DTOs vs Entities:**

- Controllers accept and return DTOs — never expose EF entities directly
- DTO naming: `ProjectDto`, `CreateProjectRequest`, `UpdateTaskRequest`

**Soft Deletes:**

- Soft-deleted records are never returned by the API — enforced by EF Core global query filter
- No `isDeleted` field in DTOs — callers never see deleted records

### Process Patterns

**Service Layer:**

- Services throw typed exceptions for business rule violations: `NotFoundException`, `ValidationException`
- Services never return `null` — throw or return empty collections

**Blazor Components:**

- Loading state: `bool _isLoading` field, set true before async call, false in `finally`
- Error state: `string? _errorMessage` field, displayed conditionally
- All service calls in components wrapped in try/catch

**Dependency Injection Registration:**

- All interfaces registered in `Program.cs` as `Scoped` (matches EF Core DbContext lifetime)

### Enforcement Guidelines

**All agents MUST:**

- Use `TaskItem` (not `Task`) for the task entity to avoid `System.Threading.Tasks.Task` collision
- Never inject `AppDbContext` directly into services — always use repository interfaces
- Never add `WHERE DeletedOn IS NULL` filters manually — the global query filter handles this
- Always use DTOs at the API boundary — never serialize EF entities

## Project Structure & Boundaries

### Complete Project Directory Structure

```text
KanbanWorkshop/                          # dotnet new blazorweb
├── KanbanWorkshop.csproj
├── Program.cs                           # DI registration, EF Core setup, app config
├── appsettings.json
├── appsettings.Development.json
│
├── Domain/
│   ├── Entities/
│   │   ├── Project.cs                   # FR1-4: Id, Name, CreatedOn, DeletedOn
│   │   └── TaskItem.cs                  # FR5-8: Id, Title, Description, ColumnStatus, ProjectId, CreatedOn, DeletedOn
│   └── Enums/
│       └── ColumnStatus.cs              # Backlog, InProgress, Done
│
├── Data/
│   ├── AppDbContext.cs                  # DbSets, global query filters (soft delete)
│   ├── Migrations/                      # EF Core generated migrations
│   └── Repositories/
│       ├── Interfaces/
│       │   ├── IProjectRepository.cs    # FR1-4, FR17
│       │   └── ITaskRepository.cs       # FR5-8, FR18
│       ├── ProjectRepository.cs
│       └── TaskRepository.cs
│
├── Services/
│   ├── Interfaces/
│   │   ├── IProjectService.cs
│   │   └── ITaskService.cs
│   ├── ProjectService.cs                # FR1-4, FR16-17, FR19
│   ├── TaskService.cs                   # FR5-8, FR16, FR18
│   └── Exceptions/
│       ├── NotFoundException.cs
│       └── ValidationException.cs
│
├── Api/
│   ├── Controllers/
│   │   ├── ProjectsController.cs        # FR1-4 endpoints
│   │   └── TasksController.cs           # FR5-8, FR10 endpoints
│   └── DTOs/
│       ├── ProjectDto.cs
│       ├── CreateProjectRequest.cs
│       ├── UpdateProjectRequest.cs
│       ├── TaskItemDto.cs
│       ├── CreateTaskRequest.cs
│       ├── UpdateTaskRequest.cs
│       └── MoveTaskRequest.cs           # FR10: column assignment
│
├── Components/
│   ├── App.razor
│   ├── Routes.razor
│   ├── Layout/
│   │   ├── MainLayout.razor
│   │   └── NavMenu.razor                # FR12-13: navigation
│   ├── Pages/
│   │   ├── ProjectList.razor            # FR4, FR12: project list view
│   │   └── Board.razor                  # FR9-11: kanban board view
│   └── Shared/
│       ├── ConfirmDialog.razor          # FR17-18: confirmation on delete
│       ├── EmptyState.razor             # FR19: empty board/list states
│       ├── ProjectCard.razor            # FR2-3: project item in list
│       └── TaskCard.razor               # FR6-7, FR10: task card with drag handle
│
└── wwwroot/
    └── app.css

KanbanWorkshop.Tests/                    # NUnit test project
├── KanbanWorkshop.Tests.csproj          # References KanbanWorkshop + Moq + NUnit
├── Services/
│   ├── ProjectServiceTests.cs
│   └── TaskItemServiceTests.cs
├── Repositories/
│   ├── ProjectRepositoryTests.cs
│   └── TaskRepositoryTests.cs
├── Api/
│   ├── ProjectsControllerTests.cs
│   └── TasksControllerTests.cs
└── TestHelpers/
    └── DbContextFactory.cs              # In-memory SQLite for integration tests
```

### Architectural Boundaries

**API Boundaries:**

- `Api/Controllers/` → accepts HTTP, validates input, calls services, returns DTOs
- Controllers never touch `AppDbContext` or repositories directly
- All routes prefixed `/api/`

**Service Boundaries:**

- `Services/` → owns all business logic and validation (FR16, soft delete enforcement)
- Services depend on repository interfaces only (`IProjectRepository`, `ITaskRepository`)
- Services throw `NotFoundException` / `ValidationException` — never return null

**Data Boundaries:**

- `Data/AppDbContext.cs` → owns the global query filter; only place `DeletedOn IS NULL` is enforced
- Repositories depend on `AppDbContext` directly — no other layer touches it
- EF migrations are the sole schema management mechanism

**Component Boundaries:**

- Blazor components call service interfaces only — never repositories or DbContext
- `Components/Shared/` contains reusable stateless UI primitives
- `Components/Pages/` contains page-level components with state management

### Requirements to Structure Mapping

| FR Group | Files |
| --- | --- |
| Project Management (FR1–4) | `ProjectService`, `ProjectsController`, `ProjectList.razor`, `ProjectCard.razor` |
| Task Management (FR5–8) | `TaskItemService`, `TasksController`, `TaskCard.razor` |
| Board Interaction (FR9–11) | `Board.razor`, `TaskCard.razor`, `Blazor.DragDrop` integration |
| Navigation (FR12–13) | `NavMenu.razor`, `Routes.razor`, `MainLayout.razor` |
| Data Persistence (FR14–15) | `AppDbContext.cs`, `Migrations/`, all repositories |
| Data Integrity (FR16–19) | `Services/Exceptions/`, `ConfirmDialog.razor`, `EmptyState.razor` |

### Data Flow

```text
Blazor Component → IProjectService / ITaskService
                → IProjectRepository / ITaskRepository
                → AppDbContext (with global query filter)
                → SQLite (kanban.db)
```

## Architecture Validation Results

### Coherence Validation

**Decision Compatibility:** All technology choices are fully compatible. .NET 10 / Blazor Web App / EF Core / SQLite is a well-supported stack with no version conflicts. Blazor.DragDrop is compatible with Blazor Server rendering. Controller-based API with DTOs aligns cleanly with the repository pattern and service layer.

**Pattern Consistency:** Naming conventions (PascalCase entities, camelCase JSON, plural REST routes) are consistent across all layers. The soft delete pattern (global query filter in `AppDbContext`) is the single source of truth — no duplication risk.

**Structure Alignment:** Folder structure directly mirrors the layer separation required by the PRD Success Criteria. Boundaries are enforced by convention with clear documented rules.

### Requirements Coverage Validation

**Functional Requirements:** All 19 FRs mapped to specific files in the project structure. No FR is left without an owning component.

**Non-Functional Requirements:**

- Performance (< 1s CRUD, < 2s board load): SQLite local access with simple queries — easily achievable
- Drag-and-drop immediate feedback: Blazor.DragDrop handles client-side interaction
- Accessibility: Blazor renders semantic HTML; `ConfirmDialog.razor` and form components support keyboard navigation

### Implementation Readiness Validation

**Decision Completeness:** All critical decisions documented. `TaskItem` naming conflict pre-empted. Soft delete behavior, DI lifetime, DTO boundaries, and test structure all specified.

**Structure Completeness:** Complete file tree with FR traceability. All shared components identified. Test project structure mirrors source.

**Pattern Completeness:** All potential agent conflict points addressed — naming, structure, soft delete enforcement, null handling, loading state, error state.

### Gap Analysis Results

**Minor Gaps (non-blocking):**

- SQLite connection string and `kanban.db` file location not yet specified — needs an entry in `appsettings.json`
- `Blazor.DragDrop` and `Moq` package versions should be verified at implementation time (latest stable)
- `DbContextFactory.cs` in test helpers needs an in-memory SQLite configuration pattern documented

No critical gaps identified.

### Architecture Completeness Checklist

#### Requirements Analysis

- [x] Project context thoroughly analyzed
- [x] Scale and complexity assessed
- [x] Technical constraints identified
- [x] Cross-cutting concerns mapped

#### Architectural Decisions

- [x] Critical decisions documented with versions
- [x] Technology stack fully specified
- [x] Integration patterns defined
- [x] Performance considerations addressed

#### Implementation Patterns

- [x] Naming conventions established
- [x] Structure patterns defined
- [x] Communication patterns specified
- [x] Process patterns documented

#### Project Structure

- [x] Complete directory structure defined
- [x] Component boundaries established
- [x] Integration points mapped
- [x] Requirements to structure mapping complete

### Architecture Readiness Assessment

**Overall Status:** READY FOR IMPLEMENTATION

**Confidence Level:** High

**Key Strengths:**

- Clean, teachable layer separation with explicit FR traceability
- Soft delete pattern centralized in one place — no scattered filter logic
- `TaskItem` naming pre-empts the most common .NET Kanban naming collision
- Test project structure mirrors source — straightforward for workshop participants to navigate

**Areas for Future Enhancement:**

- Add `appsettings.json` connection string pattern in first implementation story
- Verify `Blazor.DragDrop` and `Moq` versions at scaffold time

### Implementation Handoff

**AI Agent Guidelines:**

- Follow all architectural decisions exactly as documented
- Use `TaskItem` — never `Task` — for the task entity
- Never bypass the global query filter — all soft delete logic flows through `AppDbContext`
- Controllers return DTOs only; services throw typed exceptions only
- Refer to this document for all architectural questions

**First Implementation Priority:**

```bash
dotnet new blazorweb -n KanbanWorkshop -o KanbanWorkshop
```
