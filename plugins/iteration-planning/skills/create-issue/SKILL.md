---
name: create-issue
description: Convert a user-provided issue blob into a structured issue with Description and Acceptance Criteria using Glific project context from CLAUDE.md files. Use when users share rough bug/enhancement/feature text and need a production-ready issue draft.
---

# Ad Hoc Issue Structuring

## When to use

Use this skill when the user shares an unstructured blob of text describing a problem, bug, enhancement, or feature request and wants it converted into a structured issue.

## Inputs

- A free-form text blob from the user that explains the issue.

## Required context gathering

Before producing the structured issue, fetch and use project context from:

- `https://raw.githubusercontent.com/glific/glific/refs/heads/master/CLAUDE.md`
- `https://raw.githubusercontent.com/glific/glific-frontend/refs/heads/master/CLAUDE.md`

If either file is unavailable, continue with available context and explicitly state assumptions in your internal reasoning.

## Workflow

1. Parse the blob and classify the issue as one (or more) of:
   - user-facing feature
   - bug fix
   - enhancement
2. Map the issue to exactly one issue category for GitHub title/labels:
   - `security`
   - `bug`
   - `ops`
   - `ci`
   - `testing`
   - `enhancement`
   - `docs`
   - `refactor`
   - `<epic-name>`
   If category is unclear from the blob/input, ask the user via `AskQuestion` before drafting.
3. If the issue looks like it is part of an epic, ask the user to provide the epic name and use that value as the category label.
4. Format the issue title as:
   - `([<category>]: <short issue title>)`
   - Example: `([bug]: Contact import fails for CSV with empty phone fields)`
5. Ensure the selected category is also added as a GitHub issue label.
6. Extract key facts (user-facing only):
   - current behavior
   - expected behavior
   - affected actors/users
   - user-visible constraints/dependencies
   - user-visible risks and unknowns
7. Use CLAUDE.md context from both repositories to:
   - align terminology with project conventions
   - propose reasonable suggestions for unanswered questions
8. Apply role-based quality checks below.
9. If critical details are missing, ask targeted questions using the `AskQuestion` tool before drafting.
10. Produce only the final structured issue output format.
11. Keep the issue at a high user-facing level; avoid implementation design details unless the user explicitly asks for them.
12. After presenting suggestions/improvements, ask for explicit final confirmation (yes/no) right before issue creation.
13. If the user provided image attachments with the instructions, include them in the final GitHub issue body under a short `## Attachments` subsection using markdown image links (or plain links when embedding is not possible).
14. Only after that confirmation, create the issue in `glific/glific` using `gh`:
   - `gh issue create --repo glific/glific --title "[<category>]: <title>" --body "<structured markdown content>" --label "<category>"`
15. Confirm back with created issue URL

## GitHub issue management rules (must enforce)

- Do not create a GitHub issue unless the user has explicitly confirmed issue creation in the current chat turn.
- Always create the issue in the `glific/glific` repository (not `glific/glific-frontend` unless the user explicitly overrides this).
- Always require exactly one category from: `security`, `bug`, `ops`, `ci`, `testing`, `enhancement`, `docs`, `refactor`, or `<epic-name>`.
- Always format the final title as `[<category>]: <short issue title>`.
- Always add the same category as a GitHub label when creating the issue.
- Preserve the final structured markdown sections in the issue body.
- If the user shared image attachments, include them in the issue body as attachment links/images.

## Role-based checks (must enforce)

### Product Manager

- If the issue is user-facing, include:
  - a feature flag requirement (name it if possible)
  - metrics to measure usage/adoption
- If the issue is a bug fix, include:
  - explicit conditions to reproduce/simulate the bug
  - expected behavior after the fix

### Tech Lead

- Keep requirements outcome-focused and user-facing.
- Do not include implementation split sections such as backend requirements, frontend requirements, API changes, DB changes, or architecture breakdowns.

### QA

- Ensure acceptance criteria covers:
  - happy path
  - edge cases
  - failure/validation scenarios where relevant

## Handling missing information

For any question not answered by the user blob or role requirements:

1. Use insights from the Glific `CLAUDE.md` files.
2. Ask the user for missing details using the `AskQuestion` tool (do not ask in plain text when `AskQuestion` is available).
3. Ask only high-signal questions that materially affect user impact, scope, or acceptance criteria.
4. Keep the questionnaire short:
   - prefer 1-5 questions
   - use clear, mutually exclusive options
   - allow multi-select only when truly required
5. If a required detail is still unavailable after asking:
   - make a conservative assumption aligned with project conventions
   - clearly encode that assumption in the issue description or acceptance criteria
6. Propose practical suggestions that fit project conventions.
7. Convert those suggestions into explicit acceptance criteria statements where appropriate.

## Question quality rubric (for `AskQuestion`)

- Prioritize questions that unblock:
  - user impact and target persona
  - expected behavior and success conditions
  - rollout constraints (feature flag, backwards compatibility, migration risk)
  - QA scope (edge/failure cases)
- Avoid asking for information already inferable from the blob or `CLAUDE.md`.
- Do not ask speculative or low-value preference questions.

## Output format (STRICTLY FOLLOW THIS FORMAT)

Return only:

```markdown
## Description
...

## User Flow
...

## Acceptance Criteria
...

## Edge cases
...
```

# Guidelines
- Do not add extra sections.
- Do not create sections like "Backend Requirements", "Frontend Requirements", "Technical Requirements", or any implementation-specific breakdown.
- Keep all content high-level and user-facing.
- Do not ask follow-up questions unless absolutely required to avoid unsafe or invalid assumptions.