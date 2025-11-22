---
trigger: glob
globs: lib/**/presentation/**/*.dart
---

# Presentation Layer Rules

## 1. Purpose of the Presentation Layer

The **presentation layer** is responsible for:

- Managing **UI state** (e.g. via BLoCs, Cubits, or another state-management solution).
- Rendering **widgets/screens/pages**.
- Handling **user interactions** (taps, forms, navigation triggers).
- Exposing **sync status and errors** to the user (for offline-first scenarios).
- Driving **navigation** between screens.

The presentation layer should be **thin in business logic** and focused on UI concerns.

---

## 2. Allowed Responsibilities

### 2.1 State Management (BLoCs / Cubits / ViewModels)

State-management classes (e.g. BLoCs):

- Manage the **UI state** for a specific screen/flow.
- React to **user events** and other inputs.
- Call **domain layer** (repositories or use cases) to load/save data.
- Transform domain data into **UI-friendly structures**.
- Expose clear **loading / success / error** states.
- Optionally expose **sync states** (pending items, last sync, etc.).

### 2.2 Screens / Pages

Screens (pages):

- Represent a **full screen** or route in the app.
- Configure BLoCs/ViewModels via dependency injection (e.g. `BlocProvider`).
- Compose **smaller widgets**.
- Trigger **navigation** based on user actions and state.
- Should be **light**: minimal logic, mostly composition and wiring.

### 2.3 Widgets / Components

Widgets:

- Reusable **UI components**.
- Help split complex UIs into **small, focused pieces**.
- Can be:
  - **Feature-specific** (live in that feature’s `presentation/widgets`).
  - **Shared** (live in a global shared widgets module).

Rules:

- Prefer **composition** over huge `build` methods.
- Use **pure widgets** (receive data via constructor) when possible.
- Use widgets that connect to state (via `BlocBuilder`, etc.) only when needed.

### 2.4 Events and States

- Use **events** to model user actions and external triggers.
- Use **states** to model all possible UI configurations.
- Prefer **sealed classes** or equivalent (union types) for:
  - Strong typing.
  - Exhaustive pattern matching.

---

## 3. Forbidden Responsibilities

The presentation layer **must not**:

- Call **data sources** directly (DB, APIs, etc.).
- Instantiate or use **repository implementations** (`*_repository_impl`).
- Implement **complex business logic rules** (those belong in the domain layer).
- Contain **infrastructure concerns** (network, persistence, etc.).
- Do anything other than **simple UI-level validation** (e.g. required fields, formats).

---

## 4. Dependency Rules

### 4.1 Allowed Dependencies

The presentation layer may depend on:

- **Domain layer**:
  - Entities.
  - Repository interfaces.
  - Use cases / interactors.
  - Value objects and failures.
- **Core/common modules**:
  - Constants, theming, helpers, extensions.
- **UI framework & state-management library**:
  - Flutter widgets, BLoC, navigation library, etc.

### 4.2 Forbidden Dependencies

The presentation layer must **not** depend on:

- `data/` layer:
  - DTOs, DB models, data sources.
  - Repository implementations (`*_repository_impl`).
- Infrastructure-specific modules (DB, HTTP client, etc.).

---

## 5. Widget Rules

### 5.1 No `_build*` Helper Methods

Avoid large widgets that use many `_build*` methods inside the same class:

```dart
// ❌ Avoid this pattern
class CustomersPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          _buildHeader(),
          _buildCustomersList(),
        ],
      ),
    );
  }

  Widget _buildHeader() {
    // ...
  }

  Widget _buildCustomersList() {
    // ...
  }
}
5.2 Prefer Private Widgets

Split complex UI into private widget classes instead:

// ✅ Preferred pattern
class CustomersPage extends StatelessWidget {
  const CustomersPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: const [
          _CustomersHeader(),
          _CustomersList(),
        ],
      ),
    );
  }
}

class _CustomersHeader extends StatelessWidget {
  const _CustomersHeader();

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(16),
      child: const Text('Customers'),
    );
  }
}

class _CustomersList extends StatelessWidget {
  const _CustomersList();

  @override
  Widget build(BuildContext context) {
    return Expanded(
      child: BlocBuilder<CustomersBloc, CustomersState>(
        builder: (context, state) {
          return switch (state) {
            CustomersLoading() => const Center(child: CircularProgressIndicator()),
            CustomersLoaded(:final customers) => ListView.builder(
                itemCount: customers.length,
                itemBuilder: (context, index) => _CustomerCard(customer: customers[index]),
              ),
            CustomersError(:final message) => Center(child: Text('Error: $message')),
            _ => const SizedBox.shrink(),
          };
        },
      ),
    );
  }
}

class _CustomerCard extends StatelessWidget {
  final Customer customer;
  const _CustomerCard({required this.customer});

  @override
  Widget build(BuildContext context) {
    return ListTile(
      title: Text(customer.name),
      subtitle: Text(customer.email),
    );
  }
}

5.3 One Widget = One Responsibility

Each widget should have one clear responsibility.

If a widget grows too large, split it into smaller widgets.

Prefer composition over inheritance or big monolithic widgets.

5.4 Pure Widgets vs Connected Widgets

Pure widgets:

Receive all data through constructor parameters.

Do not access BLoCs or state directly.

Are easy to reuse and test.

Connected widgets:

Access BLoCs/ViewModels via BlocBuilder, BlocListener, etc.

Are responsible for wiring state to the UI.

Try to keep them thin and delegate rendering to pure widgets.

5.5 Shared vs Feature-Specific Widgets

Create widgets in core/shared/widgets/ (or equivalent) when:

The widget is used across multiple features.

It is a generic UI component (button, card, sync bar, etc.).

Create widgets in features/{feature}/presentation/widgets/ when:

The widget is specific to that feature.

It is not reused across other modules.


---

### 📄 Parte 2 — Presentation Layer Rules (secciones 6–9)

```md
## 6. BLoC / State-Management Rules

### 6.1 Use Sealed Classes for States and Events

Example (simplified):

```dart
sealed class CustomersState {
  const CustomersState();
}

class CustomersInitial extends CustomersState {
  const CustomersInitial();
}

class CustomersLoading extends CustomersState {
  const CustomersLoading();
}

class CustomersLoaded extends CustomersState {
  final List<Customer> customers;
  const CustomersLoaded(this.customers);
}

class CustomersError extends CustomersState {
  final String message;
  const CustomersError(this.message);
}

6.2 Exhaustive Pattern Matching

Always handle all states in UI:

BlocBuilder<CustomersBloc, CustomersState>(
  builder: (context, state) {
    return switch (state) {
      CustomersInitial() => const SizedBox.shrink(),
      CustomersLoading() => const Center(child: CircularProgressIndicator()),
      CustomersLoaded(:final customers) => CustomersList(customers: customers),
      CustomersError(:final message) => Center(child: Text('Error: $message')),
    };
  },
);

6.3 Subscription Management

BLoCs that subscribe to streams must cancel subscriptions in close():

class OrdersBloc extends Bloc<OrdersEvent, OrdersState> {
  final OrdersRepository _repository;
  StreamSubscription<List<Order>>? _subscription;

  OrdersBloc(this._repository) : super(const OrdersInitial()) {
    on<OrdersStarted>(_onStarted);
  }

  Future<void> _onStarted(
    OrdersStarted event,
    Emitter<OrdersState> emit,
  ) async {
    _subscription?.cancel();
    _subscription = _repository.watchOrders().listen(
      (orders) => add(OrdersUpdated(orders)),
    );
  }

  @override
  Future<void> close() {
    _subscription?.cancel();
    return super.close();
  }
}

6.4 BLoC File Naming

{feature}_bloc.dart – BLoC class.

{feature}_event.dart – events.

{feature}_state.dart – states.

You may either:

Use separate files under presentation/blocs/{feature}, or

Keep them in one file with part directives, as long as the structure is consistent.

7. Sync & Offline-First in the UI
7.1 Sync Status State Management

Have a dedicated sync status state manager (e.g. SyncStatusBloc).

Responsibilities:

Expose last sync date/time.

Expose number of pending items in the outbox.

Expose whether a sync is running.

7.2 Sync Widgets

Common widgets for sync (examples):

sync_status_widget – small component showing last sync & pending items.

sync_loading_view – full-screen or section-level loading view while syncing.

sync_success_view – success feedback after a completed sync.

These widgets should live in the shared widgets module and be reused across screens.

7.3 Exposing Sync to the User

The UI should:

Clearly indicate when data is being synchronized.

Show when the last successful sync occurred.

(Optionally) show how many items are pending.

Provide a manual sync trigger when appropriate (e.g. “Sync now” button).

8. Naming & Location Conventions
8.1 Screens / Pages

File name: {feature}_page.dart
Examples: customers_list_page.dart, order_details_page.dart.

Location:
features/{feature}/presentation/pages/.

8.2 Widgets

File name: {name}_widget.dart or {name}.dart
Examples: customer_card_widget.dart, order_summary_widget.dart.

Feature-specific location:
features/{feature}/presentation/widgets/.

Shared location:
core/shared/widgets/.

8.3 BLoCs / State Managers

File names: {feature}_bloc.dart, {feature}_state.dart, {feature}_event.dart.

Location:
features/{feature}/presentation/blocs/ (or state/ if using another pattern).

9. Checklist for New Features (Presentation Layer)

Before merging a feature that touches the presentation layer, verify:

 Large widgets have been split into private widgets instead of _build* methods.

 BLoCs / state managers use sealed states and events (or equivalent).

 UI switch/when expressions are exhaustive over all states.

 All stream subscriptions in BLoCs are properly cancelled in close().

 Widgets do not call repository implementations or data sources directly.

 File names and locations follow the naming conventions.

 Shared components live in the shared widgets module.

 Sync state (if applicable) is visible and understandable for the user.

Presentation layer architecture rules — UI-focused, offline-first friendly, and reusable across Flutter projects.