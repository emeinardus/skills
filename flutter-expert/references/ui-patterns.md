# UI Patterns Reference

## Widget Rules

- **Always** extract widgets into their own files — never private `_build...()` methods
- Keep every widget file under 250 lines (hard max: 400)
- Every async-driven screen must handle: loading, error, and empty state
- Prefer `ConsumerWidget` over `StatelessWidget` when accessing Riverpod
- Use `ConsumerStatefulWidget` only when local lifecycle (`initState`, animation controllers) is needed

---

## File Extraction Rule (Enforced)

```dart
// ❌ NEVER do this
class UserScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        _buildHeader(),  // ← private build method — forbidden
        _buildList(),
      ],
    );
  }

  Widget _buildHeader() => const Text('Header');
  Widget _buildList() => const Text('List');
}

// ✅ ALWAYS do this — extract to separate files
// user_screen.dart
class UserScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return const Column(
      children: [
        UserHeader(),   // ← separate file: user_header.dart
        UserList(),     // ← separate file: user_list.dart
      ],
    );
  }
}
```

---

## Loading / Error / Empty States

### Shared widgets (packages/widgetbook_app/lib/components/states/)

These widgets live in `widgetbook_app/lib/components/` — not in `lib/core/`.
Import them via `package:widgetbook_app/components/states/app_error_widget.dart`.

```dart
// packages/widgetbook_app/lib/components/states/app_error_widget.dart
class AppErrorWidget extends StatelessWidget {
  const AppErrorWidget({
    super.key,
    required this.message,
    this.onRetry,
  });

  final String message;
  final VoidCallback? onRetry;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          Icon(Icons.error_outline,
              size: 48, color: context.appColorTheme.error),
          16.verticalSpacing,
          Text(message,
              style: context.appTextTheme.bodyMedium,
              textAlign: TextAlign.center),
          if (onRetry != null) ...[
            16.verticalSpacing,
            ElevatedButton(onPressed: onRetry, child: const Text('Retry')),
          ],
        ],
      ),
    );
  }
}
```

```dart
// packages/widgetbook_app/lib/components/states/app_empty_widget.dart
class AppEmptyWidget extends StatelessWidget {
  const AppEmptyWidget({
    super.key,
    required this.message,
    this.icon = Icons.inbox_outlined,
    this.action,
  });

  final String message;
  final IconData icon;
  final Widget? action;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          Icon(icon, size: 64, color: context.appColorTheme.textDisabled),
          16.verticalSpacing,
          Text(message,
              style: context.appTextTheme.bodyMedium
                  .copyWith(color: context.appColorTheme.textSecondary),
              textAlign: TextAlign.center),
          if (action != null) ...[16.verticalSpacing, action!],
        ],
      ),
    );
  }
}
```

---

## Skeleton Loader

Use skeleton loaders when content loads from a remote source and has a
predictable layout shape. Prefer skeleton over a centered spinner for
list/card-heavy screens.

```dart
// packages/widgetbook_app/lib/components/states/app_skeleton_loader.dart
import 'package:flutter/material.dart';

class AppSkeletonLoader extends StatefulWidget {
  const AppSkeletonLoader({
    super.key,
    required this.width,
    required this.height,
    this.borderRadius = 8,
  });

  final double width;
  final double height;
  final double borderRadius;

  @override
  State<AppSkeletonLoader> createState() => _AppSkeletonLoaderState();
}

class _AppSkeletonLoaderState extends State<AppSkeletonLoader>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 1200),
    )..repeat(reverse: true);
    _animation = Tween<double>(begin: 0.4, end: 1.0).animate(_controller);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return FadeTransition(
      opacity: _animation,
      child: Container(
        width: widget.width,
        height: widget.height,
        decoration: BoxDecoration(
          color: context.appColorTheme.skeleton,
          borderRadius: BorderRadius.circular(widget.borderRadius),
        ),
      ),
    );
  }
}
```

### Skeleton usage in a list
```dart
// user_list_skeleton.dart
class UserListSkeleton extends StatelessWidget {
  const UserListSkeleton({super.key});

  @override
  Widget build(BuildContext context) {
    return ListView.separated(
      padding: const EdgeInsets.all(16),
      itemCount: 6,
      separatorBuilder: (_, __) => 12.verticalSpacing,
      itemBuilder: (_, __) => const UserCardSkeleton(),
    );
  }
}

// user_card_skeleton.dart
class UserCardSkeleton extends StatelessWidget {
  const UserCardSkeleton({super.key});

  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        AppSkeletonLoader(width: 48, height: 48, borderRadius: 24),
        12.horizontalSpacing,
        Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            AppSkeletonLoader(width: 140, height: 14),
            6.verticalSpacing,
            AppSkeletonLoader(width: 90, height: 12),
          ],
        ),
      ],
    );
  }
}
```

---

## Platform Adaptation

### Material (Android) + Cupertino (iOS)

```dart
import 'dart:io';

class AdaptiveButton extends StatelessWidget {
  const AdaptiveButton({
    super.key,
    required this.label,
    required this.onPressed,
  });

  final String label;
  final VoidCallback onPressed;

  @override
  Widget build(BuildContext context) {
    if (Platform.isIOS) {
      return CupertinoButton.filled(onPressed: onPressed, child: Text(label));
    }
    return ElevatedButton(onPressed: onPressed, child: Text(label));
  }
}
```

### Desktop (macOS, Windows, Linux) Best Practices
- Use `MouseRegion` for hover states
- Add keyboard shortcuts via `Shortcuts` + `Actions` widgets
- Use `LayoutBuilder` to adapt layout to available width
- Avoid touch-optimized minimum tap target sizes (48px) — desktop can be smaller (32px)
- Provide right-click context menus where appropriate

### Web Best Practices
- Add `SelectableText` instead of `Text` for copyable content
- Use `ScrollConfiguration` to match platform scroll physics
- Avoid `Platform.isIOS` on web — use `kIsWeb` from `flutter/foundation.dart`
- Provide loading indicators that suit web UX expectations

---

## Lazy Loading Lists

```dart
class LazyUserList extends ConsumerStatefulWidget {
  const LazyUserList({super.key});

  @override
  ConsumerState<LazyUserList> createState() => _LazyUserListState();
}

class _LazyUserListState extends ConsumerState<LazyUserList> {
  final _scrollController = ScrollController();

  @override
  void initState() {
    super.initState();
    _scrollController.addListener(_onScroll);
  }

  void _onScroll() {
    if (_scrollController.position.pixels >=
        _scrollController.position.maxScrollExtent - 200) {
      ref.read(userListNotifierProvider.notifier).loadMore();
    }
  }

  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final usersAsync = ref.watch(userListNotifierProvider);

    return usersAsync.when(
      loading: () => const UserListSkeleton(),
      error: (e, _) => AppErrorWidget(message: e.toString()),
      data: (state) => ListView.builder(
        controller: _scrollController,
        itemCount: state.users.length + (state.isLoadingMore ? 1 : 0),
        itemBuilder: (context, index) {
          if (index == state.users.length) {
            return const Center(child: CircularProgressIndicator());
          }
          return UserCard(user: state.users[index]);
        },
      ),
    );
  }
}
```
