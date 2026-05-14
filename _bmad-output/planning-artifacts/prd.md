---
stepsCompleted: ['step-01-init', 'step-02-discovery', 'step-02b-vision', 'step-02c-executive-summary', 'step-03-success', 'step-04-journeys', 'step-05-domain', 'step-06-innovation', 'step-07-project-type', 'step-08-scoping', 'step-09-functional', 'step-10-nonfunctional', 'step-11-polish']
releaseMode: single-release
inputDocuments: []
workflowType: 'prd'
classification:
  projectType: web_app
  domain: general
  complexity: low
  projectContext: greenfield
---

# Product Requirements Document - Kanban-Workshop

**Author:** EWN Workshop
**Date:** 2026-05-14

## Executive Summary

Kanban Workshop is a lightweight Blazor Server application with a .NET Web API backend, designed as a single-user Kanban task manager. The app is intentionally minimal — multiple projects, each with a task board of fixed columns (Backlog, In Progress, Done), SQLite persistence, and drag-and-drop card movement. The target user is someone familiar with Scrum/Kanban who wants to manage personal tasks across multiple projects. The codebase is structured with clean layer separation to serve as a teaching vehicle for .NET unit testing workshops, though the PRD defines the application itself.

### What Makes This Special

Deliberate simplicity is the design constraint. The domain is immediately legible to any developer familiar with agile practices, requiring no domain ramp-up. The architecture exposes clean testable seams across domain entities, service layer, API controllers, and Blazor UI — making it an effective reference implementation for unit testing instruction.

## Project Classification

- **Project Type:** Blazor Server Web App with .NET REST API backend
- **Domain:** General (task management / productivity)
- **Complexity:** Low
- **Project Context:** Greenfield
- **Primary Stack:** .NET 8+ / Blazor Server / SQLite / Entity Framework Core

## Success Criteria

### User Success

- User can create and manage multiple independent projects
- Each project displays tasks as a Kanban board with fixed columns
- Tasks can be created, edited, and moved across columns
- Board state persists between sessions

### Technical Success

- Clean layer separation: domain entities, service layer, API controllers, Blazor UI
- Each layer is independently testable with no tight coupling
- No business logic in controllers or UI components

### Measurable Outcomes

- All core CRUD operations (projects and tasks) function correctly
- Board state is persisted and restored accurately on application reload

## Product Scope

### Single Release — Complete Feature Set

All capabilities below ship as one release. No phased rollout.

**Must-Have Capabilities:**

- Project CRUD (create, rename, delete with confirmation)
- Task CRUD (create, edit, delete) — title and description fields
- Fixed columns per project: Backlog, In Progress, Done
- Drag-and-drop card movement between columns
- SQLite persistence via EF Core
- Input validation (no empty task titles)
- Confirmation dialog on destructive actions
- Project list / navigation between projects

### Future Considerations (No Planned Delivery)

- Custom column names per project
- Task ordering within columns
- Due dates or priority flags

## User Journeys

### Journey 1: Alex Sets Up a New Project

Alex has a list of tasks rattling around in their head for a home renovation project. They open the app, create a new project called "Home Reno," and are presented with a fresh board — Backlog, In Progress, Done. They add several tasks to the Backlog: "Get quotes," "Buy paint," "Schedule contractor." They drag "Get quotes" to In Progress. The board reflects exactly where things stand. Next session, they return and the state is exactly as they left it.

**Capabilities revealed:** Project creation, task creation, column assignment, card movement, persistence.

### Journey 2: Alex Manages Multiple Projects

Alex now has three active projects: Home Reno, Work Goals, Side Project. They switch between projects from a project list. Each board is independent — tasks from one project don't appear on another. They complete all tasks in Work Goals and delete the project.

**Capabilities revealed:** Project list view, project switching, project deletion.

### Journey 3: Alex Hits an Edge Case

Alex accidentally creates a task with no title and tries to save it. The app prevents submission and prompts for a title. Later, Alex tries to delete a project that still has tasks — the app confirms the action before proceeding.

**Capabilities revealed:** Input validation, confirmation dialogs, empty state handling.

### Journey Requirements Summary

| Capability | Source Journey |
| --- | --- |
| Project CRUD | Journeys 1, 2 |
| Task CRUD | Journeys 1, 3 |
| Column assignment & card movement | Journey 1 |
| Project list / navigation | Journey 2 |
| SQLite persistence | Journey 1 |
| Input validation | Journey 3 |
| Confirmation on destructive actions | Journey 3 |

## Web App Specific Requirements

### Rendering & Architecture

- **Rendering model:** Blazor Server (server-side rendering over SignalR)
- **Frontend/backend split:** Blazor UI project communicates with a separate .NET Web API project via HTTP
- **Database:** SQLite via Entity Framework Core
- **No real-time sync required** — single user, no concurrent session reconciliation needed

### Browser Support

Latest versions of Chrome, Edge, and Firefox. No legacy browser support required.

### Responsive Design

Desktop viewport is the primary target. Mobile-friendly is not required.

### Constraints

No authentication, no multi-tenancy, no SEO requirements — single-user local application.

## Functional Requirements

### Project Management

- **FR1:** User can create a new project with a name
- **FR2:** User can rename an existing project
- **FR3:** User can delete a project
- **FR4:** User can view a list of all projects

### Task Management

- **FR5:** User can create a task within a project with a title and description
- **FR6:** User can edit a task's title and description
- **FR7:** User can delete a task
- **FR8:** User can assign a task to a column at creation time

### Board Interaction

- **FR9:** User can view a project's tasks organized into Backlog, In Progress, and Done columns
- **FR10:** User can move a task between columns via drag-and-drop
- **FR11:** Each project board displays exactly three fixed columns: Backlog, In Progress, Done

### Navigation

- **FR12:** User can navigate to a project board from the project list
- **FR13:** User can return to the project list from a board view

### Data Persistence

- **FR14:** All project and task data persists between application sessions
- **FR15:** Board state (tasks and their column assignments) is restored accurately when a project is reopened

### Data Integrity

- **FR16:** System prevents creation of a task with an empty title
- **FR17:** System requires confirmation before deleting a project
- **FR18:** System requires confirmation before deleting a task
- **FR19:** System displays an appropriate empty state when a project has no tasks

## Non-Functional Requirements

### Performance

- CRUD operations (create, edit, delete for projects and tasks) complete and reflect in the UI within 1 second under normal localhost/LAN conditions
- Board load (project open with up to 100 tasks) completes within 2 seconds
- Drag-and-drop card movement provides immediate visual feedback with no perceptible lag

### Accessibility

- All interactive elements are reachable via keyboard
- All form inputs have associated labels
- Page structure uses semantic HTML elements (headings, lists, buttons, forms)
