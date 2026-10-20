# Agent Workflow, Plans, and Decisions

Use this guide for significant planning, implementation tasks, architecture decisions, or changes that span multiple components.

## Planning mode
- Inspect requirements and existing repository state first.
- State the problem, scope, assumptions, and unresolved questions.
- For architecture work, identify alternatives and trade-offs.
- Break work into small, verifiable steps.
- Do not modify files or write implementation code when the request is planning-only.

## Implementation mode
- Implement only the approved task.
- Make the smallest coherent change.
- Add/update tests and documentation where needed.
- If a material contradiction or design flaw appears, stop and report it rather than silently redesigning.
- Review the diff for unrelated changes.

## Review mode
- Findings first, ordered by severity.
- Use concrete file paths and symbols/line numbers where possible.
- Distinguish defects from risks and preferences.
- Do not fix findings unless asked.

## Plans and ADRs
Use a plan when a task has multiple dependent steps or significant uncertainty. Include scope, requirements, design, risks, steps, tests, and acceptance criteria.

For important accepted decisions, create a concise Architecture Decision Record (ADR) containing:
- Context.
- Decision.
- Alternatives considered.
- Consequences.
- Status: proposed, accepted, or superseded.

Do not record a proposal as accepted until the developer approves it. Keep documents useful and current; do not create them just to satisfy a template.
