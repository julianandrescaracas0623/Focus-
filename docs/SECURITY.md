# Security Baseline

Focus handles user activity and may handle private reflections.

Principles:
- minimum data collection;
- no secrets in source control;
- secure storage for sensitive tokens;
- HTTPS only;
- explicit timeouts/retries;
- no sensitive data in logs;
- validate all external data;
- keep dependencies minimal and reviewed;
- treat reflections as private by default.

Security-sensitive changes must include tests and document the threat/mitigation when the architecture is affected.
