# CV Builder — Agent Instructions

## Project purpose
A local-first Python desktop application for maintaining reusable CV content, assembling independent CV versions, editing content in-app, and producing consistent multi-page PDFs. Version 1 focuses on Industry CVs.

## Source of truth
`docs/REQUIREMENTS_GATHERING.md` is the primary product requirements document and source of truth.
Path: 

- Read the relevant parts before planning or changing behavior.
- Preserve confirmed requirements. Distinguish requirements, assumptions, recommendations, and open decisions.
- Do not reopen settled decisions without a concrete contradiction, implementation consequence, or material risk.
- Do not treat an architectural implication as an approved implementation choice.

## Working agreement
- Work incrementally; do not generate the whole application in one pass.
- For nontrivial work, inspect first, state the plan, make a focused change, verify it, and report results.
- Planning/review-only tasks must not modify files.
- Wait for approval before implementing a materially different architecture or resolving a product decision with significant consequences.
- Avoid unrelated refactoring, new dependencies, and scope expansion.
- Inspect the working tree before edits. Never discard user changes.
- Do not commit, amend, branch, or push unless explicitly requested.
- Report commands actually run and their actual results. Never claim unrun checks passed.

## Architecture guardrails
- Prefer the simplest design that satisfies confirmed requirements.
- Keep domain rules testable without launching the GUI or accessing the real workspace.
- Separate domain behavior from GUI widgets, YAML serialization/filesystem access, and PDF rendering.
- Keep domain concepts distinct from YAML representation and GUI presentation.
- Make dependencies explicit; avoid circular dependencies, hidden global state, and abstractions without a concrete purpose.
- Do not introduce a hosted backend, required database, microservices, plugin framework, or generalized architecture for hypothetical future needs.
- Do not select GUI, rich-text, YAML, or PDF dependencies without evaluating the relevant requirements and trade-offs.

## Domain invariants
- A canonical source entry groups variants and has a stable ID; each variant has its own unique ID.
- Tags belong to variants/blocks, and tag IDs do not depend on mutable labels.
- A source variant, a section type, a section instance, and a CV definition are distinct concepts.
- Replace the existing source under its existing ID and save a new reusable variant under a new unique ID are distinct actions.
- Editing an existing block does not create a persisted CV-local snapshot or override; it replaces the source or creates a new master-source entry.
- Source changes must not be silently propagated to other CV definitions.
- Styling files define presentation separately from candidate source content and CV composition.

## Data integrity
- Validate YAML syntax and schema before using persisted data.
- Never silently discard malformed, unknown, conflicting, or invalid data.
- Do not overwrite externally changed or invalid data without an explicit, approved resolution path.
- Normal saves are explicit; recovery files do not silently replace main files.
- Confirm destructive or potentially lossy actions.
- Consider interrupted writes and third-party synchronization conflicts. A single-file atomic write does not make a multi-file workspace transaction atomic.
- Do not claim that a source ID alone reproduces historical content after source replacement/deletion. Persist the tags needed to offer alternatives, retain rendered outputs as visual history, and follow the approved recovery behavior.

## Testing and done criteria
- Test observable behavior and requirements, including important failure paths.
- Use isolated temporary directories for filesystem tests.
- For behavior changes, add or update tests and run relevant checks where available.
- A task is complete when the agreed scope is met, relevant verification is reported, docs are updated where needed, and limitations/open decisions are identified.

## Load task-specific guidance only when relevant
Before working on a task, read the applicable guide(s) below. Do not load every guide for every task.

- Architecture, boundaries, dependency direction, or design decisions: `docs/agent-guides/architecture.md`
- Domain concepts, IDs, variants, CV composition, and source save actions: `docs/agent-guides/domain-model.md`
- YAML files, schema validation, save/reload, recovery, external changes: `docs/agent-guides/persistence.md`
- Rich text, styling templates, preview, pagination, PDF export: `docs/agent-guides/rendering.md`
- Tests, review, quality checks: `docs/agent-guides/testing-and-review.md`
- Agent workflow, plans, ADRs, approval gates: `docs/agent-guides/workflow.md`
- Python code style, naming, imports, type hints, docstrings, and formatting: `docs/agent-guides/code-styling.md`
- Selecting or evaluating third-party libraries: read `docs/agent-guides/technology-evaluation.md`

If a guide does not exist yet, say so; do not invent its contents. Read `REQUIREMENTS_GATHERING.md` for the product requirements relevant to the task.
