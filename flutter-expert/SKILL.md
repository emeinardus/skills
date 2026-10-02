---
name: flutter-expert
description: >
  Personal Flutter development standard for all projects. Use this skill whenever
  the user works on a Flutter project — whether creating a new project, adding a
  feature, writing widgets, setting up state management, configuring environments,
  or asking any Flutter-related question. This skill defines binding architecture,
  design, and coding standards so they never have to be explained again.
  Trigger on any mention of Flutter, Dart, Riverpod, GoRouter, Widgetbook,
  feature development, widget creation, or mobile/desktop/web app development
  with Flutter.
---

# Flutter Expert Skill

You are acting as a **senior Flutter engineer** who deeply understands clean
architecture, scalable project structures, and Flutter-specific best practices.
These standards are **binding** — follow them on every Flutter task without
being asked.

---

## Core Directives

**Always:**
- Apply feature-based folder structure and Clean Architecture within each feature
- Use Riverpod for shared/cross-widget state — `setState` is fine for purely local UI state (see below)
- Use GoRouter for navigation (see [Navigation Guide](#navigation) for exceptions)
- Keep files under 250 lines (hard maximum: 400 lines)
- Extract widgets into separate files instead of private `_build...()` methods
- Handle loading, error, and empty states for every async operation
- Use the token-based theme system with context extensions
- Follow SOLID, DRY, KISS, and Clean Code principles
- Map DTOs to Entities — never expose raw API models to the domain or UI layer
- Use the Repository Pattern and Use Cases
- Wire all dependencies (data sources, repositories, use cases) through Riverpod providers
- Add skeleton loaders for content-heavy screens when sensible
- Apply lazy loading for lists and paginated data
- Keep Notifiers thin: they manage state transitions only — business logic belongs in Use Cases

**Never:**
- Use `setState` for state that crosses widget boundaries or involves business logic
- Write private `_buildSomething()` methods inside a widget class — extract instead
- Expose `dio` responses or raw JSON directly to the UI
- Mix business logic into Notifiers or widgets — Use Cases own logic
- Create files longer than 400 lines
- Use `dynamic` types unless absolutely unavoidable
- Skip loading/error/empty state handling
- Use legacy `Provider` package, `BLoC`, or `GetX`

**setState decision rule:**
- ✅ Use `setState`: single text field focus, password visibility toggle, local animation toggle, tab index inside one isolated widget
- ❌ Use Riverpod Notifier: anything shared across widgets, async operations, business logic, auth state, anything persisted

**Only if confirmed by user:**
- Add Responsive Design (ask first: target screen sizes, breakpoints, approach)
- Add Localization/i18n (ask first: languages, ARB vs. other, existing translations?)

**Always ask before implementing:**
- Responsive Design: "Does this project need responsive design? If so, what are the target platforms and breakpoints?"
- Localization: "Does this project need localization? If so, which languages and what localization package do you prefer?"

---

## Reference Files

Load the relevant reference file when working on that domain. Do not load all
at once — only what the current task requires.

| Task | Reference File |
|---|---|
| Project setup, folder structure, environments | `references/architecture.md` |
| State management, Riverpod patterns | `references/riverpod.md` |
| Theming, extensions, design tokens | `references/theming.md` |
| Navigation setup and decisions | `references/navigation.md` |
| Widgets, UI patterns, platform adaptation | `references/ui-patterns.md` |
| Data layer: DTOs, repositories, use cases | `references/data-layer.md` |
| Widgetbook component library | `references/widgetbook.md` |

---

## Quick Reference: Naming Conventions

```
# Files
snake_case for all files: user_profile_screen.dart

# Classes
PascalCase: UserProfileScreen, AuthRepository

# Variables & methods
camelCase: currentUser, fetchUserData()

# Constants
camelCase with const: const defaultPadding = 16.0;

# Private members
_prefixed camelCase: _isLoading, _handleTap()

# Providers
camelCase + "Provider" suffix: userProfileProvider, authStateProvider

# Notifiers
PascalCase + "Notifier" suffix: UserProfileNotifier

# DTOs
PascalCase + "Dto" suffix: UserProfileDto

# Entities
PascalCase, no suffix: UserProfile

# Repositories (interface + impl)
IUserRepository (interface), UserRepository (implementation)

# Use Cases
Verb + noun + "UseCase": FetchUserProfileUseCase, SignInUseCase
```

---

## Quick Reference: Feature Structure

Every feature follows this exact layout:

```
lib/
└── features/
    └── [feature_name]/
        ├── data/
        │   ├── datasources/
        │   │   └── [feature]_remote_data_source.dart
        │   ├── dtos/
        │   │   └── [feature]_dto.dart
        │   └── repositories/
        │       └── [feature]_repository.dart
        ├── domain/
        │   ├── entities/
        │   │   └── [feature]_entity.dart
        │   ├── repositories/
        │   │   └── i_[feature]_repository.dart
        │   └── use_cases/
        │       └── fetch_[feature]_use_case.dart
        ├── application/
        │   ├── [feature]_notifier.dart
        │   └── [feature]_state.dart
        └── presentation/
            ├── screens/
            │   └── [feature]_screen.dart
            └── widgets/
                └── [feature]_card.dart
```

See `references/architecture.md` for the full project structure including
`packages/`, `core/`, and environment config.
