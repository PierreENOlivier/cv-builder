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

Version 1 is a local-first graphical desktop application for building CVs. It is intended to be used by multiple candidates, with each candidate having an independent collection of Source content, Styling files, CV Composer files, and rendered documents.

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

- Source content, Styling files, CV Composer files, overrides, and related data are stored in user-accessible YAML files.
- A Styling file stores appearance configuration.
- A CV Composer file stores the composition for one CV version, including its Styling reference, Source references, ordering, and local overrides.
- A Source file stores reusable candidate sections, blocks, canonical entries, variants, tags, and source metadata.
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

### Source

The Source contains reusable candidate content. It is separate from the presentation Styling.

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

When editing a block in a CV, the application keeps the changes as an in-memory draft until the user chooses an explicit save action:

- **Save version:** save the current content only to the current CV version as a local override. The source entry is not changed.
- **Replace source:** update the existing reusable source variant under the existing variant ID, increment its source revision, and update its last-modified timestamp and content hash. Other CV versions keep their existing source reference metadata and receive a warning when the change is detected.
- **Save copy to source:** create a new reusable source variant with a new ID and make the current CV version reference that new variant. If the current CV block has a local override, remove that override so the CV uses the new source variant.

Local overrides are not directly promoted. If the user wants to reuse an override, they save it as a new source variant.

These actions have the following consequences:

- Saving a version never changes the reusable Source.
- Replacing a source changes the Source but does not immediately rewrite any other CV Composer or rendered PDF.
- Saving a copy creates a new source variant; it does not modify the original variant or other CV versions. The current CV switches from its local override, if any, to the new variant.
- A local override is specific to one CV version and stored inside the CV composer file. Consequently, it is not visible as a reusable Source variant.
- A local override always takes precedence over its referenced source variant.
- A source change may still produce a warning for a block with a local override, but it does not change that block's rendered content while the override exists.
- A block with a local override displays an indicator showing that its content differs from the reusable source.
- A local override can be removed if the user wants the CV to use its referenced source variant again.

### Source references, revisions, and historical behavior

CV versions use a combination of references and local materialized content:

- An unmodified CV block stores a reference to a source variant and the source metadata observed by that CV: source revision, last-modified timestamp, and content hash.
- A block edited and saved to the CV stores its content in a CV-specific local override. It no longer depends on later changes to the referenced source variant until the user removes the override.
- A local override always wins over the referenced source variant. The application may warn when the underlying source changes, but the override remains the content used for preview and rendering.
- A CV section stores its CV Composer reference and any CV-specific section settings, such as a local section-title nickname.
- A rendered PDF is a saved historical artifact. It remains unchanged after a Source or Styling change until the user explicitly previews or renders the CV again and confirms any overwrite.
- A CV definition that still references a source variant is not silently rewritten when that source changes.
- If the current source revision, timestamp, or content hash differs from the metadata recorded in the CV, the CV block is considered outdated.
- The application detects an outdated reference when the user opens the relevant editor, previews, or renders the CV.
- The application displays a warning identifying the affected source and CV content.
- Preview may continue using the current source content after the warning.
- Final rendering/export requires confirmation after the warning.
- A block with a local override may display a source-change warning, but its preview and rendering continue to use the override.
- The source reference alone cannot reconstruct the previously used structured content after the source changes. To preserve that content, the user must already have a local override or separate source variant; the exported PDF remains available as visual history.
- Replacing the source updates the source variant's revision, timestamp, and content hash. Other CVs retain their recorded metadata and are warned when they later detect the change.

#### Required source-change scenarios

The following behavior must be testable:

1. Two CVs reference the same source variant at revision 1.
2. The user chooses **Replace source** while editing CV A. The source becomes revision 2.
3. CV B's YAML reference remains unchanged and its existing rendered PDF remains unchanged.
4. When CV B is opened, previewed, or rendered, the application warns that its source reference is outdated.
5. CV B may be previewed using the current source content, but final rendering/export requires confirmation.
6. If the user wants to preserve the previous structured content, CV B must already have a local override or separate source variant; the source reference alone is not sufficient.
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

Dates are stored as dates when provided, while the selected Styling controls their display format.

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

### Styling

Styling files define appearance separately from candidate Source content and CV Composer references.

- A Styling file defines fonts, font sizes, spacing, alignment, colors, borders, and layout rules.
- Built-in Styling files are read-only assets.
- User-created Styling copies may be writable and reusable by other CV Composers.
- The application discovers and validates Styling files before offering them for selection.
- A valid changed Styling file affects future previews and renders of CVs that reference it.
- Existing rendered PDFs remain unchanged until explicitly regenerated and overwritten.
- Users choose from supported design options rather than entering arbitrary formatting values.
- Styling editing is not part of the normal CV editor.
- An advanced feature for creating or editing custom Styling files may be added later.

### CV Composer

A CV Composer is the YAML file for one CV version. It defines how Source content is assembled, defines local overrides and which Styling file is used.

- A CV Composer references a Styling file.
- It references Source sections, blocks, canonical entries, and variants.
- It defines section order, section labels, nesting, selected blocks, and local overrides.
- The displayed section label can differ from the generic content type. For example, a section referencing `Work Experience` may be labeled `Academic Positions`.
- Sections are CV Composer containers for selected Source content, not reusable Source entries.
- A CV version may locally override a section’s displayed title, such as using `Experience` instead of `Work Experience`.
- Each section uses the consistent block design provided by the selected Styling.
- The application repeats that design for each selected block in the section.
- Work Experience and Education use repeated entry layouts.
- Skills use a dedicated repeated category layout.
- Summary uses a single paragraph text block.
- Users can create a blank CV Composer and add sections by choosing their generic content types.
- A CV Composer is independent from the Styling file and the Source content it references.

### CV versions

- Users can create multiple CV versions for a candidate.
- Versions are independent saved compositions.
- A new CV can be created from the Source.
- A new CV can also be created by duplicating an existing CV version.
- A duplicated CV can use a different Styling file and CV Composer configuration.
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
- When a CV is opened or its content is edited, previewed, or rendered, the application detects whether a referenced source revision, timestamp, or content hash has changed.
- An outdated source reference produces a warning. Preview may continue, while final rendering/export requires confirmation.
- The application presents the source-change warning and identifies the affected source and CV content. Preserving previous structured content requires an existing local override or separate source variant.
- Source changes are not silently propagated to other CV versions.

### Editing and formatting

- Users edit blocks inside the application.
- A block must be completed before it can be saved.
- An edited block remains an in-memory draft until the user explicitly chooses Save version, Replace source, or Save copy to source.
- Every edited block must be explicitly saved or cancelled before the CV can be previewed or rendered.
- Optional fields may remain empty.
- Incomplete blocks can be cancelled and discarded.
- Cancelled blocks are not saved to the Source or CV version.
- Users can apply content-level:
  - Bold.
  - Italic.
  - Hyperlinks.
- Fonts, font sizes, alignment, spacing, and layout are controlled by the selected Styling file.
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
- Each CV has a rendered PDF stored alongside its CV Composer file and local overrides.
- Re-rendering overwrites the existing PDF only after warning the user and receiving confirmation.
- Users can copy rendered PDFs elsewhere if they want to retain historical copies.

### CV folder contents

The candidate workspace should contain a structure conceptually equivalent to:

- Reusable source content.
- CV definitions/versions.
- The Styling reference associated with each CV Composer.
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
- Unrestricted Styling authoring.
- A separate advanced Styling-creation application.
- Field-by-field synchronization conflict resolution.
- Required undo/redo support.
- Manual page-break management.
- Automatic propagation of source changes to all CV versions without user review.
- Mandatory cloud service, account, or hosted database.

## Important assumptions

- The candidate profile is stored separately from reusable CV content and is used by the header Styling.
- Email is expected to be required in the candidate profile, while location and phone are optional.
- Work Experience and Education have type-specific validation rules, but the exact required-field matrix is not yet finalized.
- The application will use a GUI editor capable of handling paragraphs, bullets, bold, italic, and hyperlinks.
- YAML is primarily edited by the application, but users are intentionally allowed to inspect, back up, synchronize, and potentially modify it externally.
- The source-change workflow will require source revision numbers, last-modified timestamps, content hashes, and references from CV definitions to source IDs and the recorded source metadata.
- A local override is version-specific and is represented separately from the reusable source entry.

## Open questions to resolve before implementation

1. Which operating systems must Version 1 officially support?
2. Which Python version should Version 1 support?
3. Which fields are mandatory for each content type, especially Work Experience and Education?
4. Is email mandatory in the candidate profile, or merely recommended?
5. Should social links wrap onto additional header lines when they do not fit, or should the app warn the user?
6. What starter Styling files and predefined CV types should ship with Version 1?
7. Which predefined font, size, spacing, color, and layout choices belong in the design system?
8. How should source-change review present differences to the user?
9. Should a later version provide a central view of all CV versions with stale source references?
10. What exact date semantics should CV versions store: creation date, last-modified date, or both?
11. What should the PDF file naming convention be?
12. How should the application behave if a synchronized workspace contains a missing, duplicated, or partially written file?
13. Which accessibility requirements apply to the GUI and generated PDF?

## Architectural implications without prescribing an implementation

- The data model must distinguish generic content meaning from Styling presentation and CV Composer selection.
- Stable IDs are needed for candidates, canonical entries, variants, sections, blocks, CV versions, Styling files, CV Composers, and local overrides.
- Source variants need parent-child relationships to their canonical entries.
- CV definitions need references to source IDs and the source revisions used, plus local overrides for edited CV-specific content.
- Saved CV behavior must distinguish live source references, CV-local overrides, and rendered PDF artifacts.
- CV Composers need references to Source content and Styling files; Styling defines the presentation rules.
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
6. Create a CV Composer using a selected Styling file and blank or predefined section configuration.
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
