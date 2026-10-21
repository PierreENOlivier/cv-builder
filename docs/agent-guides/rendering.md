# Editing, Styling, Preview, and PDF Guide

Use this guide for rich-text editing, Styling design, CV definition resolution, preview, pagination, and PDF export.

## Requirements to preserve
- Version 1 prioritizes PDF export.
- Users edit content inside the application.
- Supported content formatting includes paragraphs, bullets, bold, italic, and hyperlinks.
- Styling controls fonts, sizes, spacing, alignment, colors, borders, and layout through predefined choices.
- Preview reflects the current approved editing state; unsaved drafts must be saved or cancelled before rendering.
- Preview and final PDF should share rendering behavior so they remain consistent.
- PDF content flows across pages.
- A section starting at the bottom of a page which cannot accomodate the next complete block jumps to the next page. The section and first block stay together.
- A block that cannot fit in remaining page space moves intact to the next page.
- Manual page-break management is out of scope.
- Warn and obtain confirmation before overwriting an existing rendered PDF.

## Design guardrails
- Keep Master source semantic content and CV definition selection distinct from Styling presentation/layout.
- Use a deliberate, validated rich-text representation; do not treat arbitrary GUI markup as trusted domain data.
- Preserve formatting and hyperlinks across save/load, preview, and PDF export.
- Do not let PDF layout code own domain rules.
- Resolve source content through the master source and CV definition; do not treat a CV definition as a content snapshot.
- Test page-boundary behavior and oversized blocks explicitly.
- Do not claim exact preview/PDF fidelity until demonstrated with the selected technology.

## Technology investigation
Before choosing a GUI editor or PDF library, validate:
- Cross-platform support and packaging.
- Rich-text editing capability.
- Preview/export consistency.
- Pagination and block-level layout control.
- Accessibility.
- Licensing and maintenance.
- Testability and integration complexity.

Prefer a small proof of concept for high-risk capabilities over selecting a dependency based only on popularity.
