# Focus — Engineering & Business Rules

## 1. Product
Focus is a mobile personal-growth application centered on reading, weekly commitments, daily completion, progress, and automatic achievements. For the approved v1 scope, follow `docs/BUSINESS_RULES.md` as the authoritative product and data specification.

Product principle:
> No se trata de ser perfecto todos los días, sino de construir una versión de ti mismo de la que puedas sentirte orgulloso.

Focus should encourage consistency, reflection, and recovery after failure. It must not turn growth into a punitive system.

## 2. Stack
- Flutter / Dart
- Material 3
- Feature-first layered architecture
- Local persistence behind repositories
- Tests for domain and critical UI flows
- GitHub Actions CI
- Cursor + Claude as primary AI development tools

Keep dependencies minimal. Before adding a package, justify the problem, SDK alternative, maintenance, compatibility, security and size impact.

## 3. Architecture
Dependency direction:
`presentation -> domain <- data`

Suggested structure:
```
lib/
  app/
    app.dart
    router/
    theme/
  core/
    errors/
    time/
    storage/
    utils/
  features/
    books/
      data/
      domain/
      presentation/
    habits/
      data/
      domain/
      presentation/
    achievements/
      data/
      domain/
      presentation/
    reflections/
      data/
      domain/
      presentation/
    dashboard/
      presentation/
    settings/
      presentation/
  shared/
    widgets/
    extensions/
```

Presentation owns screens/widgets/state. Domain owns entities, value objects, use cases, and repository contracts. Data owns DTOs, data sources, repository implementations and mappers.

## 4. Core domain

### Book
Conceptual fields:
- id
- title
- author
- status: pending | inProgress | completed
- progress [0,1]
- order
- startedAt
- completedAt
- notes

Rules:
- progress must remain between 0 and 1.
- completed => progress = 1.
- completedAt exists only when completed.
- preserving reading history is preferred over destructive deletion.

### Weekly commitment
Represents a concrete commitment for a week.
- id
- title
- description
- weekStart
- targetDays
- completedDays
- active

Rules:
- targetDays > 0.
- completedDays >= 0 and <= targetDays.
- a commitment can only have one completion record per calendar day.
- week changes must preserve historical data.

### Daily check-in
- commitmentId
- date
- completed
- optional note

Rules:
- one record per commitment/day.
- use one calendar/time-zone abstraction for business-date logic.

### Achievement
Achievements are derived from domain events/state. Centralize unlocking logic. Never trust externally supplied achievement state without validating it.

### Reflection
Daily reflections contain a prompt and response. Treat them as private user content by default.

## 5. Product behavior

### Books
Users can view pending/in-progress/completed books, start a book, update progress, complete it, and add notes.

### Weekly Focus
Users create commitments and target frequency, record daily completion, view progress and streaks, and retain weekly history.

### Achievements
Examples:
- first commitment;
- first completed book;
- 7 consistent days;
- complete all commitments in a week;
- maintain a streak.

Do not design rewards that encourage harmful or compulsive behavior or reward simply opening the app.

### Reflection
Reflections should promote learning and observation, not guilt.

## 6. Business-rule principles
1. No business logic in widgets.
2. One source of truth per business concept.
3. Centralize date logic.
4. Calculate streaks from history instead of trusting manually stored counters where practical.
5. Preserve history.
6. Prefer idempotent operations.
7. Use typed domain errors.
8. Keep statistics derived when possible.
9. Avoid duplicated calculations across screens.
10. Do not introduce gamification that rewards app opening itself.

## 7. Flutter/Dart quality
- Null safety.
- Avoid dynamic unless justified.
- Prefer explicit types.
- Small focused classes and widgets.
- Prefer const.
- No expensive work in build().
- No direct I/O from UI.
- Deterministic rendering.
- No giant methods.
- Comments explain decisions, not obvious code.

## 8. State management
Use one consistent state-management approach. Keep UI state separate from domain state. Represent loading, data, empty and error states explicitly where meaningful. Never silently swallow failures.

## 9. Persistence
Persistence lives behind repository interfaces. UI must not access SQLite/Isar/Hive/Supabase/Firebase directly. Domain must not be coupled to the chosen persistence provider.

## 10. Security baseline
- Never commit secrets, API keys, tokens or passwords.
- Use secure platform storage for sensitive auth/session data.
- Do not log secrets or private reflections.
- Collect the minimum personal data necessary.
- Use HTTPS only.
- Set network timeouts.
- Use controlled retry policies.
- Validate all external data.
- Handle unexpected/malformed responses safely.
- Review dependencies and transitive risk.
- Do not place sensitive information in query strings.

## 11. Error handling
Use typed errors for validation, storage, network, authentication, configuration and corrupted data. UI messages must be human-readable and must not expose internal implementation details.

## 12. Testing
Minimum business coverage:
- book progress/completion rules;
- commitment completion;
- duplicate daily completion prevention;
- streak calculation;
- achievement unlocking;
- persistence/mapping edge cases.

Important UI flows should have widget/integration coverage.

## 13. AI operating rules — Cursor + Claude
These rules apply whenever Cursor or Claude changes this repository.

### Before coding
- Read this CLAUDE.md.
- Read applicable Cursor rules under .cursor/rules/.
- Inspect relevant existing files.
- Identify the current architecture before proposing a new one.
- State assumptions internally through code comments/docs only when they are durable decisions.

### During coding
- Make minimal cohesive changes.
- Reuse existing abstractions.
- Do not create parallel implementations of the same concept.
- Do not invent packages or APIs.
- Do not bypass validation.
- Do not disable lint/analyzer rules just to pass CI.
- Keep security-sensitive work explicit.

### After coding
Run:
```
dart format --output=none --set-exit-if-changed .
flutter analyze
flutter test
```

If one cannot run locally, report exactly which check could not run and why.

### Prohibited shortcuts
- `--no-verify` to bypass quality gates.
- Empty catches.
- Hard-coded credentials.
- Silent fallback that hides data corruption.
- Deleting tests to make them pass.
- Large unrelated refactors during a feature.

## 14. Definition of Done
A change is done only when:
- requirement is satisfied;
- architecture is respected;
- validation exists;
- errors are handled;
- relevant tests exist;
- format/analyze/tests pass;
- no secret is introduced;
- documentation is updated when behavior or architecture changes.

## 15. Git workflow
Prefer small commits:
- feat:
- fix:
- refactor:
- test:
- docs:
- chore:
- security:

Keep unrelated changes separated.

## 16. Project priorities
1. Correctness
2. Security
3. Maintainability
4. UX
5. Performance
6. Implementation convenience
