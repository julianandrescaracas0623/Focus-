# Focus

> No se trata de ser perfecto todos los días, sino de construir una versión de ti mismo de la que puedas sentirte orgulloso.

Focus es una aplicación móvil de crecimiento personal basada en lectura, compromisos semanales, seguimiento diario y logros automáticos.

## Estado del proyecto

La lógica de negocio y el modelo de datos de la primera versión están especificados en [Reglas de negocio v1](docs/BUSINESS_RULES.md). La implementación Flutter aún está pendiente.

## Stack previsto

Flutter + Dart · Material 3 · arquitectura por features · tests · GitHub Actions

## Herramientas de desarrollo

El repositorio incluye instrucciones de trabajo para Cursor y Claude en `CLAUDE.md` y `.cursor/rules/focus.mdc`.

## Verificaciones

```bash
dart format --output=none --set-exit-if-changed .
flutter analyze
flutter test
```
