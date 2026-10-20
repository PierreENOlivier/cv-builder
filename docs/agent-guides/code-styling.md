# Python Code Style Guidelines

## 1. Standards

Follow these Python Enhancement Proposals (PEPs):

- **PEP 8:** Python code formatting and naming conventions.
- **PEP 257:** Docstring conventions.
- **PEP 484:** Type hints.

PEPs define conventions and guidance; they are not all mandatory language rules. Follow the requirements in this document consistently unless the project explicitly establishes a different convention.

## 2. Formatting

- Use **4 spaces per indentation level**. Never use tabs for indentation.
- Keep lines at or below **79 characters** where practical, following PEP 8. Use reasonable judgment for long strings, URLs, and other unavoidable cases.
- Separate top-level functions and classes with two blank lines.
- Separate methods within a class with one blank line.
- Use blank lines to separate logical blocks of code.
- Use spaces around binary operators, after commas, and around assignment operators where appropriate.
- Do not use unnecessary whitespace inside parentheses, brackets, or braces.
- Use trailing commas in multiline collections and function arguments when they improve readability.
- Use UTF-8 source files.
- Prefer readable, conventional formatting over compressed or clever expressions.

## 3. Naming conventions

Use the following naming conventions consistently.

| Element | Convention | Example |
|---|---|---|
| Variables | `snake_case` | `candidate_name` |
| Functions | `snake_case` | `load_candidate()` |
| Methods | `snake_case` | `save_version()` |
| Modules | `snake_case` | `cv_repository.py` |
| Packages | Short, lowercase names | `persistence` |
| Classes | `PascalCase` | `CVVersion` |
| Exceptions | `PascalCase`, usually ending in `Error` | `CVValidationError` |
| Constants | `UPPER_SNAKE_CASE` | `DEFAULT_FONT_SIZE` |
| Internal implementation details | Leading underscore | `_validate_entry()` |
| Type variables | `PascalCase` | `EntryT` |

Additional rules:

- Use descriptive names that communicate meaning.
- Avoid vague names such as `data`, `thing`, `temp`, or `obj` when a more precise name is available.
- Use verbs for functions that perform actions, such as `load_cv()` or `validate_entry()`.
- Use nouns for classes and objects representing concepts, such as `Candidate`, `CVVersion`, and `Template`.
- Do not use abbreviations unless they are widely understood in the project context.
- Do not use names that shadow Python built-ins, such as `list`, `dict`, `str`, or `id`.

## 4. Imports

- Place imports at the beginning of the module, after the module docstring and any required future imports.
- Group imports in this order:
  1. Standard library.
  2. Third-party packages.
  3. Project-local modules.
- Separate the groups with blank lines.
- Prefer explicit imports over wildcard imports.
- Avoid circular imports. If two modules depend on each other, reconsider their responsibilities and dependency direction.
- Import modules or specific symbols deliberately; avoid importing more than is needed.
- Do not modify `sys.path` to work around an incorrect package structure.

Example:

```python
import json
from pathlib import Path

import yaml

from cv_builder.domain.models import Candidate
from cv_builder.persistence.repository import CVRepository
```

Adapt the project-local import paths to the actual package structure.

## 5. Functions and methods

- Give each function one clear, well-defined responsibility.
- Keep functions short enough to understand without excessive mental effort. There is no universal line limit; split a function when doing so improves clarity or separation of responsibilities.
- Prefer explicit parameters and return values over hidden side effects.
- Use early returns when they simplify conditional logic.
- Avoid deeply nested conditionals and loops when simpler structures are available.
- Avoid boolean parameters that make a function call difficult to understand when separate functions or a clearer interface would be better.
- Avoid mutable default arguments such as `items=[]` or `options={}`.
- Do not use exceptions for ordinary control flow when a clear conditional is more appropriate.
- Raise specific exceptions that communicate what went wrong.
- Do not catch exceptions unless the code can handle them, add useful context, or recover safely.
- Avoid overly generic functions that attempt to handle unrelated responsibilities.

## 6. Type hints

- Add type hints to public functions and methods.
- Add return type hints to functions, including `-> None` when a function intentionally returns no value.
- Use type hints for important internal functions when they improve clarity or catch likely errors.
- Use standard collection types and expressive types appropriate to the project's supported Python version.
- Use `None` explicitly when it is a meaningful possible value.
- Use type aliases for complex types when doing so improves readability.
- Avoid adding unnecessary complexity merely to satisfy a type checker.
- Do not use `Any` as an escape hatch when a more precise type is reasonably available.

Example:

```python
def find_candidate(
    candidates: list[Candidate],
    candidate_id: str,
) -> Candidate | None:
    """Find a candidate by its identifier."""
    for candidate in candidates:
        if candidate.id == candidate_id:
            return candidate

    return None
```

Ensure the syntax used is compatible with the Python version declared by the project.

## 7. Docstrings and comments

- Use docstrings for public modules, classes, and functions.
- Follow PEP 257 conventions.
- Use triple double quotes for docstrings.
- Begin a one-line docstring with a concise imperative description where appropriate.
- For longer docstrings, include a short summary followed by additional details, arguments, return values, or exceptions when useful.
- Explain *why* a non-obvious decision was made rather than restating what the code does.
- Remove outdated, redundant, or misleading comments.
- Do not add comments that merely translate obvious code into English.
- Do not leave temporary debugging comments or commented-out code in committed changes.

Example:

```python
def validate_candidate(candidate: Candidate) -> None:
    """Validate required candidate information.

    Raises:
        CVValidationError: If required information is missing.
    """
```

Document actual behavior. Do not claim that a function raises an exception unless it really does.

## 8. Classes and object-oriented design

- Give each class a clear responsibility.
- Keep domain concepts separate from GUI widgets, file operations, and PDF rendering.
- Avoid classes that merely wrap a single function without providing meaningful behavior or a useful abstraction.
- Avoid large classes that manage unrelated concerns.
- Prefer composition over inheritance unless the relationship genuinely represents an "is-a" relationship.
- Keep mutable state explicit and controlled.
- Use properties when they provide a meaningful interface, not merely to wrap every attribute.
- Use dataclasses when their generated behavior fits the model; do not use them automatically for every class.
- Avoid duplicating business rules across multiple classes.

## 9. Error handling and data integrity

- Use specific exception types for meaningful failure conditions.
- Preserve useful error context when handling exceptions.
- Do not silently ignore malformed data, failed writes, missing required values, or validation errors.
- Avoid broad `except Exception` handlers unless they are justified at a deliberate application boundary.
- Keep user-facing error messages understandable and actionable.
- Never expose internal tracebacks directly to ordinary users unless explicitly operating in a developer/debug mode.
- Do not report a save as successful until the write has completed successfully.
- For persistent CV data, prioritize correctness and recoverability over brevity or convenience.

## 10. Dependencies and separation of concerns

- Prefer the Python standard library when it meets the requirement adequately.
- Add a third-party dependency only when its benefits justify the maintenance, security, and packaging costs.
- Keep GUI, domain logic, persistence, and rendering responsibilities separated.
- Domain logic should not depend directly on GUI widgets or presentation-specific code.
- Avoid circular dependencies between application modules.
- Keep file-system access behind clear persistence interfaces where practical.
- Do not introduce abstractions solely for hypothetical future requirements.

## 11. Tests and maintainability

- Write tests for behavior, especially business rules and data integrity.
- Test boundary cases and failure paths, not only successful operations.
- Use descriptive test names that communicate the behavior being verified.
- Keep tests deterministic and independent where practical.
- Do not weaken or remove a test simply because an implementation fails it.
- Avoid duplicated logic between production code and tests when it could make both incorrect in the same way.
- Remove dead code and unnecessary complexity when making changes.
- Keep changes focused on the task at hand.

## 12. Tooling

Use automated tools to enforce mechanical conventions rather than relying on manual review alone.

Recommended tools:

- **Ruff:** Linting and formatting.
- **Pyright or mypy:** Static type checking, if adopted by the project.
- **pytest:** Automated testing.

Configure the selected tools in the project configuration and keep their settings consistent across development environments and continuous integration.

Do not introduce multiple tools that perform the same job without a clear reason. Agree on the project's Python version and tool configuration before relying on version-specific syntax or enforcement rules.

## 13. Instructions for coding agents

When writing or modifying code:

1. Read the applicable project requirements and agent guides.
2. Follow the existing project conventions unless they conflict with explicit requirements or these guidelines.
3. Prefer the smallest clear change that satisfies the task.
4. Do not reformat unrelated files or rewrite working code unnecessarily.
5. Run the relevant formatter, linter, type checker, and tests when configured and practical.
6. Report checks that passed, failed, or could not be run.
7. Do not claim that code was tested or validated unless the relevant check was actually performed.
8. Explain significant design decisions and any deviations from these conventions.
