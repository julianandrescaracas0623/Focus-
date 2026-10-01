# Focus Flutter Bootstrap

This repository is intentionally bootstrapped for a Flutter application.

## Local bootstrap

From the repository root:

```bash
flutter create . --project-name focus --org com.focus
flutter pub get
dart format .
flutter analyze
flutter test
```

Do not run `flutter create` if the repository already contains a Flutter project. The generated app should preserve the configuration files already committed here and place application code under `lib/`.

## Target architecture

After bootstrap, organize `lib/` as:

```
lib/
  app/
  core/
  features/
    books/
    habits/
    achievements/
    reflections/
    dashboard/
    settings/
  shared/
```

The initial implementation should establish the app shell, theme, routing, error handling and dependency injection before feature complexity.
