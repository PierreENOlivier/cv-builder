# Testing and Review Guide

Use this guide when adding tests, reviewing changes, or changing quality checks.

## Testing principles
- Test externally observable behavior and requirements, not just implementation details.
- Use unit tests for domain rules and pure transformations.
- Use integration tests for interactions among domain logic, YAML persistence, references, and rendering.
- Test failure and recovery paths as well as happy paths.
- Keep filesystem tests isolated with temporary directories and test fixtures.
- Avoid brittle assertions about incidental formatting unless it is part of the product contract.

## High-value behavior to protect
- Source replacement and new variant creation produce distinct outcomes; CV definitions contain selection rather than saved content overrides.
- Editing a source block follows an explicit replace-or-new-entry choice.
- One CV's edits do not silently alter another CV.
- A CV reopens with its section/block skeleton, detects missing references, and offers explicit tag-based alternatives; rendered output remains visual history.
- Invalid YAML and unsupported schema versions are reported without data loss.
- External changes and unsaved local changes are detected and resolved explicitly.
- Preview and PDF export share rendering behavior.
- Blocks remain intact across page boundaries according to the requirements.

## Review checklist
Prioritize findings by severity:
1. Data loss, corruption, or silent changes.
2. Requirement violations and incorrect behavior.
3. Security or privacy issues.
4. Broken error/recovery handling.
5. Maintainability and unnecessary coupling.
6. Style preferences.

For each finding, identify the affected file/symbol and explain a reproducible scenario where possible. Distinguish definite defects from risks or preferences. Do not modify files during a review-only task.

Run relevant tests/checks and report actual commands and results. If a check cannot run, state why.
