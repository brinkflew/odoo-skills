# Odoo AI Skills

Agent skills for use with the open Agent Skills ecosystem ([agentskills.io](https://agentskills.io)). Skills live under `[skills/](skills/)` as `skills/<skill-name>/SKILL.md`.

## What are skills?

Each skill is a folder containing `**SKILL.md**` with YAML frontmatter. Required fields `**name**` and `**description**` tell the agent what the skill does and **when it should apply**; the Markdown body is the instructions the agent follows when the skill is active. Install and discovery are handled by the open `[skills` CLI](https://github.com/vercel-labs/skills); the format follows the [Agent Skills specification](https://agentskills.io).

## Skills in this repository


| Directory                                                                    | YAML `name` (`--skill` / `--list`) | Use when (summary)                                                    |
| ---------------------------------------------------------------------------- | ---------------------------------- | --------------------------------------------------------------------- |
| `[skills/odoo-backend-standards/](skills/odoo-backend-standards/SKILL.md)`   | `odoo-backend-standards`           | Python/XML Odoo addons, ORM, security, manifests, server logic, tests |
| `[skills/odoo-frontend-standards/](skills/odoo-frontend-standards/SKILL.md)` | `odoo-frontend-standards`          | OWL/static JS/XML, manifest assets, web client customization          |
| `[skills/odev-cli/](skills/odev-cli/SKILL.md)`                               | `odoo-odev-cli`                    | `odev`, databases, worktrees, shell/tests vs raw `odoo-bin`           |
| `[skills/git-workflow/](skills/git-workflow/SKILL.md)`                       | `git-workflow-ps-tech`             | PS-Tech branching, rebase-first flow, GitHub/`gh`, Odoo.sh            |
| `[skills/git-commit/](skills/git-commit/SKILL.md)`                           | `git-commit-message-format`        | Commit message format and wording                                     |


The `**name` in frontmatter** is what the CLI shows and what you pass to `**--skill`**; it can differ from the directory name (for example `git-commit` vs `git-commit-message-format`). Run `**--list**` (below) on this repo or your install source to see the canonical names.

## Using skills after installation

- **Where they live (Cursor):** project install goes to `**.agents/skills/`** by default; `**npx skills add ... -g**` installs to `**~/.cursor/skills/**` globally. See the [skills CLI agent table](https://github.com/vercel-labs/skills#supported-agents) and [Cursor Skills](https://cursor.com/docs/context/skills) for how your editor loads and matches skills.
- **How they apply:** agents typically use the skill’s `**description`** (and your task wording) to decide when to follow a skill. Some skills set optional frontmatter such as `**globs**` and `**alwaysApply**` (for example `[odoo-backend-standards](skills/odoo-backend-standards/SKILL.md)` and `[odoo-frontend-standards](skills/odoo-frontend-standards/SKILL.md)`); where the agent supports those fields, open files or policy can influence attachment. Otherwise, matching still relies mainly on `**description**`.
- **Verify:** `npx skills list` or `npx skills ls -a <agent>` to see what is installed for the agent.
- **Refresh / remove:** `npx skills update` and `npx skills remove` (see the [CLI README](https://github.com/vercel-labs/skills)). Symlink installs stay pointed at the source; copied installs may need an update when this repository changes.
- **Explicit invocation** (e.g. naming a skill or adding file context) is product-specific; use your agent’s docs rather than duplicating UI steps here.

## Install into a project

From another repository (after this one is pushed to GitHub; shorthand is `owner/repo`):

```bash
npx skills add brinkflew/odoo-skills
```

Install specific skills by `**name**` from `[--list](#list-skills-without-installing)`:

```bash
npx skills add brinkflew/odoo-skills --skill odoo-backend-standards --skill odoo-odev-cli
```

Skills are installed under `.agents/skills/` by default (see the `[skills` CLI](https://github.com/vercel-labs/skills) README for paths and flags).

## List skills without installing

```bash
npx skills add brinkflew/odoo-skills --list
```

While developing locally from a clone of this repo:

```bash
cd /path/to/odoo-skills
npm exec --yes --package=skills -- skills add . --list
```

## Maintaining installs

- **List installed:** `npx skills list` (optionally `-a <agent>`, `-g` for global scope).
- **Update:** `npx skills update` (see CLI for `-y`, `-g`, `-p`, and naming specific skills).
- **Remove:** `npx skills remove` (see CLI for `-a`, `-g`, `--all`).

## Add a new skill

From the repository root, generate a template and move it under `skills/`:

```bash
npm exec --yes --package=skills -- skills init my-new-skill
mv my-new-skill skills/my-new-skill
```

Then edit `skills/my-new-skill/SKILL.md` (update `name`, `description`, and body).