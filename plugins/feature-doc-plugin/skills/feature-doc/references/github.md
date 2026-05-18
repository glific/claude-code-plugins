# GitHub CLI reference

Read this when creating issues on GitHub (Phase 7 of the workflow).

## Prerequisites

Run these checks first. If any fail, stop and tell the user how to fix:

```bash
# Is gh installed?
which gh || echo "MISSING — install: https://cli.github.com/"

# Authenticated?
gh auth status

# What repo are we targeting?
git remote get-url origin
```

If `gh auth status` shows a different account than the user expects, they can switch with `gh auth switch`.

## Detect the target repo

```bash
# Owner/repo as gh expects it
gh repo view --json nameWithOwner -q .nameWithOwner
```

Show this to the user and confirm before creating anything. If they're in a fork and want to file against upstream, ask which.

## Check available labels

```bash
gh label list --limit 100
```

If a label you want to apply doesn't exist, **ask the user** before creating it. Don't auto-create labels — different repos have intentional label hygiene.

## Create the epic

Write the epic body to a temp file first; passing multi-line bodies via `--body` on the command line is fragile.

```bash
# 1. Write the body
cat > /tmp/epic-body.md <<'EOF'
[full epic body from the template — see ticket-plan.md]
EOF

# 2. Create the issue and capture the number
EPIC_NUM=$(gh issue create \
  --title "[Epic] Add threaded comments to posts" \
  --body-file /tmp/epic-body.md \
  --label "epic" \
  --label "feature" \
  | grep -oE '[0-9]+$')

echo "Epic: #$EPIC_NUM"
```

The `gh issue create` command prints a URL on success; the issue number is the trailing path segment.

## Create each sub-ticket

For each sub-ticket, in order:

```bash
cat > /tmp/sub-body-1.md <<'EOF'
[full sub-ticket body from the template — see ticket-plan.md]

Parent: #$EPIC_NUM
EOF

SUB1_NUM=$(gh issue create \
  --title "Add comments table, migration, and Drizzle model" \
  --body-file /tmp/sub-body-1.md \
  --label "feature" \
  --label "backend" \
  | grep -oE '[0-9]+$')

echo "Sub 1: #$SUB1_NUM"
```

Repeat for each sub-ticket, capturing each issue number.

## Link sub-tickets back into the epic

After all sub-tickets exist, update the epic body to include the task list with real issue numbers:

```bash
# Fetch current epic body
gh issue view $EPIC_NUM --json body -q .body > /tmp/epic-current.md

# Replace the placeholder sub-ticket list (or append if it's not yet there)
# Easiest: regenerate the epic body with the now-known sub-ticket numbers
cat > /tmp/epic-body-final.md <<EOF
[regenerated epic body with the sub-tickets section filled in]

## Sub-tickets

- [ ] #$SUB1_NUM — Add comments table, migration, and Drizzle model
- [ ] #$SUB2_NUM — Implement POST/GET/DELETE /api/posts/:id/comments endpoints
- [ ] #$SUB3_NUM — Build CommentThread and CommentForm components
- [ ] #$SUB4_NUM — Wire feature flag and rollout monitoring
EOF

gh issue edit $EPIC_NUM --body-file /tmp/epic-body-final.md
```

## Report back

After all issues are created, print a summary in chat:

```
Created 1 epic + 4 sub-tickets:

  Epic    #123 — https://github.com/org/repo/issues/123
  Sub 1   #124 — https://github.com/org/repo/issues/124
  Sub 2   #125 — https://github.com/org/repo/issues/125
  Sub 3   #126 — https://github.com/org/repo/issues/126
  Sub 4   #127 — https://github.com/org/repo/issues/127

Epic body updated with task list linking all sub-tickets.
```

## Failure handling

If `gh issue create` fails at any point, **stop**. Don't continue creating tickets. Report:

- Which tickets succeeded (with numbers and URLs)
- Which ticket failed (with the error)
- What's left to create

The user can decide whether to retry, edit manually, or abandon. Don't try to "recover" by guessing — half-created plans are worse than no plan.

Common failures:

- **Rate limit** — `gh` returns a 403 with `X-RateLimit-Remaining: 0`. Tell the user to wait (the reset time is in the response) or use a PAT with a higher limit.
- **Permission denied** — user lacks write access to the repo. Confirm the target repo is right.
- **Label doesn't exist** — `gh` errors with "label not found". Re-run without that label, or ask the user about creating it.

## Optional: assignee, milestone, project

If the user wants to set these, the flags are:

```bash
gh issue create \
  --assignee "@me" \
  --milestone "v1.2.0" \
  --project "Q2 Roadmap"
```

Don't apply these by default — ask first. Assignment in particular shouldn't be silent.

## Optional: GitHub sub-issues (newer feature)

If the user's repo has GitHub sub-issues enabled, they can be linked as proper parent-child rather than via task list. The `gh` CLI supports this via:

```bash
gh issue develop $SUB1_NUM --parent $EPIC_NUM
```

This is more robust than task lists (closing the parent closes children, progress tracking is built in) but requires the feature to be available on the target repo. Default to task lists for portability; offer sub-issues as an upgrade if the user mentions it.
