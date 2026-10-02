# Theming Reference

## Token-Based Theme System

All visual values are defined as tokens — never use hardcoded colors, font
sizes, or spacing values directly in widgets.

---

## app_color_theme.dart

```dart
import 'package:flutter/material.dart';

class AppColorTheme extends ThemeExtension<AppColorTheme> {
  const AppColorTheme({
    required this.primary,
    required this.onPrimary,
    required this.secondary,
    required this.onSecondary,
    required this.background,
    required this.surface,
    required this.surfaceVariant,
    required this.error,
    required this.onError,
    required this.success,
    required this.warning,
    required this.textPrimary,
    required this.textSecondary,
    required this.textDisabled,
    required this.divider,
    required this.skeleton,
  });

  final Color primary;
  final Color onPrimary;
  final Color secondary;
  final Color onSecondary;
  final Color background;
  final Color surface;
  final Color surfaceVariant;
  final Color error;
  final Color onError;
  final Color success;
  final Color warning;
  final Color textPrimary;
  final Color textSecondary;
  final Color textDisabled;
  final Color divider;
  final Color skeleton;

  static const light = AppColorTheme(
    primary: Color(0xFF2563EB),
    onPrimary: Color(0xFFFFFFFF),
    secondary: Color(0xFF7C3AED),
    onSecondary: Color(0xFFFFFFFF),
    background: Color(0xFFF8FAFC),
    surface: Color(0xFFFFFFFF),
    surfaceVariant: Color(0xFFF1F5F9),
    error: Color(0xFFDC2626),
    onError: Color(0xFFFFFFFF),
    success: Color(0xFF16A34A),
    warning: Color(0xFFD97706),
    textPrimary: Color(0xFF0F172A),
    textSecondary: Color(0xFF64748B),
    textDisabled: Color(0xFFCBD5E1),
    divider: Color(0xFFE2E8F0),
    skeleton: Color(0xFFE2E8F0),
  );

  static const dark = AppColorTheme(
    primary: Color(0xFF3B82F6),
    onPrimary: Color(0xFFFFFFFF),
    secondary: Color(0xFF8B5CF6),
    onSecondary: Color(0xFFFFFFFF),
    background: Color(0xFF0F172A),
    surface: Color(0xFF1E293B),
    surfaceVariant: Color(0xFF334155),
    error: Color(0xFFF87171),
    onError: Color(0xFF7F1D1D),
    success: Color(0xFF4ADE80),
    warning: Color(0xFFFBBF24),
    textPrimary: Color(0xFFF8FAFC),
    textSecondary: Color(0xFF94A3B8),
    textDisabled: Color(0xFF475569),
    divider: Color(0xFF334155),
    skeleton: Color(0xFF334155),
  );

  @override
  AppColorTheme copyWith({Color? primary, Color? onPrimary /* ... */}) {
    return AppColorTheme(
      primary: primary ?? this.primary,
      onPrimary: onPrimary ?? this.onPrimary,
      secondary: secondary,
      onSecondary: onSecondary,
      background: background,
      surface: surface,
      surfaceVariant: surfaceVariant,
      error: error,
      onError: onError,
      success: success,
      warning: warning,
      textPrimary: textPrimary,
      textSecondary: textSecondary,
      textDisabled: textDisabled,
      divider: divider,
      skeleton: skeleton,
    );
  }

  @override
  AppColorTheme lerp(AppColorTheme? other, double t) {
    if (other == null) return this;
    return AppColorTheme(
      primary: Color.lerp(primary, other.primary, t)!,
      onPrimary: Color.lerp(onPrimary, other.onPrimary, t)!,
      secondary: Color.lerp(secondary, other.secondary, t)!,
      onSecondary: Color.lerp(onSecondary, other.onSecondary, t)!,
      background: Color.lerp(background, other.background, t)!,
      surface: Color.lerp(surface, other.surface, t)!,
      surfaceVariant: Color.lerp(surfaceVariant, other.surfaceVariant, t)!,
      error: Color.lerp(error, other.error, t)!,
      onError: Color.lerp(onError, other.onError, t)!,
      success: Color.lerp(success, other.success, t)!,
      warning: Color.lerp(warning, other.warning, t)!,
      textPrimary: Color.lerp(textPrimary, other.textPrimary, t)!,
      textSecondary: Color.lerp(textSecondary, other.textSecondary, t)!,
      textDisabled: Color.lerp(textDisabled, other.textDisabled, t)!,
      divider: Color.lerp(divider, other.divider, t)!,
      skeleton: Color.lerp(skeleton, other.skeleton, t)!,
    );
  }
}
```

---

## app_text_theme.dart

```dart
import 'package:flutter/material.dart';

class AppTextTheme extends ThemeExtension<AppTextTheme> {
  const AppTextTheme({
    required this.displayLarge,
    required this.displayMedium,
    required this.headlineLarge,
    required this.headlineMedium,
    required this.headlineSmall,
    required this.bodyLarge,
    required this.bodyMedium,
    required this.bodySmall,
    required this.labelLarge,
    required this.labelMedium,
    required this.labelSmall,
  });

  final TextStyle displayLarge;
  final TextStyle displayMedium;
  final TextStyle headlineLarge;
  final TextStyle headlineMedium;
  final TextStyle headlineSmall;
  final TextStyle bodyLarge;
  final TextStyle bodyMedium;
  final TextStyle bodySmall;
  final TextStyle labelLarge;
  final TextStyle labelMedium;
  final TextStyle labelSmall;

  static const standard = AppTextTheme(
    displayLarge: TextStyle(fontSize: 57, fontWeight: FontWeight.w400, height: 1.12),
    displayMedium: TextStyle(fontSize: 45, fontWeight: FontWeight.w400, height: 1.16),
    headlineLarge: TextStyle(fontSize: 32, fontWeight: FontWeight.w700, height: 1.25),
    headlineMedium: TextStyle(fontSize: 28, fontWeight: FontWeight.w700, height: 1.29),
    headlineSmall: TextStyle(fontSize: 24, fontWeight: FontWeight.w600, height: 1.33),
    bodyLarge: TextStyle(fontSize: 16, fontWeight: FontWeight.w400, height: 1.5),
    bodyMedium: TextStyle(fontSize: 14, fontWeight: FontWeight.w400, height: 1.43),
    bodySmall: TextStyle(fontSize: 12, fontWeight: FontWeight.w400, height: 1.33),
    labelLarge: TextStyle(fontSize: 14, fontWeight: FontWeight.w600, height: 1.43),
    labelMedium: TextStyle(fontSize: 12, fontWeight: FontWeight.w600, height: 1.33),
    labelSmall: TextStyle(fontSize: 11, fontWeight: FontWeight.w600, height: 1.45),
  );

  @override
  AppTextTheme copyWith({TextStyle? displayLarge /* ... */}) => this;

  @override
  AppTextTheme lerp(AppTextTheme? other, double t) => this;
}
```

---

## app_theme.dart

```dart
import 'package:flutter/material.dart';
import 'app_color_theme.dart';
import 'app_text_theme.dart';

class AppTheme {
  AppTheme._();

  static ThemeData light() {
    return ThemeData(
      useMaterial3: true,
      colorScheme: ColorScheme.fromSeed(
        seedColor: AppColorTheme.light.primary,
        brightness: Brightness.light,
      ),
      extensions: const [AppColorTheme.light, AppTextTheme.standard],
    );
  }

  static ThemeData dark() {
    return ThemeData(
      useMaterial3: true,
      colorScheme: ColorScheme.fromSeed(
        seedColor: AppColorTheme.dark.primary,
        brightness: Brightness.dark,
      ),
      extensions: const [AppColorTheme.dark, AppTextTheme.standard],
    );
  }
}
```

---

## context_extensions.dart

```dart
import 'package:flutter/material.dart';
import '../theme/app_color_theme.dart';
import '../theme/app_text_theme.dart';

extension AppContextExtensions on BuildContext {
  AppColorTheme get appColorTheme =>
      Theme.of(this).extension<AppColorTheme>()!;

  AppTextTheme get appTextTheme =>
      Theme.of(this).extension<AppTextTheme>()!;

  ThemeData get theme => Theme.of(this);
  bool get isDarkMode => theme.brightness == Brightness.dark;

  // Note: for responsive sizing use flutter_screenutil (.w, .h, .r, .sp)
  // instead of reading MediaQuery here.
}
```

### Usage
```dart
// Instead of Theme.of(context).extension<AppColorTheme>()!.primary
Color primary = context.appColorTheme.primary;
TextStyle headline = context.appTextTheme.headlineLarge;
```

---

## sizing_extensions.dart → flutter_screenutil

Statt eigener Extensions mit `context`-Übergabe wird **`flutter_screenutil`**
verwendet. Es initialisiert sich einmalig im Widget-Tree und stellt `.w`, `.h`,
`.r`, `.sp` danach context-frei überall zur Verfügung.

### pubspec.yaml

```yaml
dependencies:
  flutter_screenutil: ^5.9.0
```

### Einmalige Initialisierung in app.dart

```dart
import 'package:flutter_screenutil/flutter_screenutil.dart';

class App extends ConsumerWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final router = ref.watch(appRouterProvider);

    return ScreenUtilInit(
      // Design-Basisgröße — an das primäre Design-Frame anpassen
      designSize: const Size(375, 812),
      minTextAdapt: true,
      splitScreenMode: true,
      builder: (context, child) {
        return MaterialApp.router(
          title: 'My App',
          theme: AppTheme.light(),
          darkTheme: AppTheme.dark(),
          themeMode: ThemeMode.system,
          routerConfig: router,
        );
      },
    );
  }
}
```

### Die `sizing_extensions.dart` entfällt vollständig

`flutter_screenutil` stellt die Extensions direkt auf `num` bereit — keine
eigene Datei nötig.

### Usage

```dart
import 'package:flutter_screenutil/flutter_screenutil.dart';

Container(
  width: 80.w,              // 80% der Design-Breite, skaliert
  height: 20.h,             // 20% der Design-Höhe, skaliert
  padding: EdgeInsets.all(16.r),   // responsiv skaliert
  child: Text(
    'Hello',
    style: TextStyle(fontSize: 14.sp),  // Schriftgröße skaliert
  ),
)
```

| Extension | Bedeutung |
|---|---|
| `.w` | Skaliert relativ zur Design-Breite |
| `.h` | Skaliert relativ zur Design-Höhe |
| `.r` | Skaliert nach dem kleineren der beiden Faktoren (für Radien, Icons) |
| `.sp` | Schriftgröße — berücksichtigt zusätzlich System-Textscaling |

**Hinweis:** Nur verwenden, wenn Responsive Design bestätigt wurde (vorher fragen).

---

## spacing_extensions.dart

```dart
import 'package:flutter/widgets.dart';

extension SpacingExtensions on num {
  /// Vertical SizedBox: 16.verticalSpacing
  SizedBox get verticalSpacing => SizedBox(height: toDouble());

  /// Horizontal SizedBox: 16.horizontalSpacing
  SizedBox get horizontalSpacing => SizedBox(width: toDouble());

  /// EdgeInsets all sides
  EdgeInsets get allPadding => EdgeInsets.all(toDouble());

  /// EdgeInsets horizontal
  EdgeInsets get horizontalPadding =>
      EdgeInsets.symmetric(horizontal: toDouble());

  /// EdgeInsets vertical
  EdgeInsets get verticalPadding =>
      EdgeInsets.symmetric(vertical: toDouble());
}
```

### Usage
```dart
Column(
  children: [
    const HeaderWidget(),
    16.verticalSpacing,
    const ContentWidget(),
    24.verticalSpacing,
    const FooterWidget(),
  ],
)
```
