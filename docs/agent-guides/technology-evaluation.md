# Technology Evaluation Guide

Use this guide before selecting or changing a major third-party dependency, especially GUI, rich-text editor, YAML, or PDF/rendering technology.

## Evaluation process
1. Read the product requirements relevant to the dependency.
2. Identify must-have capabilities and constraints.
3. Compare a small number of plausible options.
4. Check current maintenance, licensing, Python compatibility, cross-platform support, packaging, accessibility, and testing support.
5. Identify the highest-risk unknowns.
6. Build a small proof of concept for uncertain critical capabilities.
7. Summarize evidence, trade-offs, recommendation, and remaining risks before adoption.

## CV Builder-specific criteria
- Cross-platform desktop support.
- Rich-text editing for paragraphs, bullets, bold, italic, and hyperlinks.
- Consistent preview and PDF export.
- Control over pagination and keeping blocks intact.
- User-accessible YAML persistence and robust validation.
- Compatibility with the supported Python version and `uv` workflow.
- Reasonable packaging and maintenance burden.

Do not add dependencies for speculative future requirements. Do not claim a package supports a critical requirement without evidence or a working proof of concept.
