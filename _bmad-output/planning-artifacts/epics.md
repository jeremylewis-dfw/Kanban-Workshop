---
stepsCompleted: ['step-01-validate-prerequisites', 'step-02-design-epics', 'step-03-create-stories', 'step-04-final-validation']
inputDocuments: ['prd.md', 'architecture.md']
---

# Kanban-Workshop - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for Kanban-Workshop, decomposing the requirements from the PRD and Architecture into implementable stories.

## Requirements Inventory

### Functional Requirements

FR1: User can create a new project with a name
FR2: User can rename an existing project
FR3: User can delete a project
FR4: User can view a list of all projects
FR5: User can create a task within a project with a title and description
FR6: User can edit a task's title and description
FR7: User can delete a task
FR8: User can assign a task to a column at creation time
FR9: User can view a project's tasks organized into Backlog, In Progress, and Done columns
FR10: User can move a task between columns via drag-and-drop
FR11: Each project board displays exactly three fixed columns: Backlog, In Progress, Done
FR12: User can navigate to a project board from the project list
FR13: User can return to the project list from a board view
FR14: All project and task data persists between application sessions
FR15: Board state (tasks and their column assignments) is restored accurately when a project is reopened
FR16: System prevents creation of a task with an empty title
FR17: System requires confirmation before deleting a project
FR18: System requires confirmation before deleting a task
FR19: System displays an appropriate empty state when a project has no tasks

### NonFunctional Requirements

NFR1: CRUD operations (create, edit, delete project/task) complete and reflect in the UI within 1 second under normal localhost/LAN conditions
NFR2: Board load (project open with up to 100 tasks) completes within 2 seconds
NFR3: Drag-and-drop card movement provides immediate visual feedback with no perceptible lag
NFR4: All interactive elements are reachable via keyboard
NFR5: All form inputs have associated labels
NFR6: Page structure uses semantic HTML elements (headings, lists, buttons, forms)

### Additional Requirements

- Project scaffolded via `dotnet new blazorweb -n KanbanWorkshop -o KanbanWorkshop` (.NET 10 LTS)
- Folder-based layer separation: Domain/, Data/, Services/, Api/, Components/ within single project
- All entities (Project, TaskItem) must include CreatedOn (DateTime) and DeletedOn (DateTime?) properties
- EF Core global query filter applied in AppDbContext for soft delete: `.HasQueryFilter(e => e.DeletedOn == null)`
- Repository pattern: IProjectRepository and ITaskRepository interfaces with concrete implementations
- SQLite connection string configured in appsettings.json; database file: kanban.db
- Blazor.DragDrop NuGet package for drag-and-drop (verify latest stable version at scaffold time)
- Controller-based Web API (not Minimal API) — ProjectsController, TasksController
- DTOs at API boundary — never serialize EF entities directly
- Separate NUnit + Moq test project: KanbanWorkshop.Tests (mirrors source folder structure)
- DbContextFactory.cs in TestHelpers/ for in-memory SQLite integration tests
- All DI registrations use Scoped lifetime
- JSON serialization uses camelCase via JsonNamingPolicy.CamelCase in Program.cs

### UX Design Requirements

No UX Design document provided. UI implementation follows architecture patterns and Blazor defaults.

### FR Coverage Map

| FR | Epic | Description |
|---|---|---|
| FR1 | Epic 1 | Create project |
| FR2 | Epic 1 | Rename project |
| FR3 | Epic 1 | Delete project |
| FR4 | Epic 1 | View project list |
| FR5 | Epic 2 | Create task with title and description |
| FR6 | Epic 2 | Edit task title and description |
| FR7 | Epic 2 | Delete task |
| FR8 | Epic 2 | Assign task to column at creation |
| FR9 | Epic 2 | View tasks organized by column |
| FR10 | Epic 3 | Drag-and-drop task movement |
| FR11 | Epic 2 | Fixed three columns |
| FR12 | Epic 1 | Navigate to project board |
| FR13 | Epic 1 | Return to project list |
| FR14 | Epic 2 | Data persists between sessions |
| FR15 | Epic 2 | Board state restored on reopen |
| FR16 | Epic 2 | Prevent empty task title |
| FR17 | Epic 1 | Confirm before deleting project |
| FR18 | Epic 2 | Confirm before deleting task |
| FR19 | Epic 2 | Empty state when no tasks exist |

## Epic List

### Epic 1: Project Management

Users can create, view, rename, and delete projects, and navigate between the project list and individual board views. This establishes the full project lifecycle and the navigation shell the task board lives within.

**FRs covered:** FR1, FR2, FR3, FR4, FR12, FR13, FR17

### Epic 2: Task Board

Users can manage tasks on a Kanban board — creating tasks with title and description, editing, deleting, assigning to columns at creation time, and viewing the board organized into Backlog, In Progress, and Done. Empty states display when no tasks exist. Data persists between sessions and board state is fully restored on reopen.

**FRs covered:** FR5, FR6, FR7, FR8, FR9, FR11, FR14, FR15, FR16, FR18, FR19

### Epic 3: Drag-and-Drop Card Movement

Users can interactively move task cards between columns via drag-and-drop with immediate visual feedback. This builds on the static board from Epic 2 by wiring in the Blazor.DragDrop package and a dedicated MoveTask API endpoint.

**FRs covered:** FR10

---

## Epic 1 Stories: Project Management

Users can create, view, rename, and delete projects, and navigate between the project list and individual board views.

### Story 1.1: Project Foundation — Data Layer and Empty Project List

As a user,
I want to open the app and see a project list,
So that I have a starting point to manage my projects.

**Acceptance Criteria:**

**Given** the app is scaffolded with the correct folder structure (Domain/, Data/, Services/, Api/, Components/),
**When** I navigate to the home page,
**Then** the Projects list page renders with an empty state message and a visible "Create Project" affordance.

**Given** no projects exist in the database,
**When** `GET /api/projects` is called,
**Then** it returns HTTP 200 with an empty array.

**Given** a Project entity has `DeletedOn` set to a non-null value,
**When** `GET /api/projects` is called,
**Then** the soft-deleted project does not appear in the response (enforced by EF Core global query filter, not a manual WHERE clause).

**Given** `ProjectService.GetAllAsync()` is called,
**When** the repository returns an empty collection,
**Then** the service returns an empty list (never null).

### Story 1.2: Create a New Project

As a user,
I want to create a new project with a name,
So that I can start tracking tasks for a specific area of work.

**Acceptance Criteria:**

**Given** I provide a non-empty project name,
**When** `POST /api/projects` is called with a `CreateProjectRequest`,
**Then** it returns HTTP 201 with a `ProjectDto` containing the new project's Id, Name, and CreatedOn.

**Given** the project is created,
**When** `GET /api/projects` is called,
**Then** the new project appears in the response list.

**Given** an empty or whitespace-only name is submitted,
**When** `POST /api/projects` is called,
**Then** it returns HTTP 400 Bad Request with a validation error message.

**Given** `ProjectService.CreateAsync()` is called with a valid name,
**When** the repository saves the project,
**Then** `CreatedOn` is set to a UTC datetime and `DeletedOn` is null.

**Given** `ProjectService.CreateAsync()` is called with an empty name,
**When** the service validates the input,
**Then** a `ValidationException` is thrown before calling the repository.

### Story 1.3: Rename an Existing Project

As a user,
I want to rename an existing project,
So that I can correct or update a project's name as my work evolves.

**Acceptance Criteria:**

**Given** a project exists,
**When** `PUT /api/projects/{id}` is called with an `UpdateProjectRequest` containing a new non-empty name,
**Then** it returns HTTP 200 with the updated `ProjectDto` reflecting the new name.

**Given** a project id that does not exist,
**When** `PUT /api/projects/{id}` is called,
**Then** it returns HTTP 404 Not Found.

**Given** an empty or whitespace name is submitted,
**When** `PUT /api/projects/{id}` is called,
**Then** it returns HTTP 400 Bad Request.

**Given** `ProjectService.UpdateAsync()` is called with a valid id and new name,
**When** the project is found by the repository,
**Then** the name is updated and the updated project is returned.

**Given** `ProjectService.UpdateAsync()` is called with a non-existent id,
**When** the repository finds no matching project,
**Then** a `NotFoundException` is thrown.

### Story 1.4: Delete a Project with Confirmation

As a user,
I want to delete a project after confirming the action,
So that I can remove projects I no longer need without accidental data loss.

**Acceptance Criteria:**

**Given** a project exists,
**When** the user initiates deletion and confirms via `<ConfirmDialog />`,
**Then** `DELETE /api/projects/{id}` is called and the project no longer appears in the project list.

**Given** `DELETE /api/projects/{id}` is called for an existing project,
**When** the repository processes the delete,
**Then** it returns HTTP 204 No Content and `DeletedOn` is set to a UTC datetime (soft delete — `dbContext.Remove()` is never called).

**Given** a project is soft-deleted,
**When** `GET /api/projects` is called,
**Then** the deleted project does not appear in the response.

**Given** a project id that does not exist,
**When** `DELETE /api/projects/{id}` is called,
**Then** it returns HTTP 404 Not Found.

**Given** `ProjectService.DeleteAsync()` is called,
**When** the repository sets `DeletedOn`,
**Then** `DeletedOn` equals `DateTime.UtcNow` (within a reasonable tolerance) and the entity is not removed from the database.

### Story 1.5: Navigate Between Project List and Board

As a user,
I want to click a project to open its board and return to the project list,
So that I can switch between managing projects and working within a specific project.

**Acceptance Criteria:**

**Given** the project list displays one or more projects,
**When** I click on a project name,
**Then** I am navigated to `/board/{projectId}` and the board page renders for that project.

**Given** I am on a board page,
**When** I click the "Back to Projects" navigation element,
**Then** I am returned to the project list at `/`.

**Given** I navigate to `/board/{id}` where `{id}` does not match any project,
**When** the board component loads,
**Then** an appropriate error or not-found message is displayed.

---

## Epic 2 Stories: Task Board

Users can manage tasks on a Kanban board with full CRUD, column assignment, persistence, and empty states.

### Story 2.1: Task Board Foundation — Data Layer and Static Board View

As a user,
I want to open a project and see its Kanban board with three columns,
So that I have a visual workspace for organizing my tasks.

**Acceptance Criteria:**

**Given** a project exists and I navigate to its board,
**When** the board page loads,
**Then** three columns are displayed: Backlog, In Progress, and Done — always, regardless of task count.

**Given** a project has no tasks,
**When** the board page loads,
**Then** an `<EmptyState />` message is displayed within the board.

**Given** `GET /api/projects/{projectId}/tasks` is called,
**When** no tasks exist for the project,
**Then** it returns HTTP 200 with an empty array.

**Given** `TaskService.GetByProjectAsync(projectId)` is called,
**When** the repository returns an empty collection,
**Then** the service returns an empty list (never null).

**Given** tasks exist for a project,
**When** `GET /api/projects/{projectId}/tasks` is called,
**Then** tasks are returned as `TaskItemDto` objects — EF entities are never serialized directly.

### Story 2.2: Create a Task

As a user,
I want to create a task with a title, description, and column assignment,
So that I can add work items to my board in the right starting state.

**Acceptance Criteria:**

**Given** I provide a non-empty title, an optional description, and a column selection,
**When** `POST /api/projects/{projectId}/tasks` is called with a `CreateTaskRequest`,
**Then** it returns HTTP 201 with a `TaskItemDto` containing the new task's Id, Title, Description, ColumnStatus, and CreatedOn.

**Given** an empty or whitespace title is submitted,
**When** `POST /api/projects/{projectId}/tasks` is called,
**Then** it returns HTTP 400 Bad Request.

**Given** `TaskService.CreateAsync()` is called with an empty title,
**When** the service validates the input,
**Then** a `ValidationException` is thrown before calling the repository.

**Given** `TaskService.CreateAsync()` is called with a valid request,
**When** the repository saves the task,
**Then** `CreatedOn` is set to a UTC datetime, `DeletedOn` is null, and `ColumnStatus` matches the requested column.

**Given** a task is created with no explicit column specified,
**When** the task is saved,
**Then** `ColumnStatus` defaults to `Backlog`.

### Story 2.3: Edit a Task

As a user,
I want to edit a task's title and description,
So that I can correct or update task details as work progresses.

**Acceptance Criteria:**

**Given** a task exists,
**When** `PUT /api/projects/{projectId}/tasks/{taskId}` is called with an `UpdateTaskRequest` containing a new non-empty title and/or description,
**Then** it returns HTTP 200 with the updated `TaskItemDto`.

**Given** a task id that does not exist,
**When** `PUT /api/projects/{projectId}/tasks/{taskId}` is called,
**Then** it returns HTTP 404 Not Found.

**Given** an empty or whitespace title is submitted,
**When** `PUT /api/projects/{projectId}/tasks/{taskId}` is called,
**Then** it returns HTTP 400 Bad Request.

**Given** `TaskService.UpdateAsync()` is called with a valid id and new values,
**When** the task is found by the repository,
**Then** the title and description are updated and the updated task is returned.

**Given** `TaskService.UpdateAsync()` is called with a non-existent task id,
**When** the repository finds no matching task,
**Then** a `NotFoundException` is thrown.

### Story 2.4: Delete a Task with Confirmation

As a user,
I want to delete a task after confirming the action,
So that I can remove completed or irrelevant tasks without accidental data loss.

**Acceptance Criteria:**

**Given** a task exists,
**When** the user initiates deletion and confirms via `<ConfirmDialog />`,
**Then** `DELETE /api/projects/{projectId}/tasks/{taskId}` is called and the task is removed from the board.

**Given** `DELETE /api/projects/{projectId}/tasks/{taskId}` is called for an existing task,
**When** the repository processes the delete,
**Then** it returns HTTP 204 No Content and `DeletedOn` is set to a UTC datetime (soft delete — `dbContext.Remove()` is never called).

**Given** a task is soft-deleted,
**When** `GET /api/projects/{projectId}/tasks` is called,
**Then** the deleted task does not appear in the response.

**Given** a task id that does not exist,
**When** `DELETE /api/projects/{projectId}/tasks/{taskId}` is called,
**Then** it returns HTTP 404 Not Found.

**Given** `TaskService.DeleteAsync()` is called,
**When** the repository sets `DeletedOn`,
**Then** `DeletedOn` equals `DateTime.UtcNow` (within a reasonable tolerance) and the entity is not removed from the database.

### Story 2.5: Board State Persistence and Restore

As a user,
I want my board to look exactly the same when I reopen the app,
So that I can trust the app to preserve my work between sessions.

**Acceptance Criteria:**

**Given** tasks have been created and assigned to various columns,
**When** the application is restarted and I navigate to the project board,
**Then** all tasks appear in the same columns they were in before the restart.

**Given** `GET /api/projects/{projectId}/tasks` is called after an app restart,
**When** the SQLite database contains persisted tasks,
**Then** all tasks are returned with their correct `ColumnStatus` values.

**Given** the board loads a project with up to 100 tasks,
**When** the page renders,
**Then** all tasks are displayed correctly within 2 seconds.

**Given** `TaskService.GetByProjectAsync(projectId)` is called,
**When** the repository queries tasks filtered by `projectId`,
**Then** only tasks belonging to that project are returned (tasks from other projects never appear).

---

## Epic 3 Stories: Drag-and-Drop Card Movement

Users can interactively move task cards between columns via drag-and-drop with immediate visual feedback.

### Story 3.1: Move a Task Between Columns via Drag-and-Drop

As a user,
I want to drag a task card from one column and drop it into another,
So that I can update a task's status visually without opening an edit form.

**Acceptance Criteria:**

**Given** the board displays tasks in columns,
**When** I drag a task card and drop it onto a different column,
**Then** the card moves to the target column immediately with no perceptible lag.

**Given** a task card is dropped into a new column,
**When** `PATCH /api/projects/{projectId}/tasks/{taskId}/move` is called with a `MoveTaskRequest` containing the target `ColumnStatus`,
**Then** it returns HTTP 200 with the updated `TaskItemDto` reflecting the new `ColumnStatus`.

**Given** a task is moved via the API,
**When** `GET /api/projects/{projectId}/tasks` is called,
**Then** the task is returned with the updated `ColumnStatus`.

**Given** `TaskService.MoveAsync(taskId, targetColumn)` is called with a valid task id,
**When** the repository updates the task's `ColumnStatus`,
**Then** the updated task is returned with `ColumnStatus` equal to the target column.

**Given** `TaskService.MoveAsync(taskId, targetColumn)` is called with a non-existent task id,
**When** the repository finds no matching task,
**Then** a `NotFoundException` is thrown.

**Given** a task is dragged and dropped into the column it already belongs to,
**When** the move API is called with the same `ColumnStatus` as the current value,
**Then** it returns HTTP 200 with the task unchanged (idempotent operation).

**Given** the board is reloaded after a drag-and-drop move,
**When** `GET /api/projects/{projectId}/tasks` is called,
**Then** the task's `ColumnStatus` reflects the column it was moved to (change is persisted).
