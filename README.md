# Offline-First Architecture Rules

This repository centralizes the **architecture rules, patterns, and checklists** I use to build offline-first applications.

The main ideas:

- **Local DB as Single Source of Truth (SSoT)** – the app always reads from local storage first.
- **Outbox + delta sync** – writes are applied locally and enqueued for background synchronization.
- **Clean layering** – clear separation between `presentation`, `domain`, and `data`.
- **Consistent naming & folder structure** – predictable file names and locations (repositories, data sources, widgets, BLoCs, etc.).
- **Router & factories** – centralized navigation and dependency injection for pages and widgets.

The docs under `docs/architecture/` describe how each layer should behave:

- `00-core-principles.md` – overall philosophy and goals.
- `01-naming-convention-rules.md` – naming conventions and validation regexes.
- `02-data-layer-rules.md` – repositories, data sources, models, mappers, offline-first in data.
- `03-domain-layer-rules.md` – entities, value objects, repository interfaces, use cases.
- `04-presentation-layer-rules.md` – pages, widgets, BLoCs, UI states.
- `05-offline-first-and-sync-part1-rules.md` – SSoT, read/write flows, outbox, delta sync.
- `06-offline-first-and-sync-part2-rules.md` – conflicts, retries, edge cases.
- `07-routes-and-factories-part1-rules.md` – router configuration, navigation helpers.
- `08-routes-and-factories-part2-rules.md` – factories for pages/widgets and composition patterns.
- *(Optional)* `responsive-calculation-guidelines.md` – responsive helpers for layout/typography.

You can use this repo as:

- A **handbook** for your own projects.
- A **reference** for onboarding new teammates or interns.
- A **contract** for AI/code agents so they follow your architecture rules.
