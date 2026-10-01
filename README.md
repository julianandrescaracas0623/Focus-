# Focus

> No se trata de ser perfecto todos los días, sino de construir una versión de ti mismo de la que puedas sentirte orgulloso.

Focus es una aplicación móvil de crecimiento personal basada en lectura, compromisos semanales, cumplimiento diario, progreso, logros y reflexión.

## Stack
Flutter + Dart · Material 3 · arquitectura por features · tests · GitHub Actions

## Desarrollo con IA
El proyecto está preparado para trabajar con Cursor + Claude.

Antes de cambiar código:
1. Leer `CLAUDE.md`.
2. Revisar `.cursor/rules/focus.mdc`.
3. Inspeccionar el código existente.
4. Mantener arquitectura y reglas de negocio centralizadas.

## Verificaciones
```bash
dart format --output=none --set-exit-if-changed .
flutter analyze
flutter test
```
