# Architecture Rules Index

This document is the **entry point** to the project’s architecture handbook.  
Each section links to a more detailed document under `docs/architecture/`.

> Rule of thumb:  
> **Before creating or changing code, identify which layer/section applies and read that rule file.**

---

## 1. Core Principles

**File:** `docs/architecture/00-core-principles.md`

Defines the **global architecture philosophy**:

- Goals for UX and Developer Experience.
- Clean Architecture layering (Presentation → Domain → Data).
- Offline-first as the default design.
- Feature-oriented folder structure.
- General code style and design principles.
- Error/loading/sync principles.
- High-level router & factories principles.
- How to use the rest of the handbook.

Read this **first** when onboarding or reviewing decisions.

---

## 2. Naming Conventions

**File:** `docs/architecture/01-naming-convention-rules.md`

Defines **file naming, folder structure, and validation regexes**:

- Patterns per layer (data/domain/presentation + sync/outbox).
- Location constraints (where repositories, blocs, etc. live).
- Example validation script (`validate_naming.sh`).
- Example `dart_code_metrics` snippet.
- Naming checklist.

Use this when **creating new files** or reviewing **PR structure/naming**.

---

## 3. Data Layer Rules

**File:** `docs/architecture/02-data-layer-rules.md`

Defines how the **data layer** must work:

- Responsibilities:
  - Local data sources (DB).
  - Remote data sources (APIs).
  - Repository implementations.
  - Mappers and models (DTOs, tables).
- Offline-first patterns in the data layer:
  - Read from local DB first.
  - Write locally + enqueue outbox.
  - Sync patterns (delta sync, error mapping).
- Allowed and forbidden dependencies.
- Naming and folder conventions.
- Checklist before merging new data-layer code.

Use this when adding/changing **repositories, data sources, models or mappers**.

---

## 4. Domain Layer Rules

**File:** `docs/architecture/03-domain-layer-rules.md`

Defines the **domain layer** contracts and business logic:

- Entities and value objects (immutable where possible).
- Repository interfaces (no implementations, no frameworks).
- Use case rules:
  - When to create a use case (decision matrix).
  - When to call repositories directly from BLoCs.
- Allowed/forbidden dependencies (no framework, no data layer).
- Naming and folder conventions.
- Checklist for new domain code.

Use this when touching **business rules, entities, or use cases**.

---

## 5. Presentation Layer Rules

**File:** `docs/architecture/04-presentation-layer-rules.md`

Defines how the **UI layer** should be structured:

- Responsibilities:
  - BLoCs/state managers.
  - Pages/screens.
  - Widgets/components.
- Rules for widgets:
  - No `_build*` helper methods.
  - Prefer private widgets (`_SomeSection`) over huge build methods.
  - One widget = one clear responsibility.
- BLoC rules:
  - Sealed states and events.
  - Exhaustive pattern matching in UI.
  - Proper subscription management (`close()`).
- Shared vs feature-specific widgets.
- Naming and folder conventions.
- Checklist before merging UI changes.

Use this when creating/modifying **pages, widgets, or BLoCs**.

---

## 6. Offline-First & Sync Rules

**Files:** 
- `docs/architecture/05-offline-first-and-sync-part1-rules.md`
- `docs/architecture/06-offline-first-and-sync-part2-rules.md`

Define the **offline-first and synchronization strategy**:

- Local DB as **Single Source of Truth (SSoT)**.
- Read flow: always from local DB first, then background sync.
- Outbox pattern: enqueue → send → confirm → retry flows.
- Delta sync: timestamps/version tokens, conflict strategies.
- Responsibilities by layer (data/domain/presentation).
- End-to-end sync flows (push + pull).
- Naming conventions for sync/outbox components.
- Checklist for new features that participate in sync.

Use this when implementing anything related to **sync, outbox, or offline behavior**.

---

## 7. Router & Factories Rules

**Files:** 
- `docs/architecture/07-routes-and-factories-part1-rules.md`
- `docs/architecture/08-routes-and-factories-part2-rules.md`

Define how **routing and factories** are structured:

- Responsibilities for:
  - `PageFactory` and `WidgetFactory`.
  - `AppRouter` (GoRouter configuration).
  - `AppRoutes` enum (paths, names, protected routes).
  - `AppNavigator` helpers.
  - `AppRouterRefreshNotifier` (auth + connectivity).
- Factories:
  - How to create pages with BLoCs and initial events.
  - How to create dependency-aware widgets/modals.
- Router:
  - Redirect rules (auth, offline).
  - Handling path and query parameters.
  - Wizard and nested route patterns (ShellRoute).
- Naming and folder conventions for router/config.
- Checklist for adding new routes.

Use this when adding/changing **routes, navigation flows, or factories**.

---

## 8. (Optional) Responsive Calculation Guidelines

**File (suggested):** `docs/architecture/responsive-calculation-guidelines.md`

Recommended content:

- How to use responsive helpers (e.g. `.dW`, `.dH`, `.fS`, `.responsive()`).
- How to map Figma design values to code.
- Rules per breakpoint (mobile / tablet / desktop).
- Examples for spacing, typography, and layout components.

Use this when working on **layout, spacing, and typography**.

---

## 9. How to Use This Handbook in Practice

### 9.1 For New Features

1. Identify the feature: `features/{feature}/`.
2. Decide which layers are affected:
   - Data? Domain? Presentation? Sync? Router?
3. Before coding, skim:
   - `00-core-principles.md`
   - Plus the relevant layer documents.
4. While implementing:
   - Follow naming and folder rules (`01-naming-convention-rules.md`).
   - Apply offline-first patterns when data is involved (`05-offline-first-and-sync-part1-rules.md` + `part2`).
5. Before merging:
   - Use the checklist in each relevant section.

### 9.2 For Code Reviews

- Reference the documents directly in PR comments:
  - “This violates Data Layer Rule X (see `02-data-layer-rules.md`).”
  - “This widget should be split (see `04-presentation-layer-rules.md`).”
- Use the checklists as a **review template**.

### 9.3 For Onboarding

- Step 1: Read `00-core-principles.md`.
- Step 2: Read `04-presentation-layer-rules.md` (UI dev) or `02-data-layer-rules.md` (backend-ish dev).
- Step 3: Skim `01-naming-convention-rules.md` for structure/naming.
- Step 4: Implement a small feature following these rules.
- Step 5: Review the implementation explicitly against this index.

---

If you add a new architecture-related document under `docs/architecture/`, update this index with:

- File name.
- 3–5 bullet summary.
- When to use it.
