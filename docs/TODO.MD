# TODO

## 2026-10-21 — Documentation handover follow-up

- [ ] Review and approve example YAML schemas for Master source, Styling, and CV definitions.
- [ ] Decide the workspace directory/file layout and the recovery strategy for saves affecting multiple YAML files.
- [ ] Define domain fixtures and tests for stable IDs, tags, section/block selection, ordering, and replace-versus-new-entry behavior.
- [ ] Resolve the remaining source-deletion workflow, including how a user confirms replacement alternatives.
- [ ] Prototype rich-text editing and shared HTML/PDF rendering before selecting GUI and rendering dependencies.
- [ ] Decide PDF/HTML naming and output-location conventions.
- [ ] Define behavior for a block taller than an entire page and accessibility expectations.

## 2026-10-20

- [x] ~~Ambiguity 1 resolved: source changes, revisions, hashes, and historical CV content behavior documented.~~ Superseded by the later decision not to use saved source revisions or CV snapshots.
- [x] ~~Ambiguity 2 resolved: save actions, local overrides in the CV Composer, and source-change warnings documented.~~ Superseded by the later decision to replace source content or create a new Master source entry; persisted local overrides are not used.
- [x] ~~Naming clarified: Styling, CV Composer, and Source.~~ Superseded by the clarified terms Master source, Styling, and CV definition.
- [x] Documentation handover completed: requirements, project brief, architecture proposal, agent instructions, and relevant guides aligned.
- [x] `AGENTS.md` terminology reviewed and updated.
- [x] Documentation hand-review completed for obsolete separate override-file implications.
- [ ] ~~Review next: Ambiguity 3 — template/content and rendering stability.~~ Replaced by the rendering proof-of-concept and open decisions listed above.
- [ ] ~~Then resolve Ambiguity 4 — recovery after partial multi-file writes.~~ Retained as an open implementation decision in the new review list above.
