---
trigger: always_on
---

# Architecture Core Principles

This document defines the **core principles** that guide the architecture, code style, and patterns across the entire project.  
All other documents (Data / Domain / Presentation / Offline-First / Naming / Router) are extensions of these principles.

---

## 1. Goals

### 1.1 Product & UX Goals

- The app must **work reliably with bad or no connection** (offline-first mindset).
- The user should get **fast feedback**: screens load from local data, not from the network.
- Errors and sync issues must be **visible, understandable, and recoverable**.
- The UI should feel **consistent across features** (layout, states, loading, errors, sync).

### 1.2 Developer Experience (DX) Goals

- Code should be **predictable, boring in the good way**.
- A new developer should be able to:
  - Find where to place/change something in < 1–2 minutes.
  - Understand the sync strategy without reading the entire codebase.
- The architecture must scale for:
  - New features.
  - New team members.
  - New platforms (mobile/web/desktop) when needed.

---

## 2. Architectural Principles

### 2.1 Clean Architecture & Layers

We follow a **layered architecture** with clear separation of concerns:

- **Presentation layer**
  - Widgets, pages, BLoCs/state managers.
  - No direct access to data sources or repository implementations.
- **Domain layer**
  - Entities, value objects, repository interfaces, use cases.
  - Pure business logic; no frameworks or infrastructure.
- **Data layer**
  - Remote and local data sources, models, mappers, repository implementations.
  - Implements offline-first, outbox, sync, and error mapping.

**Core rule:**  
> Dependencies always flow **inwards**:  
> Presentation → Domain → (interfaces) → Data (implementations).

Details for each layer are defined in:

- `02-data-layer-rules.md`
- `03-domain-layer-rules.md`
- `04-presentation-layer-rules.md`

---

### 2.2 Offline-First by Default

Offline-first is not a feature; it is the **default design**:

- The **local database is the Single Source of Truth (SSoT)**.
- All reads are performed from **local storage first**.
- All writes are **applied locally** and then **enqueued in an outbox**.
- Sync runs in **background** and uses **delta synchronization** when possible.
- Presentation layer shows **local state + sync status**, not “just remote state”.

Details and patterns are defined in:

- `05-offline-first-and-sync-part1-rules.md` 
- `06-offline-first-and-sync-part2-rules.md` 

---

### 2.3 Feature-Oriented Structure

The project is organized by **feature**, not by technical type:

```text
features/
  {feature}/
    data/
    domain/
    presentation/
core/
  shared/
  config/
```

Rules:

- Each feature owns its data/domain/presentation subfolders.
- Shared code goes to core/shared (widgets, data sources, repositories, etc.).
- Configuration & routing live in core/config.

---

## 3. Code Style & Design Principles

### 3.1 General Code Style

- Write concise, technical Dart code with clear intent.
- Prefer composition over inheritance.
- Use descriptive names with auxiliary verbs for booleans:
  - isLoading, hasError, canRetry, shouldSync, etc.
- Keep functions and classes small and focused:
  - One main responsibility.
  - Extract private methods or widgets when a block grows too much.

### 3.2 File & Module Structure

Each file should follow a logical internal structure, for example:

1. Public class / exported widget.
2. Private sub-widgets or helper classes.
3. Private helpers and extensions.
4. Static content / constants / types.

Naming and folder conventions are defined in:

- `01-naming-convention-rules.md`

### 3.3 Functional & Declarative Mindset

UI and state should be declarative:

- “Given this state, render this UI” – not “do X then manually update Y”.
- Prefer immutable data structures for entities and state:
  - Use const constructors where possible.
  - Use copyWith instead of mutating objects.
- Encapsulate side effects (I/O, sync, logging) in:
  - Data layer (repositories/data sources).
  - Dedicated infrastructure services (e.g. sync services).

---

## 4. Error, Loading & Sync Principles

### 4.1 Unified Error Model

All layers use a unified error model (e.g. Result<T> + AppFailure).

- **Data layer:** catches low-level exceptions (HTTP, DB, parsing) and maps them to AppFailure before returning to domain/presentation.
- **Domain layer:** works only with Result, AppFailure, and entities/value objects.
- **Presentation layer:** never throws; it reacts to states like loading, success, error, empty, syncing.

### 4.2 Predictable States in UI

For each screen/BLoC, we explicitly define the possible states, for example:

- Initial
- Loading
- Loaded
- Error
- (optionally) Syncing, Empty, Refreshing

Rules:

Use sealed classes (or union types) for states.

Use exhaustive pattern matching in the UI:

Every switch on state must handle all cases.

Details in:

04-presentation-layer-rules.md

05-offline-first-and-sync-part1-rules.md

06-offline-first-and-sync-part2-rules.md

5. Responsive & Design Mapping Principles
5.1 Responsive Calculation (Extensions / Helpers)
We use centralized responsive helpers (e.g. .dW, .dH, .fS, .responsive()) instead of:

---

## 5. Responsive & Design Mapping Principles

### 5.1 Responsive Calculation (Extensions / Helpers)

We use centralized responsive helpers (e.g. `.dW`, `.dH`, `.fS`, `.responsive()`) instead of:

- Hard-coding pixel values everywhere.
- Repeating MediaQuery logic across widgets.
- Duplicating design tokens / base sizes per widget.

Guidelines (high level):

- Use `.dW` for width-based measurements and horizontal padding.
- Use `.dH` for height-based measurements and vertical spacing.
- Use `.fS` (or equivalent) to map Figma font sizes to responsive text.
- Use `.responsive()` to map design sizes to min/max ranges depending on screen type.

The detailed responsive rules live in:

- `responsive-calculation-guidelines.md` (optional; create if missing).

---

## 6. Router, Navigation & Factories Principles

Navigation is centralized using:

- AppRoutes (route names/paths).
- AppRouter (GoRouter configuration).
- AppNavigator (navigation helpers).
- PageFactory and WidgetFactory (dependency-aware creators).

Routing is decoupled from concrete page implementations:

- Router only knows about factories, not about internal widgets.
- Auth and connectivity integrate via:
  - AppRouterRefreshNotifier listening to auth + network to trigger redirects.

Details in:

- `07-routes-and-factories-part1-rules.md` 
- `08-routes-and-factories-part2-rules.md`

---

## 7. How to Use These Core Principles

When creating or modifying code:

- Start from the architecture:
  - Which feature?
  - Which layer (data / domain / presentation)?
- Apply offline-first mindset:
  - Read from local first.
  - Write locally + enqueue outbox.
  - Expose sync state.
- Respect naming, structure, and folder boundaries:
  - Follow `01-naming-convention-rules.md`.
- Design explicit states:
  - What are the valid states for this screen/BLoC?
  - How does the UI look in each one?
- Keep it small and testable:
  - One main responsibility per class/file.
  - Move cross-cutting or shared logic to core/shared.

---

## 8. Related Documents

This document is the entry point to the architecture.  
For details, see:

- `02-data-layer-rules.md`
- `03-domain-layer-rules.md`
- `04-presentation-layer-rules.md`
- `05-offline-first-and-sync-part1-rules.md`
- `06-offline-first-and-sync-part2-rules.md`
- `07-routes-and-factories-part1-rules.md`
- `08-routes-and-factories-part2-rules.md`
