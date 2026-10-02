# Navigation Reference

## Default: GoRouter

Use GoRouter for all projects unless the project is explicitly a simple
single-screen or demo app with no deep linking needs.

---

## Decision Guide

| Scenario | Recommendation |
|---|---|
| Most apps (multi-screen, tabs, deep links) | **GoRouter** ✅ Default |
| Simple 2–3 screen app, no deep links | `Navigator.push` is acceptable |
| Complex nested navigators + auth guards | **GoRouter** with `redirect` |
| Full-screen modals, bottom sheets | `showModalBottomSheet` / `showDialog` regardless of router |

---

## Route Constants — Core Principle

**Never** write raw path strings anywhere except in the constants files.  
**Never** call `context.go('/profile/123')` — always use typed path builders.

Every feature owns its route constants. When feature A navigates to a route
from feature B, it **duplicates** the relevant constant into its own file —
it never imports from feature B's route file. This eliminates cross-feature
coupling and makes refactoring safe.

---

## Constants Structure (per feature)

```
lib/
├── core/
│   └── router/
│       ├── app_router.dart
│       └── route_path_constants.dart   # Root-level path segments only
└── features/
    ├── auth/
    │   └── router/
    │       └── auth_route_constants.dart
    ├── home/
    │   └── router/
    │       └── home_route_constants.dart
    └── profile/
        └── router/
            └── profile_route_constants.dart
```

---

## Three Constant Classes per Feature

Each feature's route file contains exactly these three classes:

```dart
// features/profile/router/profile_route_constants.dart

/// Path segments — used only in GoRoute(path: ...) declarations
class ProfileRoutePaths {
  ProfileRoutePaths._();

  static const root = '/profile';
  static const detail = 'detail';         // relative: child of /profile
  static const edit = 'edit';             // relative: child of /profile/:userId
  static const userIdParam = ':userId';   // segment with param
}

/// Parameter keys — used to read from GoRouterState
class ProfileRouteParams {
  ProfileRouteParams._();

  static const userId = 'userId';
  static const tab = 'tab';
}

/// Full absolute paths + typed builders — used to navigate
class ProfileRouteLocations {
  ProfileRouteLocations._();

  static const root = '/profile';

  /// /profile/:userId
  static String detail(String userId) => '/profile/$userId';

  /// /profile/:userId/edit
  static String edit(String userId) => '/profile/$userId/edit';
}
```

---

## Cross-Feature Navigation Without Coupling

Feature A navigates to a profile screen. Instead of importing
`ProfileRouteLocations`, it declares its own local copy of just the
paths it needs.

```dart
// features/home/router/home_route_constants.dart

class HomeRoutePaths {
  HomeRoutePaths._();

  static const root = '/';
  static const feed = 'feed';
}

class HomeRouteParams {
  HomeRouteParams._();
  // none yet
}

class HomeRouteLocations {
  HomeRouteLocations._();

  static const root = '/';
  static const feed = '/feed';

  // Duplicated from ProfileRouteLocations — intentional, avoids coupling.
  // If profile's path ever changes, update here too (single grep, safe).
  static String profile(String userId) => '/profile/$userId';
}
```

Then in a widget inside the home feature:
```dart
// ✅ Uses home feature's own constants — no import from profile
context.go(HomeRouteLocations.profile(user.id));

// ❌ Never do this — creates cross-feature dependency
context.go(ProfileRouteLocations.detail(user.id));
```

---

## app_router.dart — All paths via constants

```dart
// core/router/app_router.dart
import 'package:go_router/go_router.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../../features/auth/application/auth_notifier.dart';
import '../../features/auth/presentation/screens/login_screen.dart';
import '../../features/auth/router/auth_route_constants.dart';
import '../../features/home/presentation/screens/home_screen.dart';
import '../../features/home/router/home_route_constants.dart';
import '../../features/profile/presentation/screens/profile_screen.dart';
import '../../features/profile/presentation/screens/profile_edit_screen.dart';
import '../../features/profile/router/profile_route_constants.dart';
import '../widgets/error/app_error_widget.dart';

part 'app_router.g.dart';

@riverpod
GoRouter appRouter(AppRouterRef ref) {
  final authState = ref.watch(authNotifierProvider);

  return GoRouter(
    initialLocation: HomeRouteLocations.root,
    debugLogDiagnostics: true,
    redirect: (context, state) {
      final isAuthenticated = authState.isAuthenticated;
      final isOnLogin =
          state.matchedLocation == AuthRouteLocations.login;

      if (!isAuthenticated && !isOnLogin) return AuthRouteLocations.login;
      if (isAuthenticated && isOnLogin) return HomeRouteLocations.root;
      return null;
    },
    errorBuilder: (context, state) =>
        AppErrorWidget(message: state.error.toString()),
    routes: [
      GoRoute(
        path: AuthRoutePaths.login,
        builder: (context, state) => const LoginScreen(),
      ),
      GoRoute(
        path: HomeRoutePaths.root,
        builder: (context, state) => const HomeScreen(),
      ),
      GoRoute(
        path: ProfileRoutePaths.root,
        builder: (context, state) {
          // ✅ Param key from constants — never a raw string
          final userId =
              state.pathParameters[ProfileRouteParams.userId]!;
          return ProfileScreen(userId: userId);
        },
        routes: [
          GoRoute(
            path: ProfileRoutePaths.edit,
            builder: (context, state) {
              final userId =
                  state.pathParameters[ProfileRouteParams.userId]!;
              return ProfileEditScreen(userId: userId);
            },
          ),
        ],
      ),
    ],
  );
}
```

---

## Navigation Patterns — All Typed

```dart
// ✅ Navigate to a path — always via RouteLocations
context.go(HomeRouteLocations.root);
context.go(HomeRouteLocations.profile(user.id));
context.push(HomeRouteLocations.profile(user.id));

// ✅ Pop back
context.pop();
context.canPop() ? context.pop() : context.go(HomeRouteLocations.root);

// ✅ Navigate with extra (passing objects, no URL param)
context.push(
  HomeRouteLocations.profile(user.id),
  extra: user,
);

// ❌ Never write raw strings
context.go('/profile/123');          // forbidden
context.go('/profile/${user.id}');   // forbidden
GoRoute(path: '/profile/:userId');   // forbidden — use ProfileRoutePaths.root
state.pathParameters['userId'];      // forbidden — use ProfileRouteParams.userId
```

---

## Auth Route Constants (example)

```dart
// features/auth/router/auth_route_constants.dart

class AuthRoutePaths {
  AuthRoutePaths._();

  static const login = '/login';
  static const register = '/register';
  static const forgotPassword = '/forgot-password';
}

class AuthRouteParams {
  AuthRouteParams._();
  // none
}

class AuthRouteLocations {
  AuthRouteLocations._();

  static const login = '/login';
  static const register = '/register';
  static const forgotPassword = '/forgot-password';
}
```

---

## Nested Navigation (Tab Bars) — with constants

```dart
// features/main/router/main_route_constants.dart
class MainRoutePaths {
  MainRoutePaths._();

  static const root = '/main';
  static const feed = 'feed';
  static const explore = 'explore';
  static const profile = 'profile';
}

class MainRouteLocations {
  MainRouteLocations._();

  static const root = '/main';
  static const feed = '/main/feed';
  static const explore = '/main/explore';
  static const profile = '/main/profile';
}
```

```dart
// In app_router.dart
GoRoute(
  path: MainRoutePaths.root,
  builder: (context, state) => const MainShell(),
  routes: [
    GoRoute(
      path: MainRoutePaths.feed,
      builder: (context, state) => const FeedScreen(),
    ),
    GoRoute(
      path: MainRoutePaths.explore,
      builder: (context, state) => const ExploreScreen(),
    ),
    GoRoute(
      path: MainRoutePaths.profile,
      builder: (context, state) => const MainProfileScreen(),
    ),
  ],
),
```

Use `StatefulShellRoute` for tab bars that preserve scroll state and
navigation stack per tab (available in GoRouter 10+).

---

## Auth Guard Pattern

```dart
redirect: (context, state) {
  final isAuthenticated = ref.read(authNotifierProvider).isAuthenticated;
  final protectedPrefixes = [
    HomeRouteLocations.root,
    MainRouteLocations.root,
  ];
  final isProtected = protectedPrefixes.any(
    (r) => state.matchedLocation.startsWith(r),
  );

  if (!isAuthenticated && isProtected) {
    // ✅ Query param key also via constant if used repeatedly
    return '${AuthRouteLocations.login}?redirect=${state.uri}';
  }
  return null;
},
```

---

## Summary: Rules at a Glance

| What | Rule |
|---|---|
| Path segments in `GoRoute(path:)` | Always `XxxRoutePaths.yyy` |
| `state.pathParameters[...]` keys | Always `XxxRouteParams.yyy` |
| `context.go(...)` / `context.push(...)` | Always `XxxRouteLocations.yyy(...)` |
| Cross-feature navigation | Duplicate the location into caller's own constants file |
| Raw strings like `'/profile/$id'` | **Forbidden everywhere** |
