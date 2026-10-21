# Domain Model Guide

Use this guide when changing domain concepts, Master source content, variants, CV definitions, or Styling.

Consult `REQUIREMENTS_GATHERING.md` for the authoritative product rules.

## Important concepts
- Candidate profile: identity and contact/header information for one candidate.
- Source: candidate-owned reusable sections, blocks, canonical entries, variants, tags, and source metadata.
- Canonical source entry: stable identity for one real-world item or reusable content concept.
- Source variant: a particular reusable wording/content version under a canonical entry; it has its own ID and tags.
- CV version: a saved composition of sections and selected blocks for a candidate.
- Styling: appearance configuration for fonts, colors, spacing, alignment, borders, and layout. Built-in Styling files are read-only; valid user-created copies may be reusable by multiple CV definitions.
- CV definition: the YAML file for one CV version, containing references to Styling and Master source content, section composition, ordering, and recovery tags. It does not contain a persisted content snapshot or local override.
- Rendered PDF: generated output associated with a CV version.

These are conceptual definitions, not a mandate to create one class per concept.

## Invariants
- Different real-world items use different canonical entries.
- Variants have unique IDs; tags belong to variants.
- A CV version selects at most one variant per canonical entry.
- A source variant and a CV version are not interchangeable concepts.
- Editing an existing source block either replaces it under the existing ID or creates a new master-source entry with a new ID.
- Source updates are not silently propagated across CVs.
- Styling defines layout and presentation; CV definition section labels can differ from generic content types.

## Source save actions
Keep these user choices semantically distinct:
1. Replace source: update the reusable source under its existing source ID.
2. Save new reusable variant: create a new master-source entry/variant with a new ID and inherited tags/metadata unless changed.
3. Save the CV definition: persist selection and arrangement only; it is not a content override action.

Before changing these flows, trace effects on the current CV, other CVs referencing the source, tag-based recovery, and future rendering. If expected behavior is not defined, describe the ambiguity and ask rather than silently choosing.

## Reproducibility
Do not assume that a source reference alone preserves the content of a previously saved CV. The CV definition records source identity and recovery tags, not a snapshot. Verify that missing references produce explicit tag-based alternatives and that rendered documents remain available as visual history.
