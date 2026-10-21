# CV Builder — Documentation Handover Update

## Review scope

The repository and documentation were inspected before editing. The repository is a clean scaffold: `src/cv_builder/__init__.py` is the only source file, there are no tests, and `pyproject.toml` contains only the initial project metadata and a `pandas` dependency. No application behavior was changed.

Inspected files:

- `AGENTS.md`
- `docs/PROJECT_BRIEF.md`
- `docs/REQUIREMENTS_GATHERING.md`
- `docs/ARCHITECTURAL_PROPOSAL.md`
- `docs/TODO.MD`
- All files under `docs/agent-guides/`
- `src/cv_builder/__init__.py`
- `pyproject.toml`, `README.md`, and `uv.lock`

## Documents changed

- `docs/PROJECT_BRIEF.md` — created the previously empty brief, covering purpose, workflow, capabilities, three-file model, outputs, and scope boundaries.
- `docs/REQUIREMENTS_GATHERING.md` — replaced the earlier requirements text with an aligned requirements source of truth covering source management, IDs/tags, section types, composition, filtering, ordering, Styling, rendering, saved CV definitions, missing references, and open questions.
- `docs/ARCHITECTURAL_PROPOSAL.md` — replaced the superseded proposal with an approved-direction description of the three-file model, boundaries, components, workflows, ownership, persistence, rendering, and integrity risks.
- `AGENTS.md` — updated concise domain and persistence guardrails and retained links to task-specific guides.
- `docs/agent-guides/domain-model.md` — removed the saved-local-override model and aligned terminology with master source and CV definitions.
- `docs/agent-guides/persistence.md` — clarified that CV definitions retain source IDs/tags for recovery rather than snapshots or overrides.
- `docs/agent-guides/rendering.md` — clarified approved editing state and resolution through master source plus CV definition.
- `docs/agent-guides/testing-and-review.md` — updated high-value tests for replace/new-entry behavior and missing-reference recovery.

The architecture, code-style, technology-evaluation, and workflow guides were inspected and remain applicable without substantive changes.

## Principal decisions now documented

- Master source, Styling, and CV definition are separate YAML responsibilities.
- Section type, source section, section instance, canonical entry, source variant/block, and CV definition remain distinct concepts.
- Stable IDs are independent of labels; tags are retained for filtering and missing-reference alternatives.
- Editing existing source content means either replacing the existing source ID or creating a new master-source entry with a new ID.
- Saved CV definitions do not contain snapshots or persisted local overrides.
- Source changes do not silently rewrite other CV definitions.
- Missing references require an alert and explicit tag-based alternatives.
- Rendered PDF/HTML output is retained as output history; a source reference alone does not guarantee structured historical reconstruction.

## Contradictions found

The previous `REQUIREMENTS_GATHERING.md`, `ARCHITECTURAL_PROPOSAL.md`, and several guides described a three-action model containing **Save version** as a persisted CV-local override, source revision/hash tracking, and source-change warnings based on those revisions. This contradicted the latest handover decision that snapshots and overrides are no longer saved in CV definitions. Those passages were removed or reframed.

The implementation does not yet contradict the revised documentation because the application is not implemented. The scaffold's `pandas` dependency and Python version remain technology/toolchain questions, not architectural decisions.

## Unresolved decisions

The updated requirements retain open questions for exact YAML schemas, filenames and workspace layout, required fields, CV-type tag representation, Styling assets, HTML output semantics, PDF naming, multi-file recovery, synchronization conflicts, accessibility, oversized blocks, and final technology selection. The requirements document is the authoritative list.

The main known limitation is intentional: Version 1 does not promise structured historical reconstruction after a source entry is replaced or deleted. The exported document is visual history and manual recreation with alternatives is the recovery path.

## Recommended next development step

Define small example YAML schemas and fixtures for the three file responsibilities, then implement and test pure domain rules for stable IDs, tags, section/block selection, ordering, and replace-versus-new-source behavior. Do not select GUI or rendering dependencies until the focused rich-text and shared HTML/PDF proofs of concept described in the architecture and technology guide are complete.

## Modified-line audit links

The following links open the changed documents. Line numbers are shown as plain text because the repository viewer does not resolve Markdown `:line` anchors in Windows paths:

- [AGENTS.md](C:/Users/pieln/Documents/gitwork/python/cv-builder/AGENTS.md) — lines 34, 46, 61
- [PROJECT_BRIEF.md](C:/Users/pieln/Documents/gitwork/python/cv-builder/docs/PROJECT_BRIEF.md) — lines 1, 9, 21, 31, 38
- [REQUIREMENTS_GATHERING.md](C:/Users/pieln/Documents/gitwork/python/cv-builder/docs/REQUIREMENTS_GATHERING.md) — lines 3, 27, 43, 64, 77, 93, 106, 140
- [ARCHITECTURAL_PROPOSAL.md](C:/Users/pieln/Documents/gitwork/python/cv-builder/docs/ARCHITECTURAL_PROPOSAL.md) — lines 3, 18, 39, 78, 98, 135, 147
- [domain-model.md](C:/Users/pieln/Documents/gitwork/python/cv-builder/docs/agent-guides/domain-model.md) — lines 5, 17, 30, 37
- [persistence.md](C:/Users/pieln/Documents/gitwork/python/cv-builder/docs/agent-guides/persistence.md) — lines 6, 19, 25
- [rendering.md](C:/Users/pieln/Documents/gitwork/python/cv-builder/docs/agent-guides/rendering.md) — lines 3, 18
- [testing-and-review.md](C:/Users/pieln/Documents/gitwork/python/cv-builder/docs/agent-guides/testing-and-review.md) — line 14

## Verification

- `git diff --check` completed without whitespace errors.
- `git status --short` shows only the intended documentation files and this report as modified/untracked work.
- No application tests were run because the repository contains no tests or implemented application behavior.
