# Widgetbook Reference

## Overview

`packages/widgetbook_app` serves two purposes in one:

1. **Component library** — reusable widgets live in `lib/components/`
2. **Widgetbook preview** — stories in `lib/stories/` render each component in isolation

The main app imports components directly from this package via a `path:`
dependency. No separate package needed.

---

## Structure

```
packages/widgetbook_app/
├── pubspec.yaml
└── lib/
    ├── main.dart                        # Widgetbook entry point
    ├── components/                      # Reusable widgets — imported by the app
    │   ├── buttons/
    │   │   └── app_button.dart
    │   ├── inputs/
    │   │   └── app_text_field.dart
    │   ├── cards/
    │   │   └── app_card.dart
    │   └── states/
    │       ├── app_error_widget.dart
    │       ├── app_empty_widget.dart
    │       └── app_skeleton_loader.dart
    └── stories/                         # One story file per component
        ├── buttons/
        │   └── app_button.stories.dart
        ├── inputs/
        │   └── app_text_field.stories.dart
        └── states/
            ├── app_error_widget.stories.dart
            ├── app_empty_widget.stories.dart
            └── app_skeleton_loader.stories.dart
```

---

## pubspec.yaml

```yaml
name: widgetbook_app
description: Component library and Widgetbook preview

environment:
  sdk: ">=3.0.0 <4.0.0"
  flutter: ">=3.10.0"

dependencies:
  flutter:
    sdk: flutter
  widgetbook: ^3.7.0
  widgetbook_annotation: ^3.1.0
  # Import the main app for theme access
  my_app:
    path: ../../

dev_dependencies:
  build_runner: ^2.4.9
  widgetbook_generator: ^3.7.0
```

## my_app/pubspec.yaml — importing components

```yaml
dependencies:
  widgetbook_app:
    path: packages/widgetbook_app
```

### Usage in app code

```dart
import 'package:widgetbook_app/components/states/app_error_widget.dart';
import 'package:widgetbook_app/components/buttons/app_button.dart';
```

---

## main.dart

```dart
import 'package:flutter/material.dart';
import 'package:widgetbook/widgetbook.dart';
import 'package:widgetbook_annotation/widgetbook_annotation.dart' as widgetbook;
import 'package:my_app/core/theme/app_theme.dart';
import 'main.directories.g.dart';

void main() => runApp(const WidgetbookApp());

@widgetbook.App()
class WidgetbookApp extends StatelessWidget {
  const WidgetbookApp({super.key});

  @override
  Widget build(BuildContext context) {
    return Widgetbook.material(
      directories: directories,
      addons: [
        MaterialThemeAddon(
          themes: [
            WidgetbookTheme(name: 'Light', data: AppTheme.light()),
            WidgetbookTheme(name: 'Dark', data: AppTheme.dark()),
          ],
        ),
        DeviceFrameAddon(
          devices: [
            Devices.ios.iPhone13,
            Devices.android.samsungGalaxyS20,
            Devices.ios.iPad,
          ],
        ),
        TextScaleAddon(min: 0.75, max: 2.0),
      ],
    );
  }
}
```

---

## Writing Components

Components in `lib/components/` follow the same widget rules as the rest of
the project (file size, no `_build` methods, etc.) but have no feature-specific
logic or Riverpod providers.

```dart
// lib/components/states/app_error_widget.dart
import 'package:flutter/material.dart';
import 'package:my_app/core/extensions/context_extensions.dart';
import 'package:my_app/core/extensions/spacing_extensions.dart';

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
          Icon(Icons.error_outline, size: 48, color: context.appColorTheme.error),
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

---

## Writing Stories

```dart
// lib/stories/states/app_error_widget.stories.dart
import 'package:flutter/material.dart';
import 'package:widgetbook_annotation/widgetbook_annotation.dart' as widgetbook;
import '../../components/states/app_error_widget.dart';

@widgetbook.UseCase(name: 'With retry', type: AppErrorWidget)
Widget errorWithRetry(BuildContext context) {
  return AppErrorWidget(
    message: context.knobs.string(
      label: 'Message',
      initialValue: 'Something went wrong. Please try again.',
    ),
    onRetry: () {},
  );
}

@widgetbook.UseCase(name: 'Without retry', type: AppErrorWidget)
Widget errorWithoutRetry(BuildContext context) {
  return AppErrorWidget(
    message: context.knobs.string(
      label: 'Message',
      initialValue: 'An unexpected error occurred.',
    ),
  );
}
```

```dart
// lib/stories/buttons/app_button.stories.dart
import 'package:flutter/material.dart';
import 'package:widgetbook_annotation/widgetbook_annotation.dart' as widgetbook;
import '../../components/buttons/app_button.dart';

@widgetbook.UseCase(name: 'Primary', type: AppButton)
Widget primaryButton(BuildContext context) {
  return Center(
    child: AppButton(
      label: context.knobs.string(label: 'Label', initialValue: 'Click Me'),
      onPressed: () {},
      isLoading: context.knobs.boolean(label: 'Loading', initialValue: false),
    ),
  );
}

@widgetbook.UseCase(name: 'Disabled', type: AppButton)
Widget disabledButton(BuildContext context) {
  return const Center(
    child: AppButton(label: 'Disabled', onPressed: null),
  );
}
```

---

## Running Widgetbook

```bash
# From packages/widgetbook_app/
dart run build_runner build --delete-conflicting-outputs
flutter run -d chrome    # Web — best for design review
flutter run -d macos     # Desktop
```

---

## When to Add to Widgetbook

Add a component to `lib/components/` and a story to `lib/stories/` whenever
the widget is:
- Used in more than one feature, or
- A primitive building block (button, input, card, badge, skeleton), or
- Something a designer or PM would want to preview in isolation

**Never** put feature-specific widgets here (e.g. `UserProfileCard`,
`OrderSummaryTile`) — those belong in `features/[name]/presentation/widgets/`.

| Widget type | Location |
|---|---|
| `AppButton`, `AppTextField`, `AppCard` | `packages/widgetbook_app/lib/components/` |
| `AppErrorWidget`, `AppEmptyWidget`, `AppSkeletonLoader` | `packages/widgetbook_app/lib/components/states/` |
| `UserCard`, `OrderTile`, feature-specific | `features/[name]/presentation/widgets/` |

---

## Future: Extracting to a Global Design System

If components need to be shared across multiple apps, extract `lib/components/`
into its own package at that point. The import paths in the app change from:

```dart
import 'package:widgetbook_app/components/buttons/app_button.dart';
```
to:
```dart
import 'package:ui_components/ui_components.dart';
```

No component code needs to change — only the dependency source.
