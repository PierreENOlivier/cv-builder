# Architecture Guide

Use this guide for architecture proposals, component boundaries, dependency direction, or significant design changes.

## Expected process
1. Read the relevant product requirements and current code/documentation.
2. Identify architectural drivers: data integrity, source reuse and independence, editable rich text, predictable PDF rendering, local-first operation, and cross-platform support.
3. Propose a small number of cohesive components with explicit responsibilities.
4. Explain dependency direction and how core domain behavior can be tested without GUI or filesystem dependencies.
5. Compare at least one plausible alternative and explain costs, risks, and why the recommendation fits.
6. Identify decisions that require human approval.
7. Do not modify files during a planning-only task.

## Design principles
- Keep domain rules independent of GUI event handlers and serialization details.
- Keep YAML/file operations behind a clear persistence boundary.
- Keep rendering/layout concerns out of domain entities.
- Use explicit inputs and outputs; avoid hidden global state.
- Prefer the smallest design that meets current requirements.
- Do not invent extra layers merely to follow a pattern.
- Do not lock in a framework before validating rich-text, preview/PDF, pagination, accessibility, and packaging needs.

## Architecture proposal deliverable
Include:
- Key requirements driving the design.
- Conceptual components and their responsibilities.
- A dependency-direction explanation or diagram.
- Critical workflow traces.
- Alternatives and trade-offs.
- Risks and validation experiments.
- Open decisions, clearly separated from recommendations.

Do not define Python classes or detailed module trees until the conceptual boundaries have been reviewed.
