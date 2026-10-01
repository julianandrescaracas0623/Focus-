# Focus Architecture

Dependency direction:

`presentation -> domain <- data`

Recommended feature structure:

```
feature/
  data/
  domain/
  presentation/
```

Cross-cutting concerns belong in `core/`. Reusable UI belongs in `shared/`.

The domain layer should remain independent of Flutter whenever practical.
