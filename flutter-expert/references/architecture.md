# Architecture Reference

## Full Project Structure

```
my_app/
├── lib/
│   ├── main.dart                    # Entry point — calls runApp with ProviderScope
│   ├── main_dev.dart                # DEV entry point
│   ├── main_prod.dart               # PROD entry point
│   ├── app/
│   │   ├── app.dart                 # MaterialApp.router + GoRouter config
│   │   └── router/
│   │       └── app_router.dart
│   ├── core/
│   │   ├── config/
│   │   │   ├── app_config.dart      # Environment values loaded from .env
│   │   │   └── feature_flags.dart   # Feature flag definitions
│   │   ├── extensions/
│   │   │   ├── context_extensions.dart
│   │   │   └── spacing_extensions.dart
│   │   ├── theme/
│   │   │   ├── app_theme.dart
│   │   │   ├── app_color_theme.dart
│   │   │   └── app_text_theme.dart
│   │   └── network/
│   │       └── dio_client.dart
│   └── features/
│       └── [feature_name]/          # See SKILL.md for feature layout
│           ├── data/
│           ├── domain/
│           ├── application/
│           └── presentation/
├── packages/
│   └── widgetbook_app/              # Component library + Widgetbook preview
│       ├── pubspec.yaml
│       └── lib/
│           ├── main.dart            # Widgetbook entry point
│           ├── components/          # ← Reusable widgets live here
│           │   ├── buttons/
│           │   │   └── app_button.dart
│           │   ├── inputs/
│           │   │   └── app_text_field.dart
│           │   ├── cards/
│           │   │   └── app_card.dart
│           │   └── states/
│           │       ├── app_error_widget.dart
│           │       ├── app_empty_widget.dart
│           │       └── app_skeleton_loader.dart
│           └── stories/             # One story file per component
│               ├── buttons/
│               │   └── app_button.stories.dart
│               ├── inputs/
│               │   └── app_text_field.stories.dart
│               └── states/
│                   ├── app_error_widget.stories.dart
│                   ├── app_empty_widget.stories.dart
│                   └── app_skeleton_loader.stories.dart
├── .env.dev
├── .gitignore
└── pubspec.yaml
```

### Package dependency graph

```
my_app  ──depends on──►  widgetbook_app  (path dependency)
```

The app imports components directly from `widgetbook_app` — no separate
package needed. If a global design system is ever needed, `components/` can
be extracted into its own package at that point.

---

## Environment Configuration

### Strategy: .env only for local DEV — never on production

| Environment | Config source | How |
|---|---|---|
| Local DEV | `.env.dev` file (gitignored) | `flutter_dotenv` loads file at startup |
| Production | Platform environment variables | Injected by CI/CD (GitHub Actions, Codemagic, Fastlane, etc.) |

**Never** ship a `.env.prod` file in the repository or bundle.  
**Never** hardcode production secrets anywhere in source.  
Production builds read config via `String.fromEnvironment()` — values are
injected at compile time by the build pipeline using `--dart-define` flags.

---

### .env.dev (local only — gitignored)
```
API_BASE_URL=https://dev-api.example.com
SOME_API_KEY=dev-key-here
```

### .gitignore
```
.env.dev
.env.prod
*.env
```

### app_config.dart
```dart
import 'package:flutter/foundation.dart';
import 'package:flutter_dotenv/flutter_dotenv.dart';

enum AppEnvironment { dev, prod }

class AppConfig {
  AppConfig._();

  static late final AppEnvironment environment;
  static late final String apiBaseUrl;
  static late final String someApiKey;

  static Future<void> initialize(AppEnvironment env) async {
    environment = env;

    if (env == AppEnvironment.dev) {
      // Local dev: load from .env.dev file
      await dotenv.load(fileName: '.env.dev');
      apiBaseUrl = dotenv.env['API_BASE_URL'] ?? '';
      someApiKey = dotenv.env['SOME_API_KEY'] ?? '';
    } else {
      // Production: values injected at compile time via --dart-define
      // CI/CD passes: --dart-define=API_BASE_URL=https://api.example.com
      apiBaseUrl = const String.fromEnvironment('API_BASE_URL');
      someApiKey = const String.fromEnvironment('SOME_API_KEY');

      assert(
        apiBaseUrl.isNotEmpty,
        'API_BASE_URL must be set via --dart-define on production builds.',
      );
    }
  }

  static bool get isDev => environment == AppEnvironment.dev;
  static bool get isProd => environment == AppEnvironment.prod;
}
```

### main_dev.dart
```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'core/config/app_config.dart';
import 'app/app.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await AppConfig.initialize(AppEnvironment.dev);
  runApp(const ProviderScope(child: App()));
}
```

### main_prod.dart
```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'core/config/app_config.dart';
import 'app/app.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  // No .env file loaded — values come from --dart-define at build time
  await AppConfig.initialize(AppEnvironment.prod);
  runApp(const ProviderScope(child: App()));
}
```

### pubspec.yaml — only dotenv for dev support
```yaml
dependencies:
  flutter_dotenv: ^5.1.0   # Only used in DEV builds

flutter:
  assets:
    - .env.dev              # Only .env.dev bundled — never .env.prod
```

---

### CI/CD: How to inject production values

**GitHub Actions example:**
```yaml
- name: Build production APK
  run: |
    flutter build apk \
      --release \
      --target lib/main_prod.dart \
      --dart-define=API_BASE_URL=${{ secrets.API_BASE_URL }} \
      --dart-define=SOME_API_KEY=${{ secrets.SOME_API_KEY }}
```

**Codemagic / Fastlane:** Pass `--dart-define` flags the same way,
pulling values from the CI environment's secrets/vault.

**Rule:** Secrets live in the CI/CD secret store (GitHub Secrets, Codemagic
environment variables, etc.) — never in the repository.

---

## Feature Flags

### feature_flags.dart
```dart
import 'app_config.dart';

class FeatureFlags {
  FeatureFlags._();

  /// Enable only in dev builds for testing
  static bool get showDebugBanner => AppConfig.isDev;

  /// Example: new checkout flow behind a flag
  static bool get newCheckoutFlowEnabled =>
      AppConfig.isDev; // flip to true in prod when ready

  /// Example: premium feature
  static bool get premiumAnalyticsEnabled => AppConfig.isProd;
}
```

### Usage in widgets
```dart
if (FeatureFlags.newCheckoutFlowEnabled)
  const NewCheckoutScreen()
else
  const LegacyCheckoutScreen(),
```

---

## Standard pubspec.yaml

```yaml
name: my_app
description: My Flutter App
version: 1.0.0+1

environment:
  sdk: ">=3.0.0 <4.0.0"
  flutter: ">=3.10.0"

dependencies:
  flutter:
    sdk: flutter

  # State management
  flutter_riverpod: ^2.5.1
  riverpod_annotation: ^2.3.5

  # Navigation
  go_router: ^13.2.0

  # Networking
  dio: ^5.4.3

  # Environment
  flutter_dotenv: ^5.1.0   # Only used in DEV builds

  # Responsive sizing (only add when responsive design is confirmed)
  # flutter_screenutil: ^5.9.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.0
  riverpod_generator: ^2.3.11
  build_runner: ^2.4.9
  custom_lint: ^0.6.4
  riverpod_lint: ^2.3.10
```
