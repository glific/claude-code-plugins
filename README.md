# Glific Claude Plugins

This repository contains Claude plugins for Glific workflows.

## Available plugin

- `iteration-planning`: Issue creation and issue update workflows for `glific/glific`.

## Included skills

- `create-issue`: Convert rough issue notes into a structured GitHub issue draft and create the issue after confirmation.
- `update-issue`: Read an existing GitHub issue, rewrite it into a structured format, and update it after confirmation.

## Install

Install this plugin through Claude Code marketplace commands (GitHub repo source):

1. Add this repository as a marketplace:

   ```bash
   /plugin marketplace add <your-org-or-user>/claude-code-plugins
   ```

2. Open the plugin manager and install `iteration-planning-plugins` from the **Discover** tab:

   ```bash
   /plugin
   ```

   Or install directly by name:

   ```bash
   /plugin install iteration-planning-plugins@glific-claude-plugins
   ```

3. Reload plugins in the current session:

   ```bash
   /reload-plugins
   ```

## Upgrade marketplace and plugin

When plugin metadata changes, refresh marketplace data and reinstall/update the plugin:

1. Refresh marketplace metadata:

   ```bash
   /plugin marketplace update glific-claude-plugins
   ```

2. Reinstall or upgrade the plugin:

   ```bash
   /plugin install iteration-planning-plugins@glific-claude-plugins
   ```

3. Reload plugins in the current session:

   ```bash
   /reload-plugins
   ```

## Add local marketplace for testing

Use a local checkout as a marketplace source while validating plugin changes:

1. Add the local marketplace path:

   ```bash
   /plugin marketplace add /absolute/path/to/claude-code-plugins
   ```

2. Install the plugin from that local marketplace entry:

   ```bash
   /plugin install iteration-planning-plugins@claude-code-plugins
   ```

3. Reload plugins after each local change:

   ```bash
   /reload-plugins
   ```

## Use

Run a skill from the installed plugin:

```bash
/iteration-planning-plugins:create-issue
```

or

```bash
/iteration-planning-plugins:update-issue
```

## Notes

- These skills use GitHub CLI (`gh`) to create or update issues, so make sure `gh` is installed and authenticated (`gh auth login`).
- Ensure you also run `gh auth refresh -s read:project` to grant access to project management commands.
- The skills are designed for the `glific/glific` repository by default.