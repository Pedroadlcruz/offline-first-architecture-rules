---
trigger: glob
globs: lib/core/config/router/**/*.dart, lib/core/config/factories/**/*.dart
---

# Router & Factories Rules

## 1. Purpose

The **router** and **factories** configuration is responsible for:

- Centralizing creation of pages and widgets with their dependencies.
- Managing navigation using `GoRouter` (or a similar routing library).
- Decoupling the router from concrete page implementations.
- Handling dependency injection for presentation layer (BLoCs / ViewModels).
- Reacting to authentication and connectivity state changes for redirects.

---

## 2. Allowed Responsibilities

### 2.1 Factories

#### PageFactory

- Create **full pages** with all required dependencies.
- Inject BLoCs/ViewModels using a **service locator / DI container**.
- Dispatch initial events when needed (e.g. `LoadData`).
- Wrap pages with layout/navigation (bottom bar, shell routes, etc.).
- Handle route parameters (path and query parameters).

#### WidgetFactory

- Create **reusable widgets** that require dependencies.
- Create **modal widgets** with their own BLoCs.
- Configure widgets that need specific setup or callbacks.
- Centralize creation of **complex widgets** to keep pages thin.

---

### 2.2 Router

#### AppRouter

- Main configuration of `GoRouter`.
- Defines **all application routes**.
- Configures **redirects and guards** (auth, connectivity, etc.).
- Integrates with a **refresh notifier** to react to auth/network changes.

#### AppNavigator

- Static helper methods for navigation.
- Route-specific methods:
  - `goToLogin`, `goToDashboard`, `goToDetail`, etc.
- Generic helpers:
  - `goTo`, `pushTo`, etc.
- Handle route parameters (path + query) consistently.

#### AppRoutes

- Central **enum (or class)** for route definitions.
- Holds path and name for each route.
- Lists of:
  - **Protected routes** (require authentication).
  - **Bottom navigation routes**.
- Groups related routes (e.g. wizard steps).

#### AppRouterRefreshNotifier

- `Listenable`/`ChangeNotifier` to notify router about state changes.
- Listens to:
  - **Authentication changes**.
  - **Connectivity changes**.
- Triggers router refresh to apply redirects.

---

## 3. Forbidden Responsibilities

The router and factories **must not**:

- Contain **business logic** (belongs in BLoCs/use cases).
- Perform **complex validation** (only basic route-level checks).
- Access **data sources** or repositories directly.
- Manage **UI state** beyond navigation concerns.

---

## 4. Dependency Rules

### 4.1 Allowed Dependencies

Factories may depend on:

- The **service locator / DI container**.
- BLoCs / ViewModels of the **presentation** layer.
- Pages and widgets from the **presentation** layer.
- `AppNavigator` for navigation callbacks.

Router components may depend on:

- `PageFactory` for page creation.
- `AppRoutes` for route definitions.
- `AppRouterRefreshNotifier` for refresh logic.
- The **service locator** to obtain the refresh notifier.

### 4.2 Forbidden Dependencies

Factories and router components must **not** depend on:

- Data sources or repository implementations.
- Use cases directly (use cases are consumed by BLoCs/ViewModels).
- Domain or data layer implementation details.

---

## 5. Factory Patterns

### 5.1 PageFactory Pattern

```dart
/// Factory responsible for creating pages with their required dependencies.
/// It centralizes page creation and decouples router from concrete widgets.
class PageFactory {
  /// Wraps page into the common app layout (nav bar, etc.).
  static Widget _createPageWithNavigation({
    required Widget page,
    bool showNavigationBar = true,
  }) {
    // Example wrapper – adapt to your layout
    return Scaffold(
      body: page,
      bottomNavigationBar: showNavigationBar ? const CustomBottomNavBar() : null,
    );
  }

  /// Creates a page with its BLoC and dispatches initial event.
  static Widget createSomePage() {
    return _createPageWithNavigation(
      page: BlocProvider(
        create: (_) => sl<SomeBloc>()..add(const SomeEvent.load()),
        child: const SomePage(),
      ),
    );
  }

  /// Creates a detail page using path parameters.
  static Widget createDetailPage({required String id}) {
    return _createPageWithNavigation(
      showNavigationBar: false,
      page: BlocProvider(
        create: (_) => sl<DetailBloc>()..add(DetailEvent.load(id: id)),
        child: DetailPage(id: id),
      ),
    );
  }
}
```

### 5.2 WidgetFactory Pattern

```dart
/// Factory for reusable widgets that need dependencies or specific setup.
class WidgetFactory {
  /// Creates a modal widget with its BLoC.
  static Widget createModal() {
    return BlocProvider(
      create: (_) => sl<ModalBloc>()..add(const ModalEvent.load()),
      child: const ModalWidget(),
    );
  }

  /// Creates a widget with parameters and callbacks.
  static Widget createCustomWidget({
    required void Function(String) onAction,
    required String currentValue,
  }) {
    return BlocProvider(
      create: (_) => sl<CustomWidgetBloc>(),
      child: CustomWidget(
        onAction: onAction,
        currentValue: currentValue,
      ),
    );
  }
}
```