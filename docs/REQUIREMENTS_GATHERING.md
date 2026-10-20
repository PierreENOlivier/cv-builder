# CV Builder — Requirements Gathering

## Document status

This document captures the requirements discovered during the initial product discussion. It describes the intended Version 1 behavior and records assumptions or decisions that still need to be finalized.

No implementation architecture, module structure, class hierarchy, framework, or database choice is prescribed here.

## Problem statement

Creating and adapting CVs currently takes unnecessary time because useful content from previously created CVs is difficult to reuse. The main sources of wasted effort are:

- Adding and removing sections and entries.
- Making small wording changes to descriptions.
- Keeping formatting consistent after content changes.

The application should let users maintain reusable CV content, assemble independent CV versions, edit them inside the application, and produce a consistently formatted PDF without requiring Word or another external editor.

## Product scope for Version 1

Version 1 is a local-first graphical desktop application for building CVs. It is intended to be used by multiple candidates, with each candidate having an independent collection of source content, templates, CV versions, overrides, and rendered documents.

The primary CV type is **Industry**. The design should allow a small set of additional predefined CV types later, but Version 1 should focus on Industry CVs.

## Confirmed functional requirements

### Workspace and candidates

- The application is a local graphical desktop application.
- The user selects where the workspace is stored.
- The workspace data is accessible as YAML files.
- The user may store the workspace wherever they choose, including a folder synchronized by a third-party tool.
- The application remembers the last-used workspace and offers it on startup.
- The user can select a different workspace.
- One workspace can contain multiple candidates.
- Each candidate has an independent folder tree.
- Candidate creation is performed through the graphical interface.
- Candidate creation prompts for:
  - Name: required.
  - Title or credentials: optional single text field, for example `Ph.D., MSc`.
  - Email: expected candidate profile field.
  - Location: optional.
  - Phone number: optional.
  - Social and other links: a repeatable list.
- Each link may contain display text, a URL, and optionally an icon.
- The header uses three conceptual lines:
  1. Candidate name and title/credentials in a large font.
  2. Location, email, and phone.
  3. Social and other links.

### YAML storage and reliability

- Source content, templates, CV definitions, overrides, and related data are stored in user-accessible YAML files.
- YAML files include an explicit schema version.
- The application validates both YAML syntax and the expected data structure.
- Invalid or malformed YAML must produce an understandable error identifying the problem where possible.
- The application must not silently discard or overwrite invalid data.
- Users explicitly click Save to write normal changes to the main YAML files.
- The application may create temporary recovery files through automatic interim saves.
- Recovery files exist to protect against crashes and do not replace explicit Save behavior.
- After a crash, the application offers to recover available edits.
- Reloading externally changed data is explicit. The user clicks Reload.
- Reload requires confirmation when unsaved application edits might be lost.
- If a YAML file changed externally while local unsaved changes also exist, saving is blocked until the conflict is resolved.
- Conflict resolution is file-level in Version 1: the user chooses the complete local file or the complete external file.
- Field-by-field merging is out of scope for Version 1.

### Source library

The source library contains reusable candidate content. It is separate from the presentation templates.

Supported or required source content includes:

- Summary text.
- Work Experience.
- Education.
- Portfolio items.
- Skills.
- Custom sections.

Source content is organized through the following concepts:

- **Canonical entry:** a user-labeled underlying item with a stable unique ID. Example: one particular bachelor’s degree.
- **Variant:** an alternative version of a canonical entry, with its own unique ID and tags.
- **Local override:** CV-specific YAML content that overrides a referenced source item without changing the source library.

Requirements:

- Canonical entries have user-defined labels.
- Variants are grouped under their canonical entry in the user interface.
- A variant has its own unique ID.
- Tags belong to variants, not canonical entries.
- Users can reuse existing tags when adding tags, while also creating new tags.
- Search can filter variants by generic content type and tags.
- Multiple selected tags use OR matching.
- Results matching all selected tags are shown first.
- A CV version may select at most one variant from a canonical entry.
- Different real-world items are represented by different canonical entries.
- Users can create a new canonical entry and its first variant directly from the CV-building workflow.
- Users can archive obsolete source entries.
- Archived entries are excluded from normal selection by default.
- Archived entries have a separate view.
- Archived entries can be restored.
- Permanent deletion is available from the archive view and requires confirmation.

### Source save actions

When working on a CV, the current block is always saved to the current CV version first. The user may then explicitly choose how source content should be handled:

- **Save version:** save the current content only to the current CV version as a local override. The source entry is not changed.
- **Replace source:** update the existing reusable source variant under the existing variant ID and increment its source revision. Other CV versions keep their existing source reference and revision until they are reviewed.
- **Save copy to source:** create a new reusable source variant with a new ID and make the current CV version reference that new variant.

Local overrides are not directly promoted. If the user wants to reuse an override, they save it as a new source variant.

These actions have the following consequences:

- Saving a version never changes the reusable source library.
- Replacing a source changes the source library but does not immediately rewrite any other CV definition or rendered PDF.
- Saving a copy creates a new source variant; it does not modify the original variant or other CV versions.
- A local override is specific to one CV version and is not visible as a reusable source variant.
- A local override can be removed if the user wants the CV to use its referenced source variant again.

### Source references, revisions, and historical behavior

CV versions use a combination of references and local materialized content:

- An unmodified CV block stores a reference to a source variant and the source revision used by that CV.
- A block edited and saved to the CV stores its content in a CV-specific local override. It no longer depends on later changes to the referenced source variant until the user removes the override.
- A CV section stores its template/reference information and any CV-specific section settings, such as a local section-title nickname.
- A rendered PDF is a saved historical artifact. It remains unchanged after a source or template change until the user explicitly previews or renders the CV again and confirms any overwrite.
- A CV definition that still references a source variant is not silently rewritten when that source changes.
- If the referenced source revision is newer than the revision recorded in the CV, the CV is considered stale for that block.
- The application detects a stale reference when the user opens the relevant editor, previews, or renders the CV. It must prompt the user before applying the newer source content.
- The prompt must identify the affected source and CV content and provide an explicit choice.
- Keeping the previous CV content creates or retains a local override based on the previously used source revision.
- Accepting the newer source updates the CV's recorded source revision and uses the newer source content.
- Replacing the source from the editor updates the source variant and its revision. The currently edited CV may use the replacement immediately; other CVs are not changed until they are reviewed.
- A CV can therefore reproduce its previously used content through its rendered PDF or through a local override, but a source reference by itself intentionally follows source revisions after the user reviews and accepts the change.

#### Required source-change scenarios

The following behavior must be testable:

1. Two CVs reference the same source variant at revision 1.
2. The user chooses **Replace source** while editing CV A. The source becomes revision 2.
3. CV B's YAML reference remains unchanged and its existing rendered PDF remains unchanged.
4. When CV B is opened, previewed, or rendered, the application reports that its source reference is stale and asks how to proceed.
5. If the user keeps the previous content, CV B receives a local override and remains stable even if the source changes again.
6. If the user accepts the newer source, CV B records revision 2 and uses that source content.
7. If the user chooses **Save copy to source**, a new variant ID is created, the original variant remains unchanged, and the current CV references the new variant.
8. If the user chooses **Save version**, only the current CV's local override changes; no source entry or other CV changes.

### Content types

#### Work Experience

Each Work Experience entry supports:

- Job title.
- Location, which may contain a combined display value such as `Google, Germany`.
- Optional start date.
- Optional end date.
- An ongoing role may display text such as `Current` instead of an end date.
- Description bullets.

Dates are stored as dates when provided, while the template controls their display format.

#### Education

Each Education entry supports:

- Optional start date.
- Graduation/end date.
- Diploma or degree title.
- University and country, stored as a free-form display value such as `University, Country`.
- Optional descriptive paragraph or bullet points.

#### Summary

- Summary is a dedicated section.
- It contains a text block with paragraph content.
- Summary text supports variants and tags.
- The summary is rendered inside a bordered section layout.
- Summary content is paragraph-based rather than bullet-based.

#### Portfolio

- Portfolio is optional.
- It contains bullet points.
- Portfolio bullets can include hyperlinks to projects.

#### Skills

- Skills are a dedicated section with their own layout.
- Skills are source entries and can have variants and tags.
- A skill entry contains a category label and formatted content.
- Example:

  - Label: `Programming`
  - Content: `Python [numpy, scikit-learn], R [tidyverse, ggplot], SQL for Databases, Git for Code versioning`

- Skills are displayed as categorized keywords/content rather than dated entries or ordinary description bullets.

#### Custom sections

- Users can create custom sections with any title.
- A custom section can contain free text or bullets.
- Predefined custom layouts may be provided for common cases such as Personal Interests.
- Custom sections should use the predefined design system rather than unrestricted formatting.

### Templates

Templates define presentation and structure separately from candidate source content.

- A template defines section order, section labels, nesting, and rendering rules.
- A template section references a generic content type.
- The template label can differ from the generic type. For example, a section referencing `Work Experience` may be labeled `Academic Positions`.
- Sections are primarily template-defined containers, not reusable candidate source entries.
- A CV version may locally override a section’s displayed title, such as using `Experience` instead of `Work Experience`.
- Template sections contain a consistent block design.
- The application repeats that block design for each selected block in the section.
- Work Experience and Education use repeated entry layouts.
- Skills use a dedicated repeated category layout.
- Summary uses a single paragraph text block.
- Templates use a predefined design system for fonts, font sizes, spacing, alignment, colors, borders, and layouts.
- Users choose from supported design options rather than entering arbitrary formatting values.
- Users can edit templates through the GUI.
- Users can create a blank template through the GUI.
- A blank template allows users to add sections and choose their generic content types.
- Starter templates are provided so users do not need to start from nothing.
- Editing a starter template creates a unique modified copy.
- A candidate-specific template copy does not modify the original starter template or templates used by other candidates.
- A separate advanced template-authoring feature may be added later.

### CV versions

- Users can create multiple CV versions for a candidate.
- Versions are independent saved compositions.
- A new CV can be created from the source library.
- A new CV can also be created by duplicating an existing CV version.
- A duplicated CV can use a different template.
- Users can explicitly choose to save changes as a new version or replace the current version.
- Each version has a unique ID.
- Version names are not required for Version 1.
- Dates are associated with versions; the exact date semantics remain open.
- A CV version references selected source blocks and sections.
- A CV version can contain CV-specific local overrides.
- Blocks and sections can be added or removed from a version.
- Optional sections such as Portfolio and Personal Interests can be omitted.
- A source block selected for a CV is unique within that CV version; the same canonical entry cannot contribute multiple variants to the same version.
- Source updates do not immediately rewrite other CV definitions or rendered PDFs.
- When a CV is opened or its content is edited, previewed, or rendered, the application detects whether a referenced source revision has changed.
- A stale source reference requires an explicit user decision before the newer source content is applied.
- The application presents options such as keeping the previous content through a local override, accepting the newer source, replacing the source, or creating a source variant.
- Source changes are not silently propagated to other CV versions.

### Editing and formatting

- Users edit blocks inside the application.
- A block must be completed before it can be saved.
- Optional fields may remain empty.
- Incomplete blocks can be cancelled and discarded.
- Cancelled blocks are not saved to the source library or CV version.
- Users can apply content-level:
  - Bold.
  - Italic.
  - Hyperlinks.
- Fonts, font sizes, alignment, spacing, and layout are controlled by the selected template.
- Direct arbitrary font and layout customization is out of scope for Version 1.
- Undo and redo are not required for Version 1.

### Ordering

- Sections can be reordered through drag-and-drop if feasible.
- Blocks can be reordered through drag-and-drop within a section.
- Manual block ordering overrides automatic date sorting.
- Work Experience and Education support sorting by date.
- Sorting supports chronological and reverse-chronological order.
- Sorting is applied when the user presses a sort action; it does not continuously override manual ordering.

### Preview, pagination, and PDF export

- The user explicitly clicks Preview to generate a preview.
- Preview reflects the current editing state, including unsaved edits.
- Preview uses the same rendering behavior as final PDF export.
- The preview should look like the final PDF.
- PDF is the Version 1 export priority.
- Users complete the CV workflow inside the application without requiring Word or another editor.
- PDF content flows automatically across multiple pages.
- A block should not be split if it cannot fit entirely in the remaining space.
- Such a block moves to the next page.
- Following blocks may continue on that page or later pages.
- Manual page-break management is not required for Version 1.
- Each CV has a rendered PDF stored alongside its definition, local template, and overrides.
- Re-rendering overwrites the existing PDF only after warning the user and receiving confirmation.
- Users can copy rendered PDFs elsewhere if they want to retain historical copies.

### CV folder contents

The candidate workspace should contain a structure conceptually equivalent to:

- Reusable source content.
- CV definitions/versions.
- The local template associated with each CV.
- CV-specific local overrides.
- The rendered PDF for each CV.

The exact YAML file names and directory layout remain implementation details to be finalized.

## Confirmed constraints

- Local-first operation.
- User-controlled file location.
- User-accessible YAML files.
- `uv` manages Python and project dependencies.
- A README documents installation, the supported Python version, and usage.
- The GUI is launched from the terminal or a platform-specific script.
- The application should be suitable for cross-platform use, subject to the final dependency and packaging choices.
- No hosted service, account system, or required database is part of Version 1.
- The application should avoid silently changing user data.
- Explicit user confirmation is required for destructive or potentially lossy actions.

## Explicitly out of scope for Version 1

- OCR extraction from existing CV documents.
- Automatic classification of OCR-extracted content.
- Application tracking or an archive of which CV was sent to which application.
- DOCX export as the primary output.
- Unrestricted template authoring.
- A separate advanced template-creation application.
- Field-by-field synchronization conflict resolution.
- Required undo/redo support.
- Manual page-break management.
- Automatic propagation of source changes to all CV versions without user review.
- Mandatory cloud service, account, or hosted database.

## Important assumptions

- The candidate profile is stored separately from reusable CV content and is used by the header template.
- Email is expected to be required in the candidate profile, while location and phone are optional.
- Work Experience and Education have type-specific validation rules, but the exact required-field matrix is not yet finalized.
- The application will use a GUI editor capable of handling paragraphs, bullets, bold, italic, and hyperlinks.
- YAML is primarily edited by the application, but users are intentionally allowed to inspect, back up, synchronize, and potentially modify it externally.
- The source-change workflow will require source revision metadata and references from CV definitions to source IDs and revisions.
- A local override is version-specific and is represented separately from the reusable source entry.

## Open questions to resolve before implementation

1. Which operating systems must Version 1 officially support?
2. Which Python version should Version 1 support?
3. Which fields are mandatory for each content type, especially Work Experience and Education?
4. Is email mandatory in the candidate profile, or merely recommended?
5. Should social links wrap onto additional header lines when they do not fit, or should the app warn the user?
6. What starter templates and predefined CV types should ship with Version 1?
7. Which predefined font, size, spacing, color, and layout choices belong in the design system?
8. How should source-change review present differences to the user?
9. Should a later version provide a central view of all CV versions with stale source references?
10. What exact date semantics should CV versions store: creation date, last-modified date, or both?
11. What should the PDF file naming convention be?
12. How should the application behave if a synchronized workspace contains a missing, duplicated, or partially written file?
13. Which accessibility requirements apply to the GUI and generated PDF?

## Architectural implications without prescribing an implementation

- The data model must distinguish generic content meaning from template presentation.
- Stable IDs are needed for candidates, canonical entries, variants, sections, blocks, CV versions, templates, and local overrides.
- Source variants need parent-child relationships to their canonical entries.
- CV definitions need references to source IDs and the source revisions used, plus local overrides for edited CV-specific content.
- Saved CV behavior must distinguish live source references, CV-local overrides, and rendered PDF artifacts.
- Templates need references from presentation sections to generic content types.
- The rendering process must understand semantic content types and their different layouts.
- The editor needs a controlled rich-text representation for bold, italic, hyperlinks, paragraphs, and bullets.
- YAML validation and schema versioning are core product capabilities, not incidental implementation details.
- Save, reload, recovery, and conflict resolution need explicit state handling.
- PDF rendering must support block-level pagination so blocks are not split across pages.
- The local-first design requires careful handling of atomic writes, temporary recovery files, and externally changed files.

## Suggested Version 1 acceptance themes

Version 1 should be considered functionally complete when a user can:

1. Create or open a workspace.
2. Create multiple candidates with independent data.
3. Create a candidate profile and manage repeatable links.
4. Add reusable source entries and variants with tags.
5. Search source variants by type and tags.
6. Create a CV from a starter or blank template.
7. Add, remove, and reorder sections and blocks.
8. Select source variants and create CV-specific overrides.
9. Edit content inside the GUI with the supported formatting controls.
10. Save source content and CV versions explicitly.
11. Recover edits after a simulated crash.
12. Detect malformed YAML and schema-version problems.
13. Detect external file changes and resolve file-level conflicts.
14. Generate a faithful preview.
15. Render a multi-page PDF without splitting a block across pages.
16. Save the rendered PDF beside the CV definition after confirmation when overwriting.
