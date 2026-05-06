---
name: odoo-backend-standards
description: >-
  Specifies Odoo backend standards: Python ORM and models.Model, controllers,
  security (ir.model.access, record rules), domains/search, i18n on server and XML data,
  performance, Odoo tests, __manifest__.py and packaging.
  Use when editing .py addons, XML data/rules, manifests, server logic, or tests.
  For OWL, static/**/*.js/xml, asset bundles (@web), use odoo-frontend-standards.
globs: "**/*.py"
alwaysApply: false
---

# Odoo Python Coding Standards

These standards adapt our core Python coding guidelines specifically for Odoo development, aligning with both modern Python practices and the official Odoo Coding Guidelines.

## Imports

- **Order:** Standard Library → Odoo (`odoo`, `odoo.exceptions`, `odoo.http`, `odoo.models`, `odoo.fields`, `odoo.api`) → Third Party → Local/Relative.
- **Top-level by default:** Always use top-level imports. **Local imports** are permitted ONLY when strictly required to break a circular dependency or to avoid crashing a module load if a specific external dependency is missing.
- **Ruff/Isort:** Maintain `I` rules for `isort` where applicable, ensuring Odoo imports are treated as their own distinct category or primary third-party block. Do NOT fight the linter.

## Typing & Docstrings

- **Type Hints:** While Odoo relies heavily on dynamic recordsets (`self`), type hint external utility functions, controller responses, and domain helpers. For Odoo models, type hints are less idiomatic but acceptable for simple business logic methods.
- **Recordsets:** When type-hinting Odoo recordsets in custom Python helpers, you can use strings or generic placeholders if using Odoo 15+.
- **Docstrings:**
  - **Required:** Every model, controller, and major business method must have a docstring.
  - **Compute/Onchange:** Briefly explain what triggers it and what it calculates.

## Odoo ORM & Coding Style

- **Recordsets (`self`):** Understand that `self` is always a collection (recordset). Always account for multiple records. Use `for record in self:` explicitly unless using recordset operations like `mapped()` or `filtered()`.
- **Avoid Loops within Loops:** Use Odoo's built-in recordset methods. Rely on `filtered`, `mapped`, and `sorted` as they are optimized for readability.
- **F-Strings:** Use f-strings for all string formatting except translatable strings. For translatable strings (e.g., `UserError(_('...'))`), strictly use `%(...)s` formatting because `_()` needs to match static string definitions for extraction.
- **Truthiness:** Be explicit with recordset checks. `if recordset:` correctly evaluates to `True` if the recordset has elements. Do not use `if len(recordset) > 0:`.

## Data Validation

- **API Constrains:** Use `@api.constrains('field_name')` for complex cross-field validation. Always `raise odoo.exceptions.ValidationError`.
- **Database Constraints:** Prefer `Model.constraint` (or `_sql_constraints` in version 19-) over `@api.constrains` for simple uniqueness or check constraints, as they are significantly faster and enforced at the database level.
- **`@api.onchange`:** Use for UI responsiveness only (suggest values, warnings). Never rely on onchange alone for persisted invariants; enforce the same rules in computes, `@api.constrains`, `write`, or explicit business methods so API and imports behave correctly.

## API Decorators & CRUD Overrides

- **`create`:** Prefer `@api.model_create_multi` with a batch-oriented implementation when overriding `create`; avoid accidental per-record loops that defeat ORM batching.
- **`@api.model`:** Use when the method must not depend on the current recordset’s records (e.g. class-level helpers, `create` flows that start from an empty recordset).
- **`super()`:** Always chain `super()` correctly; capture return values (`res = super().write(...)`) and return them when the API contract requires it. Document behavior changes in overrides that alter access or side effects.

## Multi-Company

- **Fields:** Use `company_id` / `company_ids` consistently with Odoo patterns for the model’s scope.
- **Cross-company fields:** Set `check_company=True` on relational fields where cross-company links must be invalid; rely on framework checks instead of ad hoc validation when possible.
- **Context:** Prefer `with_company(company)` (and `with_user` when appropriate) to fix multi-company context instead of reaching for `sudo()`.

## Odoo Web Controllers

- **Controllers:** Keep HTTP controllers (`odoo.http.Controller`) thin. Controllers should handle request validation, routing, and access control, passing the heavy lifting to `request.env['your.model']`.
- **JSON/XML Routing:** Use `@http.route(..., type='json')` or `type='http'` appropriately. For JSON-RPC, return dictionaries; for HTTP, return rendered QWeb views or standard Werkzeug responses.
- **CSRF & Auth:** Pay strict attention to `auth='user'`, `auth='public'`, and `csrf=True/False`. Never disable CSRF on a POST route unless dealing with external webhooks that have alternate HMAC validation.
- **Untrusted input:** Never trust client-supplied domains, field lists, or model names without validation; enforce allowed models/fields and call `check_access_rights` / `check_access_rule` on the server.

## Declarative Model Fields

- **Labels and Definitions:** Avoid using `string=` if the field name is self-explicit as it is transformed by the ORM. Include `help=` for non-obvious fields to aid end-users.
- **Relational Fields:** Define `Many2one`, `One2many`, and `Many2many` fields carefully. For `One2many`, always define the inverse `Many2one` field on the target model.
- **Default Values:** Use `default=...` with care. For complex defaults, pass a method reference (e.g., `default=lambda self: self.env.user.company_id.id`) rather than executing logic in the field definition.

## Computed Fields

- **Search Methods:** Prefer implementing `search=...` over setting `store=True` on compute fields if the only reason is for searching, filtering or grouping.
- **Stored Computed Fields:** If setting `store=True`, you must have an accurate `@api.depends` decorator or the field will not recalculate when underlying data changes. If storing the field for search, filter or group purposes consider adding an index to this field.

## Model Class Body Order

Follow the Odoo official standards for ordering attributes and methods inside an Odoo models.Model:

1. **Private Attributes** — _name, _description, _inherit, _order.
2. **Selection/Default Methods** — Methods returning selection lists or complex defaults.
3. **Standard Fields** — fields.Char, fields.Many2one, etc.
4. **Compute, Inverse, and Search Methods** — Methods bound to compute fields.
5. **Constrains & Onchange** — `@api.constrains` and `@api.onchange` methods.
6. **CRUD Method Overrides** — Overrides of `create`, `read`, `write`, `unlink`, `copy`.
7. **Action Methods** — Methods called directly from UI buttons (returning action dicts).
8. **Business Logic Methods** — Any other custom business methods.

## Database & Migrations

- **Schema Generation:** Odoo automatically manages the DB schema via field definitions. Never modify the database schema directly using SQL.
- **Migrations:** When fundamentally changing a data structure (e.g., changing a field type or moving data), write standard Odoo migration scripts in the `migrations/` directory of the module (`pre-`, `post-` and `end-` scripts).
- **Environment SQL:** Use `self.env.cr.execute()` ONLY when the ORM is too slow for a massive bulk update or complex aggregation.

## Performance & Optimization

- **N+1 Prevention:** This is the most critical Odoo optimization. Never execute queries inside a for loop. Pre-fetch data or use `read_group` / `search_read`.
- **Search vs. Browse:** Use `search()` when you only need IDs. Use `browse()` when you have IDs and need recordsets. Use `search_read()` when you need dictionary data for an API or QWeb loop without instantiating full ORM records.
- **Context:** Be mindful of `self.with_context()`. Only pass context variables that are explicitly needed (e.g., `prefetch_fields=False` for massive cron jobs).

## Internationalization

- **`_()` and extraction:** Wrap only **static literal** strings in `_()` so gettext can extract them. Use named placeholders (`%`, `%(name)s`) for dynamic fragments; never build the translatable string with f-strings.
- **Model copy:** Translate user-visible model metadata appropriately (`_description`, field `help` / `string` where exposed in the UI) and keep message modules loaded for translators.

## Errors, Logging & Debugging

- **Odoo Exceptions:** Use `odoo.exceptions.UserError`, `ValidationError`, and `AccessError` for expected UI-facing errors. Never return global 500s for business logic.
- **Logging:** Configure `_logger = logging.getLogger(__name__)`. Use `_logger.info()`, `_logger.warning()`, and `_logger.error()`. Do not use print statements.
- **Translations:** Wrap all user-facing exception messages in `_()`, e.g., `raise UserError(_("The record cannot be processed."))`.

## Security

- **ACLs (`ir.model.access`):** Declare explicit access lines for every model that needs CRUD; follow least privilege (minimal groups / perms). Weak or missing ACLs are a common production issue.
- **Record rules (`ir.rule`):** Use record rules where row-level isolation is required; avoid extremely heavy domains on hot models without indexes or caching considerations.
- **Business methods:** Validate arguments and enforce access inside model methods callable from RPC (`check_access_rights`, `ensure_one()`, etc.), not only in views.
- **Sudo:** Avoid `self.sudo()` unless strictly necessary. It bypasses all record rules and access rights. Always document why `sudo()` is used with a comment.
- **Environment:** Prefer `self.with_user(bot_user)` or `self.with_company(company)` over blind `sudo()`.
- **SQL Injection:** NEVER interpolate user input into `self.env.cr.execute()`. Always use parameterized bindings.

```py
# ❌ BAD
self.env.cr.execute(f"SELECT id FROM res_users WHERE login='{username}'")

# ✅ GOOD
self.env.cr.execute("SELECT id FROM res_users WHERE login=%s", (username,))
```

## Mail & Chatter

- **Inherits:** When using `mail.thread` or `mail.activity.mixin`, keep inherit order and overrides predictable; do not log secrets, tokens, or personal data in chatter.
- **`message_post`:** Override only when necessary; preserve standard keyword arguments and access checks expected by the mail stack.

## Crons & Long-Running Jobs

- **Idempotency:** Scheduled actions should tolerate retries; avoid assuming a single run per logical event.
- **Batching:** Process records in batches with limits; avoid unbounded `search()` + full ORM loops in crons.
- **Transactions:** Use explicit commits / savepoints only when you accept partial progress and understand failure modes; prefer ORM-safe patterns for maintainability.

## Document Numbers & Sequences

- **Prefer `ir.sequence`:** Use standard sequences (per company / journal as required) for document numbers; do not implement ad hoc `MAX(id) + 1` or similar patterns that break under concurrency.

## Module Packaging (`__manifest__.py`)

- **Dependencies:** List every module you import or inherit from in `depends` but rely on transitive dependencies remaining implicit.
- **Metadata:** Keep `version`, `license`, `summary`, and `author` accurate; order `data` files so security (ACLs, groups) loads before data that depends on those rights.
- **Version**: Keep version formatted as `<odoo-major>.<odoo-minor>.<module-major>.<module-minor>.<module-patch>`. Follow semver principles for module versioning.

## XML & Static Data (for Module Authors)

- **External IDs:** Use stable, namespaced IDs (`my_module.action_invoice_custom`); avoid renaming IDs across versions without migration. Avoid the `my_module.` part if adding new records in the current module as it is inferred by the ORM.
- **`noupdate`:** Mark reference data with `noupdate="1"` when upgrades must not overwrite customer or localized customizations unintentionally.

## Testing

- **Odoo Test Classes:** Use `odoo.tests.common.TransactionCase` for standard unit tests, `SavepointCase` when savepoint isolation fits the scenario, and `HttpCase` for controller/UI tests.
- **Tags:** Tag tests with `@tagged('post_install', '-at_install')` to control when they run in the CI/CD pipeline.
- **Data:** Rely on XML demo data or explicitly create minimal, self-contained test records in the `setUp` method. Do not rely on specific production data existing.
- **Access:** Add tests that assert record rules and ACL behavior for critical flows, not only happy-path ORM calls.

## Version-Specific Notes

- Prefer patterns from the Odoo version you ship (for example `@api.model_create_multi`, newer constraint APIs). When copying examples from docs or older modules, verify they match your target Odoo branch and deprecate obsolete decorators or wizard patterns proactively.
