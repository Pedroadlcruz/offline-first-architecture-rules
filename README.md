# Offline-First Architecture Rules

This repository centralizes the **architecture rules, patterns, and checklists** I use to build offline-first applications.

The main ideas:

- **Local DB as Single Source of Truth (SSoT)** – the app always reads from local storage first.
- **Outbox + delta sync** – writes are applied locally and enqueued for background synchronization.
- **Clean layering** – clear separation between `presentation`, `domain`, and `data`.
- **Consistent naming & folder structure** – predictable file names and locations (repositories, data sources, widgets, BLoCs, etc.).
- **Router & factories** – centralized navigation and dependency injection for pages and widgets.

The docs under `docs/architecture/` describe how each layer should behave:

- `01_core_principles.md` – overall philosophy and goals.
- `02_data_layer_rules.md` – repositories, data sources, models, mappers, offline-first in data.
- `03_domain_layer_rules.md` – entities, value objects, repository interfaces, use cases.
- `04_presentation_layer_rules.md` – pages, widgets, BLoCs, UI states.
- `05_offline_first_and_sync.md` – SSoT, outbox, delta sync, conflicts.
- `06_naming_and_regex.md` – naming conventions and validation regexes.
- `07_router_and_factories.md` – router, navigation helpers, page/widget factories.

You can use this repo as:

- A **handbook** for your own projects.
- A **reference** for onboarding new teammates or interns.
- A **contract** for AI/code agents so they follow your architecture rules.
