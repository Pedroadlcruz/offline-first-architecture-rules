---
trigger: glob
globs: lib/core/config/router/**/*.dart, lib/core/config/factories/**/*.dart
---

6. Router Patterns
6.1 AppRoutes Enum
/// Central enum for all application routes.
/// Defines paths, names, and route groups.
enum AppRoute {
  login('/login', 'login'),
  dashboard('/dashboard', 'dashboard'),
  detail('/detail/:id', 'detail'),
  noNetwork('/nonetwork', 'no_network');

  const AppRoute(this.path, this.name);

  final String path;
  final String name;

  /// Routes that require authentication.
  static const List<AppRoute> protectedRoutes = [
    AppRoute.dashboard,
    AppRoute.detail,
  ];

  /// Routes that show the bottom navigation bar.
  static const List<AppRoute> bottomNavRoutes = [
    AppRoute.dashboard,
  ];
}

6.2 AppRouter (GoRouter Configuration)
/// Main router configuration using GoRouter.
/// Centralizes all route definitions and redirect logic.
class AppRouter {
  static final _rootNavigatorKey = GlobalKey<NavigatorState>();

  static final GoRouter router = GoRouter(
    initialLocation: AppRoute.login.path,
    navigatorKey: _rootNavigatorKey,
    refreshListenable: sl<AppRouterRefreshNotifier>(),
    redirect: (context, state) {
      final notifier = sl<AppRouterRefreshNotifier>();

      // Redirect to /nonetwork when offline
      if (!notifier.isOnline) {
        return AppRoute.noNetwork.path;
      }

      // Check if the current location requires authentication
      final isProtected = AppRoute.protectedRoutes.any(
        (route) => state.matchedLocation.startsWith(route.path),
      );

      // Redirect unauthenticated users away from protected routes
      if (!notifier.isAuthenticated && isProtected) {
        return AppRoute.login.path;
      }

      // Redirect authenticated users away from auth routes
      if (notifier.isAuthenticated &&
          state.matchedLocation == AppRoute.login.path) {
        return AppRoute.dashboard.path;
      }

      return null;
    },
    routes: [
      GoRoute(
        path: AppRoute.login.path,
        name: AppRoute.login.name,
        builder: (context, state) => PageFactory.createLoginPage(),
      ),
      GoRoute(
        path: AppRoute.dashboard.path,
        name: AppRoute.dashboard.name,
        builder: (context, state) {
          final shouldSync =
              state.uri.queryParameters['shouldSync']?.toLowerCase() == 'true';
          return PageFactory.createDashboardPage(shouldSync: shouldSync);
        },
      ),
      GoRoute(
        path: AppRoute.detail.path,
        name: AppRoute.detail.name,
        builder: (context, state) {
          final id = state.pathParameters['id']!;
          return PageFactory.createDetailPage(id: id);
        },
      ),
    ],
  );
}

6.3 AppNavigator Helpers
/// Central navigation helpers using GoRouter.
/// Provides a clean API to navigate from anywhere in the app.
class AppNavigator {
  // Route-specific methods
  static void goToLogin(BuildContext context) =>
      context.goNamed(AppRoute.login.name);

  static void goToDashboard(BuildContext context, {bool shouldSync = false}) =>
      context.goNamed(
        AppRoute.dashboard.name,
        queryParameters: {'shouldSync': shouldSync.toString()},
      );

  static void goToDetail(BuildContext context, String id) =>
      context.goNamed(
        AppRoute.detail.name,
        pathParameters: {'id': id},
      );

  // Generic helpers
  static void goTo(BuildContext context, AppRoute route) =>
      context.goNamed(route.name);

  static void pushTo(BuildContext context, AppRoute route) =>
      context.pushNamed(route.name);
}

6.4 AppRouterRefreshNotifier Pattern
/// Notifier that listens to auth and connectivity changes
/// and notifies GoRouter to re-evaluate redirects.
class AppRouterRefreshNotifier extends ChangeNotifier {
  final AuthRepository _authRepository;
  final NetworkInfo _networkInfo;

  StreamSubscription<User?>? _authSubscription;
  StreamSubscription<ConnectivityResult>? _connectivitySubscription;

  User? _currentUser;
  bool _isOnline = true;

  User? get currentUser => _currentUser;
  bool get isOnline => _isOnline;
  bool get isAuthenticated => _currentUser != null;

  AppRouterRefreshNotifier(
    this._authRepository,
    this._networkInfo,
  ) {
    _initListeners();
  }

  void _initListeners() {
    _authSubscription = _authRepository.authStateChanges.listen((user) {
      if (_currentUser != user) {
        _currentUser = user;
        notifyListeners();
      }
    });

    _connectivitySubscription =
        _networkInfo.onConnectivityChanged.listen((result) {
      final newOnlineStatus = _networkInfo.isOnline(result);
      if (_isOnline != newOnlineStatus) {
        _isOnline = newOnlineStatus;
        notifyListeners();
      }
    });

    _initConnectivityState();
  }

  Future<void> _initConnectivityState() async {
    try {
      _isOnline = await _networkInfo.isConnected;
      notifyListeners();
    } catch (_) {
      _isOnline = false;
      notifyListeners();
    }
  }

  @override
  void dispose() {
    _authSubscription?.cancel();
    _connectivitySubscription?.cancel();
    super.dispose();
  }
}

7. Wizard & Nested Route Patterns
7.1 ShellRoute for Wizards
// Wizard using ShellRoute to keep the same BLoC across steps.
final wizardRoutes = ShellRoute(
  builder: (context, state, child) => child,
  routes: [
    GoRoute(
      path: AppRoute.wizard.path,
      redirect: (context, state) => AppRoute.wizardStep1.path,
    ),
    GoRoute(
      path: AppRoute.wizardStep1.path,
      name: AppRoute.wizardStep1.name,
      builder: (context, state) => PageFactory.createWizardStep1(),
    ),
    GoRoute(
      path: AppRoute.wizardStep2.path,
      name: AppRoute.wizardStep2.name,
      builder: (context, state) => PageFactory.createWizardStep2(),
    ),
  ],
);

7.2 Nested Routes
GoRoute(
  path: AppRoute.parent.path,
  name: AppRoute.parent.name,
  builder: (context, state) => PageFactory.createParentPage(),
  routes: [
    GoRoute(
      path: '/:childId',
      name: 'child_detail',
      builder: (context, state) {
        final childId = state.pathParameters['childId']!;
        return PageFactory.createChildDetailPage(childId: childId);
      },
    ),
  ],
);

8. Naming & Location
8.1 Factories

page_factory.dart – PageFactory and helpers.

widget_factory.dart – WidgetFactory and helpers.

Location:

core/config/factories/

8.2 Router

app_router.dart – AppRouter with GoRouter instance.

app_navigator.dart – AppNavigator helpers.

app_routes.dart – AppRoute enum or similar.

app_router_refresh_notifier.dart – notifier class.

Location:

core/config/router/

9. Checklist for New Routes

Before adding a new route, verify:

 The route is defined in AppRoute with path and name.

 There is a corresponding factory method in PageFactory.

 BLoCs/ViewModels are properly injected in the factory.

 Initial events are dispatched when needed.

 There is a helper method in AppNavigator for navigation.

 If the route requires auth, it is added to protectedRoutes.

 If it should show the bottom navigation, it is added to bottomNavRoutes.

 Path and query parameters are handled correctly.

 The route is registered in AppRouter.routes.

10. Recommended Patterns (Short Examples)
10.1 Simple Page with BLoC
// PageFactory
static Widget createSimplePage() {
  return _createPageWithNavigation(
    page: BlocProvider(
      create: (_) => sl<SimpleBloc>()..add(const SimpleEvent.load()),
      child: const SimplePage(),
    ),
  );
}

// AppRouter
GoRoute(
  path: AppRoute.simple.path,
  name: AppRoute.simple.name,
  builder: (context, state) => PageFactory.createSimplePage(),
);

// AppNavigator
static void goToSimple(BuildContext context) =>
    context.goNamed(AppRoute.simple.name);

10.2 Page with Query Parameters
// PageFactory
static Widget createPageWithParams({required bool shouldSync}) {
  return _createPageWithNavigation(
    page: BlocProvider(
      create: (_) => sl<PageBloc>()
        ..add(PageEvent.load(shouldSync: shouldSync)),
      child: const PageWidget(),
    ),
  );
}

// AppRouter
GoRoute(
  path: AppRoute.page.path,
  name: AppRoute.page.name,
  builder: (context, state) {
    final shouldSync =
        state.uri.queryParameters['shouldSync']?.toLowerCase() == 'true';
    return PageFactory.createPageWithParams(shouldSync: shouldSync);
  },
);

// AppNavigator
static void goToPage(BuildContext context, {bool shouldSync = false}) =>
    context.goNamed(
      AppRoute.page.name,
      queryParameters: {'shouldSync': shouldSync.toString()},
    );

10.3 Detail Page with Path Parameters
// PageFactory
static Widget createDetailPage({required String id}) {
  return _createPageWithNavigation(
    showNavigationBar: false,
    page: BlocProvider(
      create: (_) => sl<DetailBloc>()..add(DetailEvent.load(id: id)),
      child: DetailPage(id: id),
    ),
  );
}

// AppRouter
GoRoute(
  path: AppRoute.detail.path, // e.g. '/detail/:id'
  name: AppRoute.detail.name,
  builder: (context, state) {
    final id = state.pathParameters['id']!;
    return PageFactory.createDetailPage(id: id);
  },
);

// AppNavigator
static void goToDetail(BuildContext context, String id) =>
    context.goNamed(
      AppRoute.detail.name,
      pathParameters: {'id': id},
    );

10.4 Modal via WidgetFactory
// WidgetFactory
static Widget createSelectionModal({
  required void Function(String) onSelected,
  required String currentValue,
}) {
  return BlocProvider(
    create: (_) => sl<SelectionModalBloc>(),
    child: SelectionModal(
      onSelected: onSelected,
      currentValue: currentValue,
    ),
  );
}

// Usage in a page
showModalBottomSheet(
  context: context,
  builder: (_) => WidgetFactory.createSelectionModal(
    onSelected: (value) {
      // Handle selection
      Navigator.pop(context);
    },
    currentValue: currentValue,
  ),
);


Router and factories architecture rules — centralized navigation, dependency injection, and clean routing for Flutter + GoRouter apps.