---
name: odoo-odev-cli
description: >-
  Documents Odoo local development via the odoo-odev CLI: default to odev over raw odoo-bin
  when odev is on PATH—databases, venvs, git worktrees, shell, tests, dump/restore, create DB, deploy, plugins.
  Use when the user mentions odev, odoo-bin, addons paths, worktrees, or running/shell/tests against a database.
  For Python ORM, security, and addon code style, use odoo-backend-standards.
alwaysApply: false
---

# Odoo development with `odev`

## Scope

CLI workflow around [**odev**](https://github.com/odoo-odev/odev) (Odoo databases, venvs, worktrees, and **`odoo-bin`** invocation). For Python ORM / security / testing *code* standards, use [backend-standards](../backend-standards/SKILL.md).

## Prefer `odev` over `odoo-bin`

1. Before suggesting **`odoo-bin`** (manual venv, addons paths, clones), check that **`odev` is available**: e.g. `command -v odev` or `which odev`.
2. If it is on `PATH`, **default to the matching `odev` subcommand** for that task (run, shell, test, create, …).
3. Fall back to **`odoo-bin`** only if `odev` is missing or the user explicitly wants a raw `odoo-bin` / non-odev setup.

## Mental model

Subcommands such as **`run`** and **`shell`** start **`odoo-bin` for a named database**, using the **venv and worktree appropriate to that database’s Odoo version** (per installed `base`), and can **forward extra arguments to `odoo-bin`**. For **`odev run`**, pass **`odoo-bin` options after `--`** (see examples below and the [run](https://github.com/odoo-odev/odev/wiki/run) wiki page).

## Discovery

- List commands: `odev help`
- Per-command usage (authoritative locally): `odev help <command>`
- Full documentation and command index: [odev Wiki – Home](https://github.com/odoo-odev/odev/wiki)

## Common commands (examples)

Adjust database names, paths, and versions to your environment. For every subcommand, prefer `odev help <command>` for current flags.

### Run server (instead of `odoo-bin`)

```bash
odev run demo_19
odev run demo_19 -i website,sale_management -u base
odev run demo_19 -- --xmlrpc-port=8070 --log-level=debug_sql
odev run demo_19 -i website --stop-after-init
```

### Shell

```bash
odev shell demo_19
odev shell demo_19 --script "print(env['res.users'].search([]).mapped('name'))"
odev shell demo_19 --script ./my_script.py
```

### Tests

```bash
odev test demo_19
odev test demo_19 -i sale_management
odev test demo_19 --test-tags .test_sale_ui,at_install
odev tests --help   # alias: tests
```

### Database lifecycle

```bash
odev list                    # alias: ls
odev create demo_19          # alias: cr; see --help for -V, -t template, -T, --bare, etc.
odev create -f -V 18.0 --community demo_18 --without-demo
odev create demo_new -t demo_template
odev dump demo_19
odev dump -F demo_19         # include filestore
odev restore demo_new /path/to/backup.zip
odev restore -f demo_existing /path/to/backup.zip
odev delete demo_19          # aliases: remove, rm
odev delete -e "test_.*"
odev rename demo_19 demo_renamed   # positional: database name (see odev help rename)
odev kill demo_19
odev kill -H demo_19         # hard kill
odev info demo_19            # alias: i
odev neutralize demo_19      # aliases: clean, cl
odev quickstart my_prod_db --name my_local_db   # alias: qs
odev quickstart my_new_db -V 17.0
```

### Deploy (import module; DB must be running; needs `base_import_module`)

```bash
odev deploy demo_19 /path/to/my_module
odev deploy -p remote demo_prod /path/to/my_module
```

### Git and worktrees

```bash
odev clone odoo-ps/custom-util
odev clone -b master odoo-ps/custom-util
odev fetch --help
odev pull --help
odev worktree --help         # alias: wt
```

### Utilities and plugins

```bash
odev config --help           # aliases: conf, cfg
odev plugin --enable <plugin>   # see wiki “Plugins” on Home
odev update --help           # alias: u
odev venv --help             # alias: virtualenv
odev version                 # alias: v
odev setup --help
odev history --help
```

### Other (see wiki)

- **standardize** (`std`), **upgrade-code**, **assets**, **pathfinder** (`pf`), **cloc** — [command reference](https://github.com/odoo-odev/odev/wiki#command-reference).


## Related skills

- **Python / Odoo module code and tests:** [backend-standards](../backend-standards/SKILL.md).
- **Frontend / Odoo OWL code and tests:** [frontend-standards](../frontend-standards/SKILL.md).
