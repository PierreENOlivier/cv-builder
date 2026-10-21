# CV Builder — Requirements Gathering

## Document status

This document is the product requirements source of truth for Version 1. It separates confirmed requirements, constraints, assumptions, recommendations, and unresolved decisions. It does not prescribe Python modules, classes, or third-party libraries.

## Problem statement and scope

The application helps a job candidate maintain reusable CV content, assemble independent CV definitions for different applications, edit content in the application, and render consistent documents without requiring Word or another external editor.

Version 1 is a local-first graphical desktop application focused on Industry CVs. A workspace may contain multiple candidates. Each candidate has a reusable source library, styling files, CV definitions, and rendered outputs.

Version 1 does not include a hosted service, account system, required database, collaborative editing, or unrestricted document-layout authoring.

## Confirmed functional requirements

### Workspace and candidate profile

- The user selects a workspace location and may place it in a third-party-synchronized folder.
- The application remembers the last-used workspace and allows switching workspaces.
- One workspace may contain multiple candidates with independent data.
- Candidate creation is performed through the GUI.
- A candidate has a required name and optional title/credentials, location, phone number, email, and repeatable social/other links.
- A link may contain display text, URL, and optional icon metadata.
- The rendered header has conceptual lines for identity, contact/location, and links.

### Three-file responsibility model

The workflow uses three distinct YAML responsibilities. They must not be collapsed into one interchangeable file:

1. **Master source** — candidate profile data, section types and their field/layout definitions, reusable sections/blocks, canonical entries, variants, tags, and source metadata.
2. **Styling** — document-wide visual settings such as fonts, colors, spacing, borders, typography, and page layout.
3. **CV definition** — the composition for one CV: metadata, selected styling, ordered section instances, selected source references, and CV-specific ordering/configuration.

The exact filenames and directory layout remain open. The responsibilities do not.

### YAML storage and reliability

- User-accessible persisted data is YAML with explicit schema versions.
- YAML syntax and expected structure are validated before data is used.
- Invalid, unknown, conflicting, or unsupported data is not silently discarded or overwritten.
- Normal saves are explicit. Recovery/interim files may protect against crashes but do not silently replace main files.
- Reloading external changes is explicit and warns if local unsaved edits may be lost.
- If local unsaved edits conflict with external file changes, saving is blocked until the conflict is resolved.
- Version 1 conflict resolution is whole-file local-versus-external selection; field-level merging is out of scope.
- Safe writes and recovery behavior must account for interrupted operations across more than one file.

### Master source and domain concepts

- A **section type** defines the fields and presentation structure appropriate for content, such as `Education — compact` or `Education — detailed`. These are distinct types, not merely a display toggle.
- A **section** in the master source groups reusable blocks by reference; it does not duplicate their contents.
- A **canonical entry** groups versions of one real-world item or reusable content concept and has a stable unique ID.
- A **source variant/block** has its own stable unique ID, fields, labels, tags, and metadata.
- Tags belong to variants/blocks, not canonical entries. Sections may also have tags for categorization and filtering as defined by the source schema.
- Every section, block/variant, and tag has a stable unique identifier independent of mutable labels.
- Content tags group related reusable material, such as alternative descriptions of one role or generic section categories such as Education.
- CV-type tags describe intended CV use, such as Data Scientist or Researcher.
- A CV may select at most one variant from a canonical entry.
- Different real-world items use different canonical entries.
- Archived source entries are excluded from normal selection, can be restored, and may be permanently deleted only after confirmation.

Supported content includes Summary, Work Experience, Education, Portfolio, Skills, and custom sections. Required fields and exact schemas for each remain partly open.

### Source save actions and data integrity

Edits remain drafts until the user explicitly saves or cancels them. The three source-handling actions are distinct:

1. **Replace source:** replace the existing source entry under its existing ID.
2. **Save new reusable variant:** create a new source entry/variant with a new unique ID, label, and the selected tags and metadata.
3. **Save a CV-specific change:** under the latest decision, this is not a persisted local override. The edited result must either replace the original source or become a new master-source entry.

When a completely new section or block is created, it is added to the master source with a unique ID, label, tags, and metadata. When an existing section or block is edited, the user chooses whether to replace it under its ID or save the edited content as a new master-source entry. A copied/edited entry retains the tags and associated tag metadata of the source it was based on unless the user changes them explicitly.

Source changes are not silently propagated into other CV definition files. A CV definition records the source IDs and tags needed to resolve missing entries and offer alternatives. It does not contain saved snapshots or local content overrides.

### CV creation and composition

- The user selects or creates a candidate, then creates an empty CV or reuses an existing CV definition.
- An empty CV can use an existing CV-type tag or a new one.
- The user adds section instances and chooses a section type for each instance.
- A section instance is distinct from its master-source section type and from each selected source block.
- The block-selection dialog shows blocks associated with the chosen section type, supports tag filtering, and always presents an `Empty Block` option first regardless of filtering.
- Selecting `Empty Block` asks how many empty blocks to create.
- The user selects one or more blocks and adds them to the section.
- Empty blocks can be completed using only the fields defined by the selected section type.
- New or edited blocks may be assigned existing or new tags.
- Sections and blocks can be added, removed, and reordered. Drag-and-drop may be used where feasible.
- Work Experience and Education support explicit chronological and reverse-chronological sorting. Manual ordering takes precedence until the user sorts again.
- A CV definition references the selected Styling file and source IDs/tags; it does not become a copy of the master source.

### Library viewer and source changes

- A library viewer shows source sections and blocks with tag filtering.
- Blocks are organized under their generic section type.
- Users can edit or delete source sections and blocks, subject to confirmation for destructive actions.
- Existing CV definitions are not rewritten merely because source content changes.
- When opening a CV, the application builds its section/block skeleton and checks referenced sections and blocks.
- If a reference is missing, the application alerts the user and offers alternatives filtered by the tags recorded in the CV definition.
- The user may select an alternative or add a replacement source block. The system must not silently substitute content.
- A previously exported PDF remains the final rendered artifact until explicitly regenerated and confirmed for overwrite.
- A source reference alone cannot recreate the exact prior structured content after the referenced source is replaced or deleted. The exported document can serve as visual history, and the user can manually recreate the definition using replacement blocks.

### Styling and rendering

- Styling is independent of source content and CV structure.
- A Styling file controls fonts, typography, colors, spacing, alignment, borders, section labels, entry layouts, and page settings.
- Section types may define their own semantic layout requirements; Styling controls their visual presentation.
- Built-in styles are read-only. User-created styles may be supported after the initial implementation and must be validated before selection.
- Preview and final rendering use the same rendering behavior.
- Preview reflects the current approved editing state.
- PDF is the Version 1 export priority; HTML preview/export is also intended.
- Content flows automatically across multiple pages. A block that cannot fit in remaining space moves intact to the next page. Manual page breaks are out of scope.
- Existing rendered files are overwritten only after warning and confirmation.

### Editing and formatting

- Users edit blocks inside the application.
- A block must satisfy the relevant section-type validation before it can be saved.
- Supported content formatting is paragraphs, bullets, bold, italic, and hyperlinks.
- Fonts, sizes, alignment, spacing, colors, and layout come from Styling rather than arbitrary block formatting.
- Incomplete drafts can be cancelled and are not saved.
- Undo/redo is not required for Version 1.

## Non-functional requirements and constraints

- Local-first operation and user-controlled files.
- Cross-platform suitability, subject to technology evaluation.
- Domain rules must be testable without launching the GUI or using a real workspace.
- The design should remain understandable for a junior solo developer.
- Dependencies must be selected only after evaluating rich text, rendering, pagination, packaging, licensing, accessibility, and testability.
- `uv` manages Python and project dependencies.
- The application must prioritize correctness, recoverability, and explicit user action over convenience.

## Known assumptions and recommendations

- YAML remains the user-visible serialization format; exact schemas and file partition are implementation decisions.
- A modular monolith with explicit domain, application, persistence, and rendering boundaries is recommended for Version 1.
- A small proof of concept should validate rich-text editing and shared HTML/PDF rendering before dependencies are selected.

## Open questions

1. Exact YAML schemas, filenames, and candidate/workspace directory layout.
2. Whether a section itself is a reusable source object in all cases or only a structural definition with referenced blocks.
3. Required fields and validation rules for each content type.
4. Exact semantics of CV-type tags and whether they are stored as a distinct tag category in YAML.
5. Whether email is required in a candidate profile.
6. Starter Styling files and supported predefined CV types.
7. HTML export format and whether it is a saved artifact or preview output only.
8. PDF naming and output-location conventions.
9. The recovery/journal strategy when one user action changes multiple YAML files.
10. Behavior for duplicate, partially written, or synchronization-conflict files.
11. GUI, rich-text, YAML-validation, HTML, and PDF technology choices.
12. Behavior when a single block is taller than an entire page.
13. Accessibility requirements for the GUI and generated documents.

## Acceptance themes

- A new source block receives stable identity, labels, tags, and metadata.
- Replace and save-new-variant have distinct effects and do not silently alter other CV definitions.
- A deleted source reference produces an explicit alert and tag-based alternatives.
- A CV can be reopened, its sections and source references checked, and its rendered output previewed/exported.
- Invalid YAML, interrupted writes, and external changes are handled without silent data loss.
- Preview and PDF export share layout behavior, including block-level page flow.
