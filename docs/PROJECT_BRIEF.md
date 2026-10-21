# CV Builder — Project Brief

## Purpose

CV Builder is a local-first Python desktop application for maintaining reusable CV content and assembling tailored CVs for different roles, industries, and applications. Version 1 focuses on Industry CVs and is intended for a junior solo developer to understand and extend.

The application keeps reusable candidate content separate from document appearance and from the composition of any one CV. It produces consistent multi-page documents without requiring Word or another external editor.

## Main capabilities

- Maintain a candidate profile and reusable source content.
- Organize sections and content blocks with stable IDs and tags.
- Define section types with their own fields and layout requirements.
- Create CV definitions by selecting, arranging, and filtering source blocks.
- Edit source content in the application and choose whether to replace an existing entry or save a new reusable variant.
- Select independent document Styling.
- Preview and export CVs as HTML and PDF.
- Save and reopen CV definitions from user-accessible YAML files.
- Detect missing source references and offer explicit tag-based alternatives.

## Three-file architecture

Each candidate workflow uses three distinct YAML responsibilities:

- **Master source:** reusable candidate/profile data, section types, sections, blocks/variants, tags, and metadata.
- **Styling:** document-wide visual settings such as typography, colors, spacing, borders, and page layout.
- **CV definition:** one CV's metadata, selected Styling, ordered section instances, selected source IDs/tags, and composition settings.

The files are related but not interchangeable. A CV definition references reusable source content; it does not contain a saved snapshot or persisted local override of that content. The final rendered PDF remains a historical visual artifact until explicitly regenerated.

## Intended workflow

The user selects a candidate, creates or reuses a CV definition, adds section instances, chooses section types, filters and selects source blocks, fills empty blocks, orders sections and blocks, chooses Styling, previews, and exports. The library viewer provides a separate way to inspect and maintain reusable source content.

When an existing source block is edited, the user explicitly chooses to replace it under its existing ID or save the edit as a new master-source entry with a new ID. Source changes do not silently rewrite other CV definitions.

## Scope boundaries

Version 1 is local-first and YAML-based. It does not require a hosted backend, account system, database, collaboration, unrestricted layout authoring, or a full content-versioning/snapshot system. Exact schemas, file layout, technology choices, and some content validation rules remain open decisions documented in `docs/REQUIREMENTS_GATHERING.md`.
