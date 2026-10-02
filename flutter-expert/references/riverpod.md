# Riverpod Reference

## Rules

- **Always** use `flutter_riverpod` + `riverpod_annotation` with code generation
- **Never** use legacy `Provider` package, `ChangeNotifierProvider`, or `GetX`
- Use `@riverpod` annotation for all providers
- Run `dart run build_runner watch` during development
- **Notifiers are thin** — they manage state transitions only; business logic lives in Use Cases
- Wire every dependency (data source, repository, use case) through a dedicated Riverpod provider

## setState vs. Riverpod — Decision Rule

```dart
// ✅ setState is fine: purely local, single-widget UI state
class PasswordField extends StatefulWidget {
  const PasswordField({super.key, required this.controller});
  final TextEditingController controller;

  @override
  State<PasswordField> createState() => _PasswordFieldState();
}

class _PasswordFieldState extends State<PasswordField> {
  bool _obscure = true;

  @override
  Widget build(BuildContext context) {
    return TextField(
      controller: widget.controller,
      obscureText: _obscure,
      decoration: InputDecoration(
        suffixIcon: IconButton(
          icon: Icon(_obscure ? Icons.visibility_off : Icons.visibility),
          onPressed: () => setState(() => _obscure = !_obscure),
        ),
      ),
    );
  }
}

// ✅ Also fine with setState: local tab index, accordion expanded state,
//    checkbox within a self-contained form widget, animation toggle

// ❌ Needs Riverpod: auth state, user profile, cart, theme mode,
//    anything fetched from network, anything shared across screens
```

---

## Provider Types

| Type | Use when |
|---|---|
| `@riverpod` (functional) | Simple derived/computed values, no mutation |
| `@riverpod` class (Notifier) | Mutable state with business logic |
| `@riverpod` async (AsyncNotifier) | Async mutable state (lists, user data) |
| `FutureProvider` | Read-only async data, no mutation needed |
| `StreamProvider` | Real-time streams (Firestore, WebSocket) |

---

## Functional Provider (read-only)

```dart
// user_repository_provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../domain/repositories/i_user_repository.dart';
import '../data/repositories/user_repository.dart';

part 'user_repository_provider.g.dart';

@riverpod
IUserRepository userRepository(UserRepositoryRef ref) {
  return UserRepository(ref.watch(dioClientProvider));
}
```

---

## Separation of Concerns in Notifiers

A Notifier's job is **only** to manage state transitions and delegate to Use Cases.
It must not contain filtering logic, validation, API calls, or mapping.

```dart
// ❌ WRONG — business logic inside the Notifier
class UserListNotifier extends _$UserListNotifier {
  Future<void> deleteUser(String userId) async {
    // Notifier is doing repository work directly — violates SoC
    final dio = ref.read(dioClientProvider);
    await dio.delete('/users/$userId');
    final updated = state.value!.where((u) => u.id != userId).toList();
    state = AsyncData(updated);
  }
}

// ✅ CORRECT — Notifier delegates to Use Case
class UserListNotifier extends _$UserListNotifier {
  Future<void> deleteUser(String userId) async {
    // Only state transition logic here
    await ref.read(deleteUserUseCaseProvider).execute(userId);
    ref.invalidateSelf(); // triggers rebuild via build()
  }
}
```

---

## Async Notifier (mutable async state)

```dart
// user_list_notifier.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../domain/entities/user.dart';
import '../domain/use_cases/fetch_users_use_case.dart';

part 'user_list_notifier.g.dart';

@riverpod
class UserListNotifier extends _$UserListNotifier {
  @override
  Future<List<User>> build() async {
    return _fetchUsers();
  }

  Future<List<User>> _fetchUsers() {
    final useCase = ref.read(fetchUsersUseCaseProvider);
    return useCase.execute();
  }

  Future<void> refresh() async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(_fetchUsers);
  }

  Future<void> deleteUser(String userId) async {
    final useCase = ref.read(deleteUserUseCaseProvider);
    await useCase.execute(userId);
    ref.invalidateSelf();
  }
}
```

---

## State Class Pattern

For complex state, use a plain Dart class (no Freezed per minimal setup):

```dart
// auth_state.dart
class AuthState {
  const AuthState({
    this.isLoading = false,
    this.user,
    this.error,
  });

  final bool isLoading;
  final User? user;
  final String? error;

  bool get isAuthenticated => user != null;

  AuthState copyWith({
    bool? isLoading,
    User? user,
    String? error,
  }) {
    return AuthState(
      isLoading: isLoading ?? this.isLoading,
      user: user ?? this.user,
      error: error ?? this.error,
    );
  }
}
```

```dart
// auth_notifier.dart
@riverpod
class AuthNotifier extends _$AuthNotifier {
  @override
  AuthState build() => const AuthState();

  Future<void> signIn(String email, String password) async {
    state = state.copyWith(isLoading: true, error: null);
    try {
      final user = await ref.read(signInUseCaseProvider).execute(email, password);
      state = state.copyWith(isLoading: false, user: user);
    } catch (e) {
      state = state.copyWith(isLoading: false, error: e.toString());
    }
  }

  void signOut() {
    state = const AuthState();
  }
}
```

---

## Consuming Providers in Widgets

```dart
// Always use ConsumerWidget or ConsumerStatefulWidget — never StatefulWidget + ref
class UserListScreen extends ConsumerWidget {
  const UserListScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final usersAsync = ref.watch(userListNotifierProvider);

    return usersAsync.when(
      loading: () => const UserListSkeleton(),
      error: (error, _) => AppErrorWidget(message: error.toString()),
      data: (users) {
        if (users.isEmpty) return const AppEmptyWidget(message: 'No users found.');
        return ListView.builder(
          itemCount: users.length,
          itemBuilder: (context, index) => UserCard(user: users[index]),
        );
      },
    );
  }
}
```

---

## Provider Scoping & Overrides

```dart
// Override for testing or feature-specific instances
ProviderScope(
  overrides: [
    userRepositoryProvider.overrideWithValue(MockUserRepository()),
  ],
  child: const App(),
)
```

---

## Dependency Injection via Riverpod

```dart
// dio_client_provider.dart
@riverpod
Dio dioClient(DioClientRef ref) {
  final dio = Dio(BaseOptions(baseUrl: AppConfig.apiBaseUrl));
  dio.interceptors.add(LogInterceptor(responseBody: true));
  return dio;
}
```

All dependencies (Dio, repositories, use cases) are wired through Riverpod —
never use a service locator like `get_it`.
