---
trigger: glob
globs: lib/**/data/**/*.dart
---

# Data Layer Rules

## 1. Purpose of the Data Layer

The **data layer** is responsible for:

* Implementing all **data access** (local and remote).
* Converting between **data models** (DTOs, ORM/DB models) and **domain entities**.
* Implementing the **repository interfaces** defined in the domain layer.
* Managing **local persistence** (database, cache, file storage, etc.).
* Handling **synchronization** with remote services.
* Implementing the **offline-first strategy**, when the project requires it.

---

## 2. Allowed Responsibilities

### 2.1 Data Sources

**Local data sources**

* Read/write operations on the local database or storage.
* Complex queries using DAOs/Repositories specific to the DB technology.
* Transactional operations and batch writes.

**Remote data sources**

* HTTP/HTTPS (or gRPC, etc.) calls to remote APIs.
* Serialization / deserialization of DTOs.
* Basic network error handling and retries.

### 2.2 Repository Implementations

Repositories in the data layer must:

* Implement the **repository interfaces** defined in `domain/repositories`.
* Coordinate between **local** and **remote** data sources.
* Apply the **offline-first policy** when applicable:

  * Prefer reading from **local storage**.
  * Trigger background sync when network is available.
  * Use an **outbox** (or similar mechanism) for pending write operations.
* Wrap results in a unified result type
  (e.g. `Result<T>`, `Either<Failure, T>` or project-specific equivalent).
* Map low-level errors to **domain-level failures** (e.g. `AppFailure`).

### 2.3 Mappers

* Convert between:

  * **Data models** (DTOs, DB models) ↔ **Domain entities**.
* Live in `data/mappers/` (or equivalent module).
* Can be implemented as:

  * Static functions, or
  * Extension/utility methods.
* **No business rules** inside mappers; only structural mapping.

### 2.4 Models

* **DTOs**: structures used to communicate with APIs or external systems.
* **Database/ORM models**: structures used to map DB tables/collections.
* Located under `data/models/` and/or `data/tables/` (or equivalent).
* Contain **data only**, no domain logic.

---

## 3. Forbidden Responsibilities

The data layer **must not**:

* Contain **business rules or domain logic**
  (complex decisions, policies, workflows).
* Be accessed **directly from the presentation/UI layer**.
* Depend on **UI frameworks** (widgets, view components, controllers).
* Implement **validation rules** that belong to the domain
  (only basic format / type / parsing validation is allowed).

---

## 4. Dependency Rules

### 4.1 Allowed Dependencies

Data sources may depend on:

* Infrastructure services:

  * Database client / ORM.
  * HTTP client or API client.
  * Cache, key-value storage.
* Cross-cutting services:

  * Logging, metrics, tracing.
  * Network status checker.
* Data-layer types:

  * DTOs, DB models, mappers.
  * Result/Failure abstractions shared across layers.

Repository implementations may depend on:

* Local and remote **data sources**.
* Mappers between data models and entities.
* Network status / connectivity services.
* Result/failure abstractions.

### 4.2 Forbidden Dependencies

The data layer **must not** depend on:

* `presentation` (UI, controllers, view models, BLoCs, etc.).
* `domain` **implementation details** (use cases, domain services, etc.).
* Any **framework-specific UI** or platform component.

Allowed exception:

* Repositories in the data layer may depend on **repository interfaces** defined in `domain/repositories`.

---

## 5. Offline-First & Sync Rules

> Applies when the project uses offline-first or background sync.

### 5.1 Read Path (Single Source of Truth)

* The **local store is the Single Source of Truth (SSoT)**.
* Repositories must:

  * Read from **local storage first** (DB/cache).
  * Return data to the caller as soon as the local read completes.
  * Optionally trigger a **background sync** to refresh local data.
* Remote reads must **update local storage**, not bypass it.

### 5.2 Write Path with Outbox

* All write operations follow this sequence:

  1. **Write to local storage** (insert/update the local model).
  2. **Enqueue** an entry in the **outbox** with:

     * Entity type / table name.
     * Local entity ID.
     * Action (create, update, delete).
     * Serialized payload needed by the API.
  3. A separate sync process reads the outbox, sends changes to the server,
     and updates local records with remote IDs / statuses.

* The repository method returns the **local result** (e.g. local ID) and does not block on the remote sync.

### 5.3 Delta Sync (Per Table / Entity Type)

* Remote data sources for sync endpoints should:

  * Accept a **timestamp or version token** (e.g. `lastSync`) to request only changes since the last sync.
  * Return:

    * New / updated records.
    * Optionally, a list of deleted IDs or a tombstone marker.
  * Handle “already up-to-date” responses gracefully.

* After a successful sync:

  * The data layer updates local tables/collections accordingly.
  * The **lastSync token** is persisted locally per table/entity type.

### 5.4 Error Handling in Sync

* Network or server errors during sync **must not** corrupt local data.
* Sync operations should:

  * Log failures.
  * Mark outbox items for retry (with backoff).
  * Not crash the app or block local reads.

---

## 6. Naming & Location Conventions

> Use “feature” or “module” as the generic word for domain areas (e.g. `users`, `orders`, `visits`).

### 6.1 Data Sources

* **Local data source** file names:
  `{feature}_local_data_source.*`
  e.g. `users_local_data_source.dart`, `ordersLocalDataSource.cs`
* **Remote data source** file names:
  `{feature}_remote_data_source.*`
* Location:
  `features/{feature}/data/data_sources/` (or equivalent module path).

### 6.2 Repositories

* **Implementation** file names:
  `{feature}_repository_impl.*`
* Location:
  `features/{feature}/data/repositories/`
* The **interfaces** live in:
  `features/{feature}/domain/repositories/`
  and must **not** use the `_impl` suffix.

### 6.3 Mappers

* File names:
  `{entity}_mapper.*`
* Location:
  `features/{feature}/data/mappers/`
* Only mapping logic (no I/O, no business rules).

### 6.4 Models

* **DTOs**: `{entity}_dto.*` or `{entity}_model.*`
  Location: `features/{feature}/data/models/`
* **Database/ORM models**: `{entity}_db_model.*` or `{entity}_table.*`
  Location: e.g. `core/data/database/tables/` or similar.

### 6.5 DAOs / Repositories for DB

* File names: `{entity}_dao.*` or `{entity}_db_repository.*`
* Location: under the database module, for example:
  `core/data/database/daos/`

---

## 7. Checklist for New Features (Data Layer)

Before merging a feature that touches the data layer, verify:

* [ ] All repositories implement the expected **domain interfaces**.
* [ ] Reads follow the **SSoT rule**: data comes from local storage.
* [ ] Write operations that should sync remotely **use the outbox**.
* [ ] Mappers correctly convert **data models ↔ domain entities**.
* [ ] Results are wrapped in the project’s **Result/Failure abstraction**.
* [ ] Errors from data sources are translated into **domain failures**.
* [ ] File names and locations follow the **naming conventions** above.
* [ ] There are no **forbidden dependencies** (no direct UI/domain-impl access).
* [ ] Sync endpoints use **delta sync** or are explicitly documented if they don’t.
* [ ] New DB operations are covered by **tests** (where applicable).

---

*Data layer architecture rules — generic, offline-first friendly, and reusable in any project.*