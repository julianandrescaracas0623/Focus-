# Claude workflow for Focus

Use `CLAUDE.md` as the authoritative project specification.

## Session protocol
1. Read `CLAUDE.md` and applicable `.cursor/rules/*.mdc`.
2. Inspect relevant code before editing.
3. Identify the feature/domain boundary.
4. Implement the smallest coherent change.
5. Add/update tests for changed business behavior.
6. Run formatting, analysis and tests.
7. Summarize files changed, business rules affected, tests run, and any limitations.

## Decision hierarchy
Product requirements > existing domain rules > architecture > style preferences > implementation convenience.

When requirements conflict with this document, preserve safety and correctness and document the conflict.

## Flutter implementation
Use feature-first layered architecture:
presentation -> domain <- data.

Keep business logic in entities/use cases, not screens.
Keep persistence and APIs behind repository/data-source abstractions.
Avoid adding dependencies unless necessary.

## Security
Assume external input is untrusted.
Never commit secrets.
Never log credentials, tokens or private reflection content.
Use secure storage for sensitive credentials.
Use HTTPS only.
Use explicit timeouts and safe retry policies.

## Quality gates
Do not declare work complete until:
- dart format --output=none --set-exit-if-changed .
- flutter analyze
- flutter test

If the current environment cannot execute a command, report that exactly; never claim it passed.
