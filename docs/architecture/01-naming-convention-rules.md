---
trigger: always_on
---

Naming and Regex Conventions
1. File Naming Patterns by Layer

All file names use snake_case and end with .dart.

1.1 Data Layer
Local Data Sources

Pattern: *_local_data_source.dart
Examples:

children_local_data_source.dart

centers_local_data_source.dart
Regex:

^[a-z_]+_local_data_source\.dart$

Remote Data Sources

Pattern: *_remote_data_source.dart
Examples:

children_remote_data_source.dart

centers_remote_data_source.dart
Regex:

^[a-z_]+_remote_data_source\.dart$

Repository Implementations

Pattern: *_repository_impl.dart
Examples:

children_repository_impl.dart

planning_repository_impl.dart
Regex:

^[a-z_]+_repository_impl\.dart$


Note: These live only in data/repositories/, never in domain/.

Mappers

Pattern: *_mapper.dart
Examples:

child_mapper.dart

planning_mapper.dart
Regex:

^[a-z_]+_mapper\.dart$

Models

DTOs: *_model.dart
Examples:

child_model.dart

center_model.dart
Regex:

^[a-z_]+_model\.dart$


DB / Table models: *_table.dart
Examples:

children_table.dart

sync_outbox_table.dart
Regex:

^[a-z_]+_table\.dart$

DAOs (or DB Repositories)

Pattern: *_dao.dart
Examples:

children_dao.dart

centers_dao.dart
Regex:

^[a-z_]+_dao\.dart$

1.2 Domain Layer
Entities

Pattern: *.dart or *_entity.dart (optional suffix)
Examples:

child.dart

planning.dart

child_entity.dart
Regex:

^[a-z_]+(?:_entity)?\.dart$


Convention: Prefer names without the _entity suffix, as long as they are clear.

Repository Interfaces

Pattern: *_repository.dart (no _impl)
Examples:

children_repository.dart

planning_repository.dart
Regex:

^[a-z_]+_repository\.dart$

Use Cases / Interactors

Pattern: *_use_case.dart or *_usecase.dart (must be consistent across the project)
Examples:

get_children_for_current_user_use_case.dart

create_visit_usecase.dart
Regex:

^[a-z_]+_(use_case|usecase)\.dart$


Convention: Verb in present tense, snake_case.
Recommendation: Standardize on *_use_case.dart if starting from scratch.

Value Objects

Pattern: *.dart or *_value_object.dart (optional suffix)
Examples:

form_runtime_context.dart

form_runtime_context_value_object.dart
Regex:

^[a-z_]+(?:_value_object)?\.dart$


Convention: Prefer names without the _value_object suffix, as long as they are clear.

1.3 Presentation Layer
Pages / Screens

Pattern: *_page.dart
Examples:

children_list_page.dart

planning_detail_page.dart
Regex:

^[a-z_]+_page\.dart$

Widgets

Pattern: *_widget.dart
Examples:

child_card_widget.dart

sync_status_widget.dart
Regex:

^[a-z_]+_widget\.dart$

BLoCs (or Similar State Managers)

BLoC class: *_bloc.dart
Regex:

^[a-z_]+_bloc\.dart$


State: *_state.dart
Regex:

^[a-z_]+_state\.dart$


Event: *_event.dart
Regex:

^[a-z_]+_event\.dart$


Structure options:

# Multi-file structure
blocs/
  customers_bloc/
    customers_bloc.dart
    customers_state.dart
    customers_event.dart

# Or single file with parts
blocs/
  customers_bloc.dart   // with 'part' for state/event

1.4 Core / Shared
Sync / Outbox Naming

Outbox-related: *_outbox_*.dart
Examples:

outbox_local_data_source.dart

outbox_remote_data_source.dart

outbox_repository.dart
Regex:

^[a-z_]*outbox[a-z_]*\.dart$


Sync-related: *_sync_*.dart
Examples:

sync_local_data_source.dart

sync_remote_data_source.dart

sync_repository.dart

sync_status_bloc.dart
Regex:

^[a-z_]*sync[a-z_]*\.dart$

2. File Location Rules

Applies to a feature-based architecture like features/{feature}/data | domain | presentation.

2.1 Feature Structure
features/
  {feature}/
    data/
      data_sources/
        *_local_data_source.dart
        *_remote_data_source.dart
      mappers/
        *_mapper.dart
      models/
        *_model.dart
      repositories/
        *_repository_impl.dart

    domain/
      entities/
        *.dart  (or *_entity.dart)
      repositories/
        *_repository.dart
      use_cases/   (or usecases/)
        *_use_case.dart (or *_usecase.dart)
      value_objects/
        *.dart  (or *_value_object.dart)

    presentation/
      blocs/
        *_bloc.dart
        *_state.dart
        *_event.dart
      pages/
        *_page.dart
      widgets/
        *_widget.dart

2.2 Core / Shared Structure (Example)
core/
  shared/
    data/
      data_sources/
        outbox_local_data_source.dart
        outbox_remote_data_source.dart
        sync_local_data_source.dart
        sync_remote_data_source.dart
      repositories/
        outbox_repository_impl.dart
        sync_repository_impl.dart

    domain/
      repositories/
        outbox_repository.dart
        sync_repository.dart

    blocs/
      sync_status/
        sync_status_bloc.dart
        sync_status_state.dart
        sync_status_event.dart

    widgets/
      sync_status_widget.dart
      sync_loading_view.dart
      sync_success_view.dart

3. Sync / Outbox Naming Conventions
3.1 Outbox

Data sources:

outbox_local_data_source.dart

outbox_remote_data_source.dart

Repository interface:

outbox_repository.dart (domain)

Repository implementation:

outbox_repository_impl.dart (data)

DB table / model:

sync_outbox_table.dart

DAO:

outbox_dao.dart

3.2 Sync

Data sources:

sync_local_data_source.dart

sync_remote_data_source.dart

Repository interface:

sync_repository.dart (domain)

Repository implementation:

sync_repository_impl.dart (data)

BLoC / ViewModel:

sync_status_bloc.dart

sync_status_state.dart

sync_status_event.dart

Widgets:

sync_status_widget.dart

sync_loading_view.dart

sync_success_view.dart

4. Regex Library (for Validation)
4.1 Data Layer
# Local Data Source
^[a-z_]+_local_data_source\.dart$

# Remote Data Source
^[a-z_]+_remote_data_source\.dart$

# Repository Implementation
^[a-z_]+_repository_impl\.dart$

# Mapper
^[a-z_]+_mapper\.dart$

# Model
^[a-z_]+_model\.dart$

# DB Table
^[a-z_]+_table\.dart$

# DAO
^[a-z_]+_dao\.dart$

4.2 Domain Layer
# Entity
^[a-z_]+(?:_entity)?\.dart$

# Repository Interface
^[a-z_]+_repository\.dart$

# Use Case
^[a-z_]+_(use_case|usecase)\.dart$

# Value Object
^[a-z_]+(?:_value_object)?\.dart$

4.3 Presentation Layer
# Page
^[a-z_]+_page\.dart$

# Widget
^[a-z_]+_widget\.dart$

# BLoC
^[a-z_]+_bloc\.dart$

# State
^[a-z_]+_state\.dart$

# Event
^[a-z_]+_event\.dart$

5. Regex for Location Constraints
5.1 *_repository_impl.dart Must Be in data/repositories
^features/[a-z_]+/data/repositories/[a-z_]+_repository_impl\.dart$

5.2 *_repository.dart Must Be in domain/repositories
^features/[a-z_]+/domain/repositories/[a-z_]+_repository\.dart$

5.3 *_bloc.dart Must Be in presentation/blocs
^features/[a-z_]+/presentation/blocs/[a-z_]+_bloc\.dart$


(Adjust the base path features/ if your project uses a different structure.)

6. Example Validation Script

Example Bash script to enforce some naming/location rules:

#!/bin/bash
# validate_naming.sh

set -e

# 1. Validate that *_repository_impl.dart is in data/repositories
find lib/features -name "*_repository_impl.dart" | while read -r file; do
  if [[ ! "$file" =~ features/[a-z_]+/data/repositories/ ]]; then
    echo "ERROR: $file should be in data/repositories/"
    exit 1
  fi
done

# 2. Validate that *_repository.dart (interfaces) are in domain/repositories
find lib/features -name "*_repository.dart" ! -name "*_repository_impl.dart" | while read -r file; do
  if [[ ! "$file" =~ features/[a-z_]+/domain/repositories/ ]]; then
    echo "ERROR: $file should be in domain/repositories/"
    exit 1
  fi
done

# 3. Validate BLoC file naming
find lib/features -name "*_bloc.dart" | while read -r file; do
  filename=$(basename "$file")
  if [[ ! "$filename" =~ ^[a-z_]+_bloc\.dart$ ]]; then
    echo "ERROR: $file does not follow *_bloc.dart naming."
    exit 1
  fi
done

echo "✅ All naming validations passed!"

7. Example dart_code_metrics Rules (Optional)
# analysis_options.yaml
dart_code_metrics:
  rules:
    - prefer-single-widget-per-file
    - avoid-private-typedef-functions
  anti-patterns:
    - long-method
    - long-parameter-list
  metrics:
    cyclomatic-complexity: 20
    maximum-nesting-level: 5
    number-of-parameters: 4
    source-lines-of-code: 50


You can extend this section with custom rules later (e.g. scanning widgets importing *_repository_impl.dart etc.).

8. Naming Checklist

Before creating a new file, verify:

 The file name matches the correct pattern for its type (entity, model, repo, widget, etc.).

 The file is placed in the correct directory for its layer (data, domain, presentation).

 There are no naming conflicts (e.g. repository interface vs implementation).

 File names use snake_case consistently.

 Suffixes are consistent: _impl only in the data layer, not in domain.

 Use case files follow one chosen convention (*_use_case.dart or *_usecase.dart) across the whole project.