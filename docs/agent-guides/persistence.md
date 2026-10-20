# Persistence and Data Integrity Guide

Use this guide for YAML serialization, schema versions, workspace layout, saving, reloading, recovery, and external file changes.

## Required behavior
- Source content, templates, CV definitions, overrides, and related data are stored in user-accessible YAML files.
- YAML has explicit schema-version information.
- Validate both YAML syntax and expected structure.
- Give understandable errors and identify the affected file/field where possible.
- Never silently discard or overwrite invalid or conflicting data.
- Main-file saves are explicit. Recovery/interim files do not silently replace the main files.
- Reload is explicit and warns if unsaved edits may be lost.
- Detect external changes. If local unsaved edits conflict with external changes, block save until the user resolves the file-level conflict.
- Version 1 conflict resolution chooses the whole local file or whole external file; do not add field-level merge.
- Confirm destructive or potentially lossy operations.

## Implementation guidance
- Keep serialization and filesystem details behind a clear boundary.
- Validate loaded data before exposing it to domain operations.
- Use safe single-file writes, such as temporary-file plus atomic replacement where supported.
- Treat a multi-file workspace update as a separate consistency problem; single-file atomic replacement does not make the whole workspace transactional.
- Do not assume cloud-sync tools provide locking, ordering, or conflict-free writes.
- Consider missing, duplicated, partially written, stale, or unsupported-schema files.
- Keep migrations explicit, testable, and non-destructive.
- Never overwrite user data merely because parsing failed.

## Tests to consider
Malformed YAML; unsupported schema version; missing file; duplicate IDs; invalid references; interrupted save; external modification; simultaneous local edits; reload cancellation; recovery acceptance/rejection; and conflicting synchronized files.

Use temporary directories and isolated fixtures. Never test destructive behavior against a real user workspace.
