# CV Builder — Architecture Proposal

## Status and scope

This document describes the approved architectural direction for the documentation handover. It does not select final GUI, YAML, rich-text, HTML, or PDF libraries and does not prescribe a complete Python package tree. The repository is currently a scaffold with no implemented application architecture.

The settled product boundary is a local modular monolith using three distinct user-visible YAML responsibilities: Master source, Styling, and CV definition.

## Architecture at a glance

```text
                 +----------------------+
                 | GUI / application     |
                 | workflow              |
                 +----------+-----------+
                            |
          +-----------------+-----------------+
          |                                   |
          v                                   v
 +---------------------+             +---------------------+
 | Domain/composition  |             | Persistence         |
 | IDs, tags, selection |             | YAML, validation,   |
 | ordering, rules      |             | safe saves/reload   |
 +----------+----------+             +----------+----------+
            |                                   |
            +-----------------+-----------------+
                              v
                    +-------------------+
                    | Render model and  |
                    | HTML/PDF output   |
                    +-------------------+

 Master source YAML  ---> reusable content and section definitions
 Styling YAML        ---> visual presentation
 CV definition YAML  ---> selected source IDs, order, and composition
```

The GUI should not directly manipulate YAML or rendering widgets. Domain rules should be usable without the GUI or a real workspace.

## Three-file model

### Master source

The master source owns reusable candidate data and structural definitions:

- Candidate identity and contact/header information.
- Section types, including their fields and semantic layout requirements.
- Reusable sections and blocks, with references rather than duplicated block content.
- Canonical entries and their source variants.
- Stable IDs, labels, tags, tag categories, and source metadata.

A canonical entry represents one real-world item or reusable concept. Each variant has its own stable ID and tags. Tags are independent objects or references whose IDs do not depend on labels. Content tags relate variants; CV-type tags describe intended CV use. A source block is not a CV definition, and a section type is not a section instance.

### Styling

Styling owns document-wide presentation: fonts, sizes, colors, spacing, alignment, borders, section labels, entry layouts, and page settings. Section types describe what fields and semantic arrangement a block needs; Styling determines the visual treatment. Styling must not contain candidate source content or CV selection state.

Built-in Styling files are read-only. User-created Styling files may be supported as validated reusable files. Changing Styling affects future previews/renders, not an already-rendered PDF.

### CV definition

A CV definition owns one CV's composition:

- CV metadata and intended CV type/tag.
- Selected Styling reference.
- Ordered section instances and their selected section types.
- Selected source sections/blocks, canonical IDs, variant IDs, and recorded tags needed for recovery/alternative selection.
- Block and section order and CV-specific labels/configuration.

The CV definition does not contain a saved content snapshot or persisted local override. Editing an existing block results in either replacement under the original source ID or a new master-source entry with a new ID. A missing source reference is surfaced to the user and is not silently substituted.

## Main components and responsibilities

### GUI and application workflow

Presents candidate, library, composition, editor, filter, preview, and export workflows. It manages explicit user confirmation and draft/save/cancel actions, but delegates validation and persistence to application services.

### Domain and composition

Models candidate/profile data, section types, sections, canonical entries, variants, tags, CV definitions, and resolved compositions. Enforces stable-ID rules, tag filtering, one-variant-per-canonical-entry, valid ordering, and distinct replace/new-entry behavior. It does not read YAML or access GUI widgets.

### Persistence

Loads and validates YAML, applies explicit schemas, tracks file state, handles safe saves/reload/recovery, and reports malformed, missing, conflicting, or unsupported data. It owns filesystem paths and serialization, not business rules.

### Rendering

Resolves an approved CV definition with master source and Styling into a semantic render model, then produces preview and HTML/PDF output through a shared layout behavior. It owns pagination and presentation, not source mutation or CV selection rules.

### Technology adapters

Keep chosen GUI, YAML, rich-text, HTML, and PDF libraries behind narrow boundaries. No dependency is settled until it is evaluated against cross-platform support, rich text, preview/PDF consistency, pagination, licensing, packaging, accessibility, and testability.

## Core workflows

### Create and compose a CV

1. Select or create a candidate.
2. Create an empty CV definition or reuse an existing one.
3. Choose a CV-type tag and Styling.
4. Add a section instance and choose its section type.
5. Filter source blocks by section type and tags; always show `Empty Block` first.
6. Select blocks, add them, fill empty blocks, and save or cancel edits.
7. Reorder sections and blocks or apply explicit chronological sorting.
8. Save the CV definition and preview the resolved composition.

### Maintain the source library

The library viewer shows reusable sections and blocks under generic section types. Creating a block writes a new master-source ID. Editing an existing block requires an explicit choice:

- Replace the existing source under its current ID; or
- Save a new reusable entry/variant with a new ID, retaining source tags and metadata unless deliberately changed.

The operation must not silently update other CV definitions. If it changes both source and CV files, persistence must validate and safely handle the multi-file save plan.

### Reopen a CV with source changes

1. Load and validate the CV definition, master source, and selected Styling.
2. Build the section and block skeleton from the CV definition.
3. Resolve each source ID and verify its recorded tags.
4. If an entry is missing, alert the user and offer tag-based alternatives.
5. Require an explicit user choice before replacing the missing reference.
6. Preview or render only after all required references are resolved.

The source reference alone cannot guarantee reconstruction of a previously rendered structured document after replacement or deletion. The exported PDF is retained visual history; manual recreation using alternatives remains the recovery path.

### Preview and export

The application resolves the current CV definition, master source, and Styling into one render model. Preview and final HTML/PDF export use the same layout behavior. Blocks flow across pages; a block that does not fit in the remaining space moves intact to the next page. Overwriting an existing rendered file requires confirmation. An intrinsically oversized block is an unresolved design case.

## Ownership and persistence

The source owns reusable content and tags. Styling owns presentation. The CV definition owns selection and arrangement. The rendered PDF/HTML output owns the result of a rendering operation and is not a source of structured data.

All YAML has an explicit schema version and is validated before entering the domain. Invalid files remain available for recovery/correction. Explicit saves, safe temporary-file replacement, external-change detection, and recovery of partial multi-file operations are persistence concerns. Atomic replacement of one file does not make a multi-file update transactional.

## Error handling and integrity

- Never silently drop unknown, invalid, duplicate, or conflicting data.
- Report the affected file and field/location where possible.
- Block saves when local unsaved edits conflict with external changes.
- Use whole-file local/external conflict resolution in Version 1.
- Confirm deletion, source replacement where it affects reusable content, and rendered-output overwrite.
- Do not silently substitute a missing source block; offer explicit alternatives based on recorded tags.
- Keep domain tests independent of GUI and real filesystem state.

## Risks and unresolved decisions

The exact YAML schemas, filenames, directory layout, multi-file recovery strategy, technology choices, accessibility requirements, and oversized-block behavior remain open. These are tracked in `docs/REQUIREMENTS_GATHERING.md`.

The principal integrity trade-off is deliberate: Version 1 does not implement a full source-history or snapshot system. References keep the source library reusable, while rendered documents provide visual history. If future requirements demand structured historical reconstruction, that should be approved as a separate architecture decision.

## Recommended next steps

1. Define example schemas and fixtures for the three file responsibilities.
2. Implement/test pure domain rules for IDs, tags, selection, ordering, and source save actions.
3. Prototype YAML validation and safe multi-file save/recovery behavior.
4. Prototype rich-text editing and shared HTML/PDF rendering before selecting dependencies.
5. Reconcile the chosen implementation with the requirements and update this proposal only when decisions are approved.
