# CV Builder — Initial Architecture Proposal

## Proposal status

This is an initial architecture proposal for review. It separates confirmed product requirements from architectural recommendations and unresolved decisions.

It intentionally does not define Python classes, method signatures, a complete directory tree, or final third-party dependencies.

The proposal is based on:

- `AGENTS.md`.
- `docs/REQUIREMENTS_GATHERING.md`.
- `docs/agent-guides/architecture.md`.
- `docs/agent-guides/domain-model.md`.
- `docs/agent-guides/persistence.md`.
- `docs/agent-guides/rendering.md`.
- `docs/agent-guides/technology-evaluation.md`.

The current repository is a small scaffold with a `uv` project entry point and no existing application architecture. The current `pyproject.toml` requires Python `>=3.14` and includes an initial `pandas` dependency; neither should be treated as a settled technology decision for the application.

## 1. Executive summary

The recommended starting point is a local modular monolith with four clear concerns:

1. A GUI and application workflow layer.
2. A framework-independent domain layer for CV composition and source rules.
3. A workspace persistence layer for YAML, schema validation, recovery, and external-file conflicts.
4. A rendering layer that produces both preview output and final PDF output from the same resolved document model.

The most important design choice is how a saved CV detects source changes and preserves content when needed:

- An unmodified CV block keeps a reference to a source variant and the source metadata it last observed: revision, modification timestamp, and content hash.
- Editing a block and choosing **Save version** creates or updates a CV-specific local override containing a snapshot of the edited content.
- **Replace source** changes the current content under the existing source variant ID and updates its source metadata; it does not preserve immutable historical source content.
- **Save copy to source** creates a new reusable variant and makes the current CV reference it.
- A rendered PDF is a historical artifact and does not change until the user explicitly previews or renders again and confirms an overwrite.
- A changed source is detected when a CV is opened, edited, previewed, or rendered. The application warns the user; preview may continue, while final rendering/export requires confirmation.

This live-reference model fits the requirements better than immutable source history or full snapshots in every CV. Source references keep CV files small and allow source updates to be reused. Local overrides provide a deliberate snapshot only when a CV needs content independent of its source. The exported PDF remains the historical visual artifact, but it is not a structured source for reconstructing old rich text.

The architecture should remain deliberately small. It should not introduce a database, hosted backend, plugin system, or generalized event architecture. The highest-risk capabilities—rich-text editing, faithful preview/PDF output, block-level pagination, and safe multi-file workspace updates—should be validated with focused proof-of-concept work before final technology selection.

## 2. Architectural drivers

### 2.1 User-accessible, local-first persistence

Confirmed requirements:

- The application is local-first and graphical.
- The user selects the workspace location.
- Data is stored in user-accessible YAML files.
- Users may synchronize the workspace with third-party tools.
- Normal saves are explicit, while temporary recovery saves protect against crashes.

Architectural consequence:

Persistence cannot be hidden behind an application-managed opaque store. YAML files, schema versions, file fingerprints, recovery artifacts, and conflict states must be first-class concerns. The GUI must not write files directly; it should ask an application workflow to perform a validated save.

### 2.2 Data integrity under external changes

Confirmed requirements:

- YAML syntax and schema must be validated.
- Invalid data must not be silently discarded or overwritten.
- Reload is explicit and warns about unsaved edits.
- External changes combined with local unsaved changes block saving until a whole-file conflict is resolved.
- A synchronized workspace may contain missing, duplicated, partially written, stale, or unsupported-schema files.

Architectural consequence:

The persistence boundary must distinguish at least:

- Loaded and validated data.
- File metadata observed when data was loaded.
- Unsaved application state.
- Recovery state.
- External-change and conflict state.

A single-file atomic replacement can reduce corruption during one write, but it cannot make a multi-file workspace update transactional. The design therefore needs an explicit workspace consistency strategy rather than assuming that atomic file replacement solves all synchronization problems.

### 2.3 Reusable source content and independent CV behavior

Confirmed requirements:

- Canonical entries group variants.
- Variants have unique IDs and tags.
- A CV selects at most one variant from a canonical entry.
- Source updates are not silently propagated to other CVs.
- CV-specific local overrides are supported.
- Source changes are reviewed when a CV encounters a newer source revision.

Architectural consequence:

Source identity, source revision, CV selection, and CV-local content must be separate concepts. A source variant cannot double as a CV version, and a local override cannot be written back to reusable source data implicitly.

### 2.4 Template-driven presentation

Confirmed requirements:

- Templates define presentation separately from candidate content.
- Templates reference generic content types and provide section labels, ordering, nesting, and layouts.
- Sections have consistent repeated block designs.
- Users can create blank templates and edit candidate-specific template copies through the GUI.
- Formatting values come from a predefined design system.

Architectural consequence:

The domain should store semantic content independently of visual formatting. Template evaluation belongs near rendering and application composition, not inside source entries or CV business rules.

### 2.5 In-app rich-text editing

Confirmed requirements:

- Users edit content inside the application.
- Supported content formatting includes paragraphs, bullets, bold, italic, and hyperlinks.
- Fonts, font sizes, spacing, alignment, colors, borders, and layout are controlled by templates.

Architectural consequence:

The application needs a deliberate, validated rich-text representation. It should not treat arbitrary markup emitted by a GUI widget as trusted domain data. The representation must survive save/load, source variants, local overrides, preview, and PDF rendering.

### 2.6 Faithful preview and PDF rendering

Confirmed requirements:

- Preview is explicitly generated.
- Preview reflects unsaved edits.
- Preview and final PDF should use the same rendering behavior.
- PDF is the Version 1 output priority.
- Content flows across pages.
- A block moves intact to the next page when it cannot fit in the remaining space.
- Existing PDFs require overwrite confirmation.

Architectural consequence:

Preview and export should share one rendering pipeline and one pagination model. The GUI should not maintain a separate approximate layout engine. The renderer should consume a resolved, semantic CV document and produce a preview/export representation without mutating source data.

### 2.7 Cross-platform local application

Confirmed requirements:

- The application is Python-based and managed with `uv`.
- It is launched from a terminal or platform-specific script.
- Cross-platform use is desired, subject to final dependency choices.

Architectural consequence:

GUI, rich-text, PDF, font, filesystem, and packaging decisions must be evaluated together. Choosing a GUI toolkit in isolation could make faithful rendering, accessibility, or packaging difficult later.

## 3. Domain concepts and relationships

The following are conceptual boundaries, not a mandate to create one class per item.

### 3.1 Persistent concepts

#### Workspace

The user-selected root containing candidate data and starter or reusable templates. The workspace is a persistence boundary and synchronization unit. Its exact file layout remains open.

#### Candidate profile

The identity and contact information used by the CV header:

- Name.
- Optional title or credentials.
- Email.
- Optional location.
- Optional phone number.
- Repeatable social or other links.

A candidate profile is separate from reusable CV content because the same profile data can be used by multiple CV versions.

#### Canonical source entry

A stable, user-labeled identity for one real-world item or reusable content concept. Examples include one degree, one job, or one summary concept.

A canonical entry groups variants. It is not itself a piece of rendered CV text and does not replace a CV version.

#### Source variant

A reusable content version under one canonical entry. A variant has:

- A unique variant ID.
- A parent canonical-entry ID.
- A generic content type.
- Formatted content appropriate to that type.
- Tags.
- A source revision number.
- A last-modified timestamp.
- A content hash calculated from canonical semantic content.
- Archive state or lifecycle metadata.

Tags belong to variants because the user may want to find different wording for different contexts.

#### Template

A presentation definition containing:

- Section order and nesting.
- Display labels.
- Generic content-type references.
- Block layout definitions.
- Design-system choices.

Templates do not own candidate source content. A candidate-specific or CV-specific template copy can be edited without changing the starter template.

#### CV version

A saved composition for one candidate. It contains or references:

- A unique CV ID.
- Candidate identity.
- Its local template specification or template copy.
- Section composition and order.
- Selected source references.
- Source revisions last accepted for those references.
- Source revision number, last-modified timestamp, and content hash observed for those references.
- CV-specific section settings such as a nickname label.
- Local overrides.
- Ordering decisions and sorting state where required.
- Rendered PDF metadata and location.

A CV version is independent as a saved composition, but an unmodified block may still observe a source revision change when the user explicitly reviews the CV.

#### Section composition

A CV-specific use of a template section. It may contain:

- The template section reference or copied section specification.
- A local display-title override.
- Section ordering.
- Selected block references.

The section is a presentation/container concept. Its selected content comes from source variants or local overrides.

#### Block selection

A CV-specific selection of one source variant for a canonical entry, or one local override for that selection. A CV may select at most one variant from a canonical entry.

The selection stores enough source identity and comparison metadata to detect whether the source changed after the CV last observed it. The content hash is the reliable change detector; the timestamp and revision are useful for explanation and audit.

#### Local override

CV-specific content stored separately from the reusable source library. It is created when the user saves an edited block to the CV version or chooses to preserve older content after a source-change review.

An override is not directly promoted to reusable source. The user must explicitly save its content as a new source variant. If the source changed after the CV was last opened, the old structured content is not recoverable from the source reference alone; it must already exist as an override, a separate variant, or a previously exported PDF that the user can manually consult.

#### Rendered PDF artifact

The PDF produced for a CV version. It is a persistent output artifact, not the canonical domain representation of the CV. It remains unchanged when sources or templates change until the user explicitly regenerates it and confirms replacement.

### 3.2 Runtime and UI concepts

These should not be treated as persistent domain records unless a concrete requirement later demands it:

- Workspace session and selected candidate.
- Editor session and dirty state.
- Preview request and temporary preview artifact.
- Source-change review session.
- Save action choice.
- Reload confirmation.
- Recovery prompt.
- File conflict resolution state.
- GUI selection, drag-and-drop state, and dialog state.

The runtime may derive indexes such as tags, canonical-entry usage, and stale-source references from validated persisted data. Those indexes should be rebuildable rather than becoming an additional source of truth unless performance measurements justify persistence.

### 3.3 Core relationships

Conceptually:

```text
Workspace
└── Candidate
    ├── Profile
    ├── Source catalog
    │   └── Canonical entry
    │       └── Source variants
    │           └── Tags and revisions
    ├── Candidate/CV templates
    └── CV versions
        ├── Local template specification
        ├── Section compositions
        │   └── Block selections
        │       ├── Source variant reference + accepted revision
        │       └── Optional local override
        └── Rendered PDF artifact
```

This relationship allows source reuse while warning users when a referenced source changed. It also gives the application a precise place to preserve older content when the user explicitly creates a local override or source variant.

## 4. Proposed components and responsibilities

The recommended shape is a small modular monolith. The components below are boundaries of responsibility, not a final Python package tree.

### 4.1 GUI and presentation component

Responsibilities:

- Display candidates, source entries, variants, tags, templates, CV versions, and archive views.
- Provide forms and editors for supported content types.
- Display template-driven editing controls without exposing arbitrary design-system values.
- Track visible dirty state and unsaved edits.
- Request Preview, Save, Reload, Restore, Archive, Delete, and Export actions.
- Show validation errors, source-change reviews, recovery prompts, overwrite confirmations, and file conflicts.
- Provide drag-and-drop ordering where supported.

The GUI should translate user intent into application commands. It should not parse YAML, decide source-revision semantics, write files, or implement PDF layout rules.

### 4.2 Application workflow/orchestration component

Responsibilities:

- Coordinate user-facing operations such as creating a candidate, adding a source variant, creating a CV, saving an override, previewing, and exporting.
- Load validated workspace state through persistence interfaces.
- Apply domain rules and produce explicit outcomes for the GUI.
- Coordinate source-change review and the three source save actions.
- Manage dirty state, save intent, reload confirmation, recovery decisions, and overwrite confirmation at the use-case level.
- Coordinate multi-file save plans and report partial-failure recovery requirements.

This component is the boundary between UI actions and domain/persistence/rendering operations. It should not become a second domain model containing hidden business rules; rules that define valid CV behavior belong in the domain component.

### 4.3 Domain and composition component

Responsibilities:

- Represent candidate, source, variant, template, CV composition, section, block selection, local override, and revision concepts.
- Enforce invariants:
  - Stable IDs are unique within their intended scope.
  - Variants belong to canonical entries.
  - Tags belong to variants.
  - A CV selects at most one variant per canonical entry.
  - A local override cannot mutate reusable source data.
  - Source changes are not silently propagated.
  - Section labels can be overridden locally without changing the template source.
- Validate semantic content after persistence validation, such as supported content types and valid references.
- Resolve a CV composition into a document model using source content, accepted revisions, and local overrides.
- Detect stale source references based on source revision metadata.
- Apply chronological or reverse-chronological ordering and preserve manual ordering decisions.

This component should be testable without launching the GUI, reading a real workspace, or generating a PDF.

### 4.4 Workspace persistence component

Responsibilities:

- Discover and load a user-selected workspace.
- Read YAML files and validate syntax.
- Validate schema versions and structural constraints.
- Perform explicit, non-destructive migrations when supported.
- Validate cross-file references and detect duplicate or missing IDs.
- Record file fingerprints or equivalent external-change metadata.
- Write validated data using safe single-file replacement where supported.
- Manage recovery files and recovery metadata.
- Detect external modifications before reload or save.
- Provide file-level conflict choices: complete local file or complete external file.
- Preserve invalid or conflicting files for user inspection rather than overwriting them.

The persistence component should expose validated data and explicit failure states to the application workflow layer. It should not expose YAML implementation details to GUI widgets.

### 4.5 Template and design-system component

Responsibilities:

- Represent supported template section types and predefined styles.
- Resolve generic content types to section and block rendering rules.
- Validate that a template references supported content types and design-system choices.
- Create independent candidate/CV template copies from starter templates.
- Provide labels, ordering, nesting, borders, spacing, typography, and layout values to the renderer.

This component should define presentation configuration, not candidate content. It should also avoid becoming a general-purpose arbitrary template language in Version 1.

### 4.6 Rich-text/content component

Responsibilities:

- Represent supported paragraphs, bullets, bold, italic, and hyperlinks in a validated form.
- Convert between editor state and the domain representation.
- Preserve formatting through YAML persistence, preview, and PDF export.
- Sanitize or reject unsupported markup.
- Keep content semantics distinct from template-level font and layout rules.

Whether this is a separate implementation component or a cohesive part of the domain/content boundary is an open design choice. The responsibility must exist somewhere explicitly.

### 4.7 Rendering component

Responsibilities:

- Resolve the current CV composition into a semantic render model.
- Apply template section and block layouts.
- Layout paragraphs, bullets, links, dates, and skill categories.
- Flow content across pages.
- Keep a block intact when it cannot fit in the remaining space.
- Produce the preview representation and final PDF from the same rendering behavior.
- Report layout failures, including content that cannot fit even on an empty page.
- Write or return a PDF artifact without mutating source or CV data.

The renderer should not decide whether a source update is accepted or whether a CV is saved. It receives a resolved composition from the application/domain side.

### 4.8 Technology adapters

GUI, YAML parser/validator, rich-text editor, PDF renderer, font handling, and filesystem behavior are technology adapters behind the conceptual boundaries above.

The architecture should not make a specific library part of the domain or application workflow contracts until the required proof-of-concept work is complete.

## 5. Dependency direction

Recommended direction:

```text
GUI / desktop entry point
              │
              ▼
Application workflows
       │       │       │
       │       │       └── Rendering interface ──► Rendering adapter
       │       └────────── Persistence interface ─► YAML/filesystem adapter
       ▼
Domain and composition rules
       ▲
       │
Template/design and rich-text representations
```

The diagram is conceptual. The important rule is that dependencies point toward stable behavior and data concepts, not toward GUI or file-format details.

### Allowed dependencies

- The GUI may depend on application workflow interfaces and view models.
- Application workflows may depend on domain rules and persistence/rendering interfaces.
- Persistence adapters may depend on YAML libraries, filesystem APIs, schema validators, and the domain serialization boundary.
- Rendering adapters may depend on PDF/layout technology and the resolved render model.
- Template and rich-text representations may be consumed by domain composition and rendering, subject to a clear ownership decision.
- The application entry point may assemble concrete adapters.

### Dependencies to avoid

- Domain rules importing GUI widgets.
- Domain rules reading or writing YAML files.
- GUI event handlers writing YAML directly.
- PDF layout code modifying source entries or CV definitions.
- Persistence code deciding whether a user intended Replace source, Save copy, or Save version.
- A global workspace singleton shared by unrelated screens.
- A renderer reaching into the filesystem to discover source content.
- Circular dependencies between source content, templates, and CV versions.
- A generic event bus or plugin framework introduced before a concrete need exists.

These restrictions keep domain behavior testable and prevent a UI concern, file format, or renderer from becoming the accidental owner of product rules.

## 6. Critical workflow traces

### 6.1 Creating and saving reusable source content

1. The user opens a candidate and chooses a generic content type.
2. The GUI requests the existing tag suggestions and source entries from the application workflow layer.
3. The user creates a canonical entry and its first variant, or creates another variant under an existing canonical entry.
4. The editor produces the validated rich-text/content representation.
5. The application workflow validates required fields and domain invariants.
6. The persistence component checks whether the relevant files changed externally since they were loaded.
7. If there is a conflict, saving stops and the GUI offers the complete-local-file versus complete-external-file choice.
8. If there is no conflict, the persistence component validates the resulting YAML and schema version.
9. The workspace save writes the relevant file safely and updates the source revision number, modification timestamp, and canonical-content hash.
10. Recovery data for the successfully saved operation can be cleared or marked superseded.
11. The GUI refreshes source indexes and shows the new canonical entry/variant and tags.

The domain does not know whether the data came from YAML or another future persistence mechanism. The persistence component does not decide what a valid Work Experience or Summary means; it delegates semantic validation to the domain boundary.

### 6.2 Editing an entry used by a CV and choosing a save action

1. The user opens a CV version and edits a selected block.
2. The application resolves the block from its local override if one exists; otherwise it resolves the referenced source variant at the CV's recorded revision.
3. The user finishes the block. Cancelling discards the in-progress edit.
4. On Save, the application first validates the edited block and saves it to the current CV context as a local override.
5. The application checks the stored source metadata against the current source metadata if the block still has a source reference.
6. If the source changed, the GUI warns the user. Preview may continue, but final rendering/export requires confirmation.
7. The user chooses one of the existing actions:
   - **Save version:** create or update the local override. The source and other CVs do not change.
   - **Replace source:** replace the current content under its existing variant ID, update its revision/timestamp/hash, and leave other CVs referencing that source. They will warn when they next observe the changed source.
   - **Save copy to source:** create a new variant ID from the edited content and switch the current CV reference to that new variant. The original source and other CVs do not change.
8. The application persists the affected CV and/or source files through the workspace persistence boundary.
9. If multiple files must change, the application uses a coordinated save plan and provides recovery information if the operation cannot complete consistently.

The three actions must remain semantically distinct. A GUI label or shortcut must not hide whether reusable source data is being modified.

### 6.3 Saving and reopening a CV without silently changing its content

1. The user explicitly saves a CV version.
2. The CV definition stores its template specification, section composition, source references, accepted source revisions, ordering, and local overrides.
3. The rendered PDF, if any, remains the previously generated artifact.
4. Another user action changes a referenced source variant.
5. The CV definition and its existing PDF are not automatically rewritten.
6. When the CV is later opened, edited, previewed, or rendered, the application compares the recorded source revision/timestamp/hash with current source metadata.
7. If a reference is changed, the application displays a warning identifying the affected source and CV block.
8. Preview may continue using the current source content after the warning.
9. Final rendering/export requires confirmation after the warning.
10. If the user wants to preserve the previously used structured content, that content must already exist as a local override or separate source variant. The source reference alone cannot reconstruct it after replacement.

This workflow intentionally distinguishes a saved CV definition from a rendered PDF. A PDF is reproducible as a visual artifact until overwritten; a reference-based CV follows the current source after warning and confirmation, while a local override or separate variant preserves structured content.

### 6.4 Previewing and exporting a multi-page PDF

1. The user clicks Preview.
2. The application gathers the current editing state, including unsaved GUI edits.
3. It resolves source references, local overrides, accepted revisions, section order, block order, and template settings.
4. It checks for changed source metadata and displays a warning. Preview may proceed; final rendering/export requires confirmation.
5. The rendering component converts the resolved composition into a semantic render model.
6. The layout engine applies the predefined design system and places blocks across pages.
7. If a block cannot fit in the remaining space, the layout engine moves the complete block to the next page.
8. The same rendering behavior produces the preview and the final PDF.
9. Preview generation does not save the CV definition or create a new CV version.
10. On export or render, the application asks for confirmation before replacing an existing PDF.
11. The persistence/output boundary writes the new PDF beside the CV definition and reports failures without changing the CV definition.

The renderer should explicitly report the exceptional case of a block larger than an entire page. The current requirements define what happens when a block does not fit in the remaining space, but do not yet define how an intrinsically oversized block should be handled.

### 6.5 Malformed YAML, failed writes, and external file changes

#### Opening or reloading malformed YAML

1. The persistence component reads the file without exposing it to the domain.
2. It parses YAML and validates the schema version and structure.
3. If parsing or validation fails, the application identifies the affected file and field or location where possible.
4. The invalid file is preserved.
5. The GUI offers a recovery or correction path; it does not silently replace the file with defaults.

#### Failed or interrupted writes

1. The application validates the complete intended file content before writing.
2. It writes through a temporary file and uses an atomic replacement strategy where supported.
3. It records enough recovery information to identify an incomplete multi-file operation.
4. If the write fails, the original valid file remains intact where the platform permits.
5. The GUI reports the failed operation and does not claim the save succeeded.

#### External changes and conflicts

1. The persistence component records file metadata when a file is loaded or last saved.
2. Before Save or Reload, it checks whether the file changed externally.
3. Reload is explicit and warns when local unsaved edits may be lost.
4. If both local unsaved edits and external changes exist, Save is blocked.
5. The GUI displays the conflict and offers whole-file local or whole-file external selection.
6. The selected file is validated before it becomes active application state.
7. Field-level merging is not attempted in Version 1.

Because third-party synchronization tools do not guarantee locking or ordering, the workspace layer should also detect duplicate/conflict files and surface them rather than assuming the newest timestamp is correct.

## 7. Risks and alternatives

### 7.1 Source references versus snapshots

Recommendation: use source references plus source revisions and explicit CV-local overrides.

Alternative: snapshot all selected content into every CV.

- Benefit: simple historical reproduction.
- Cost: duplicates reusable content, makes source improvements harder to discover, and conflicts with the user's desire to avoid many copied versions.

Alternative: store only source IDs.

- Benefit: minimal storage and straightforward reuse.
- Cost: a previously saved CV cannot be reproduced after a source update unless the old PDF still exists; it also creates unsafe silent-change risks.

The recommended combination is the smallest model that supports both reuse and deliberate preservation.

### 7.2 Multi-file YAML workspace versus one YAML document

Recommendation: keep the user-visible workspace organized into multiple conceptual files/folders, while defining a clear consistency boundary for operations that touch more than one file.

Alternative: one large workspace YAML file.

- Benefit: simpler whole-workspace save and conflict detection.
- Cost: poor isolation between candidates/CVs, larger diffs, more frequent conflicts, and less convenient manual inspection.

Alternative: a local database with YAML import/export.

- Benefit: stronger transactional and query behavior.
- Cost: violates the requirement that user-accessible YAML is the primary workspace representation and complicates third-party file synchronization.

The requirements favor user-accessible YAML, but the exact file partition and multi-file transaction/recovery strategy remain an important design decision.

### 7.3 GUI technology and rich-text editing

Recommendation: choose only after a small proof of concept demonstrates the required editor behavior and cross-platform packaging.

Options to compare include:

- A native Python GUI toolkit with its native rich-text controls.
- A desktop webview approach with a browser-based editor.
- A more specialized cross-platform GUI toolkit with rich-text support.

Evaluation criteria:

- Paragraphs, bullets, bold, italic, and hyperlinks.
- Reliable conversion to a validated internal representation.
- Accessibility and keyboard editing.
- Cross-platform behavior and `uv`-based development.
- Preview integration and packaging.
- Licensing, maintenance, and testability.

The main trade-off is between native desktop integration and the maturity/flexibility of browser-oriented rich-text editing. A popular toolkit should not be selected without demonstrating the required editing and serialization behavior.

### 7.4 PDF/rendering technology

Recommendation: prototype the render model and block-level pagination before selecting the final renderer.

Options to compare include:

- Direct PDF/layout libraries.
- HTML/CSS-to-PDF pipelines.
- A shared document-layout engine with separate preview and PDF adapters.

Evaluation criteria:

- Exact preview/PDF consistency.
- Keep-block-together pagination.
- Hyperlinks and typography.
- Fonts and cross-platform reproducibility.
- Multi-page layout diagnostics.
- Accessibility and PDF metadata.
- Licensing, maintenance, and testability.

HTML/CSS may offer rich layout features, while a direct PDF engine may offer more deterministic control. Either approach can fail the requirement if preview uses a materially different layout path from export.

### 7.5 Rich-text representation

Recommendation: use a small validated representation for the supported formatting rather than persisting arbitrary GUI markup directly.

Possible approaches include:

- A restricted HTML-like representation.
- Markdown plus explicit inline marks and links.
- A purpose-built structured text representation.

The trade-off is expressiveness versus validation and renderer control. Version 1 needs only paragraphs, bullets, bold, italic, and hyperlinks, so a constrained representation is preferable to a general document format.

### 7.6 Multi-file consistency and third-party synchronization

Risk: a user action such as Save copy to source may update a source file and a CV file. The filesystem can succeed for one and fail for the other, or a synchronization tool can observe an intermediate state.

Recommendation: represent multi-file saves as explicit save plans, validate all affected files first, write safely, and retain recovery metadata for incomplete operations. A later implementation can choose whether a small journal, manifest, or pending-operation file is necessary; the architecture should not assume that individual atomic writes provide workspace-wide atomicity.

### 7.7 Schema evolution and external editing

Risk: users may synchronize or manually edit YAML while the application evolves its schema.

Recommendation: keep schema versions explicit, make migrations non-destructive and testable, and retain backups or migration recovery paths. Unknown fields and unsupported versions need an intentional policy rather than automatic deletion.

### 7.8 Template scope

Risk: the template editor can expand into a general layout-authoring system, undermining Version 1 simplicity.

Recommendation: implement only predefined design-system choices and known semantic section/block layouts. Treat advanced template authoring as a separate future capability.

## 8. Open decisions

These are unresolved product or technology decisions, not silently chosen by this proposal:

1. Official operating-system support for Version 1.
2. Supported Python version. The current scaffold says `>=3.14`, but this should be confirmed as a product/toolchain decision.
3. Mandatory fields for every content type, including the exact Work Experience and Education validation matrix.
4. Whether email is required or merely recommended in a candidate profile.
5. Header behavior when social links do not fit on the intended line.
6. Starter templates and predefined CV types shipped with Version 1.
7. The exact predefined design-system choices.
8. The visual and interaction design for source-change review and differences.
9. Whether a later version needs a central stale-reference view across CVs.
10. Whether CV dates mean creation date, last-modified date, or both.
11. PDF file naming conventions.
12. Exact workspace file partition and the multi-file save/recovery strategy.
13. Behavior for missing, duplicated, partially written, or synchronization-conflict files.
14. Accessibility requirements for the GUI and generated PDFs.
15. Behavior when a single block is taller than an entire page.
16. The GUI, rich-text, YAML validation, and PDF technology choices after proof-of-concept evaluation.

## 9. Recommended next steps

1. Review and approve the content-preservation model: source reference plus revision, CV-local override, and rendered PDF artifact.
2. Decide the workspace file partition and the minimum multi-file save/recovery mechanism.
3. Define the initial domain invariants and example YAML fixtures without committing to final GUI or rendering libraries.
4. Run a rich-text/editor proof of concept covering paragraphs, bullets, bold, italic, hyperlinks, load/save round trips, and accessibility.
5. Run a rendering proof of concept covering the same content, preview/PDF consistency, hyperlinks, font handling, multi-page flow, keep-block-together behavior, and oversized blocks.
6. Run a persistence proof of concept covering malformed YAML, schema versions, safe writes, recovery files, external changes, and whole-file conflicts in a temporary workspace.
7. Compare technology options using the criteria in `docs/agent-guides/technology-evaluation.md` and record evidence before selecting dependencies.
8. After the proof of concepts, produce a detailed design for the approved boundaries and a test plan for the required workflows.

### Five most important decisions before detailed design

1. Approve the source-reference, revision, local-override, and rendered-artifact behavior for saved CVs.
2. Choose the workspace file partition and how multi-file saves recover from partial failure.
3. Confirm supported operating systems and the Python/toolchain support policy.
4. Select the GUI/rich-text approach after validating editor behavior and packaging.
5. Select the rendering/PDF approach after validating preview fidelity and block-level pagination.
