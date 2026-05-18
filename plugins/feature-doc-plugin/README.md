# feature-doc — Claude Code plugin

Turn a solution doc into a feature implementation document, then a ticket plan, then GitHub issues. All grounded in your actual codebase.

```
Solution doc  →  Implementation doc  →  Ticket plan  →  GitHub issues
   (yours)      (Claude drafts it)    (epic + sub-tickets)    (gh CLI)
```

## What it does

When triggered, the skill walks through seven phases:

1. **Detect** — figures out if you pointed at a solution doc, pasted a ticket, or just described the feature
2. **Gather** — asks only the critical clarifying questions
3. **Explore** — reads your codebase for similar patterns, affected files, and conventions
4. **Draft** — writes the implementation doc into `docs/specs/<feature-slug>.md`
5. **Refine** — iterates with you until the doc is ready
6. **Plan tickets** — proposes one epic + sub-tickets (3–7, each S or M sized) for your review
7. **Create on GitHub** — after your explicit confirmation, uses `gh` CLI to create the issues and links them as a task list on the epic

You can stop at any phase — type "save the plan, I'll create them myself" after Phase 6 and you get a markdown file instead of GitHub issues.

## Install

### Option A — local install for testing

From the directory containing this plugin folder:

```
/plugin install ./feature-doc-plugin
```

### Option B — share with the team via a marketplace

Push this plugin folder to a git repo, then teammates run:

```
/plugin marketplace add your-org/your-plugins-repo
/plugin install feature-doc
```

## Prerequisites

- Claude Code
- `gh` CLI installed and authenticated (`gh auth status` should be green) — only needed for Phase 7
- Git remote pointing at the target GitHub repo

## Use

**Full pipeline from a solution doc:**

```
/feature-doc:create docs/solutions/threaded-comments.md
```

**Full pipeline starting from scratch:**

```
/feature-doc:create
```

…then paste a ticket, describe the feature, or attach a doc. The skill takes it from there.

**Jump straight to tickets** (if you already have an impl doc from a previous session):

```
/feature-doc:tickets docs/specs/threaded-comments.md
```

You can also invoke implicitly — say *"plan the implementation for adding threaded comments"* and the skill should trigger on the description.

## Structure

```
feature-doc-plugin/
├── .claude-plugin/
│   └── plugin.json
├── commands/
│   ├── feature-doc.md           # /feature-doc:create
│   └── tickets.md               # /feature-doc:tickets
└── skills/
    └── feature-doc/
        ├── SKILL.md             # the seven-phase workflow
        ├── templates/
        │   └── feature-doc-template.md   # the impl doc template
        └── references/
            ├── ticket-plan.md   # chunking rules + epic/sub-ticket templates
            └── github.md        # gh CLI sequence
```

## Customize

Most teams will want to tweak these:

- **`skills/feature-doc/templates/feature-doc-template.md`** — sections of the impl doc. Drop / add as your team needs. The skill reads whatever is there.
- **`skills/feature-doc/references/ticket-plan.md`** — chunking rules and ticket templates. Adjust if your team prefers vertical slicing, different default labels, or a different acceptance criteria style.
- **`skills/feature-doc/references/github.md`** — gh CLI commands. Edit if you want different defaults (e.g., auto-assign, default milestone, sub-issues instead of task lists).
- **`skills/feature-doc/SKILL.md`** description field — controls *when* the skill auto-triggers. Add team-specific phrases ("write an RFC", "scope this epic") if devs find it under-triggering.

After editing, run `/reload-plugins` in Claude Code.

## Safety

The skill **never** creates GitHub issues without your explicit "yes" after showing you the plan. If anything fails mid-creation, it stops and tells you exactly what got created so you can recover. No silent retries, no half-done plans.

If `gh` isn't installed, the skill falls back to writing ticket bodies as local markdown files you can copy-paste.

## Tips for the team

- The solution doc doesn't need to be polished — a few paragraphs of "here's how I think this should work" is enough. The skill fills in the gaps.
- Don't skip Phase 3 (codebase exploration) even on small features. It's what makes the doc useful instead of generic.
- The first draft of the impl doc is the start of the conversation, not the end. Refine it before generating tickets — fixing the doc is much cheaper than editing 5 tickets after creation.
- If the breakdown feels wrong in Phase 6, just say "merge 2 and 3" or "split the frontend ticket by route" — the skill will rework it.
