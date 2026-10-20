# Domain Model Guide

Use this guide when changing domain concepts, source content, variants, CV versions, templates, or overrides.

Consult `REQUIREMENTS_GATHERING.md` for the authoritative product rules.

## Important concepts
- Candidate profile: identity and contact/header information for one candidate.
- Canonical source entry: stable identity for one real-world item or reusable content concept.
- Source variant: a particular reusable wording/content version under a canonical entry; it has its own ID and tags.
- CV version: a saved composition of sections and selected blocks for a candidate.
- Template: presentation and section configuration, separate from source content.
- Local override: CV-specific content that does not mutate the reusable source.
- Rendered PDF: generated output associated with a CV version.

These are conceptual definitions, not a mandate to create one class per concept.

## Invariants
- Different real-world items use different canonical entries.
- Variants have unique IDs; tags belong to variants.
- A CV version selects at most one variant per canonical entry.
- A source variant and a CV version are not interchangeable concepts.
- A local override is specific to a CV version and must not mutate reusable source data.
- Source updates are not silently propagated across CVs.
- Template labels/layouts can differ from generic content types.

## Source save actions
Keep these user choices semantically distinct:
1. Replace source: update the reusable source under its existing source ID.
2. Save copy to source: create a new reusable variant with a new ID.
3. Create local override: save CV-specific content without changing reusable source content.

Before changing these flows, trace effects on the current CV, other CVs referencing the source, source revision history, and future rendering. If expected behavior is not defined, describe the ambiguity and ask rather than silently choosing.

## Reproducibility
Do not assume that a source reference alone preserves the content of a previously saved CV. Follow the approved policy for persisted CV content, source identity/revision metadata, and change review. Verify that reopening and rendering a saved CV behaves according to that policy.
