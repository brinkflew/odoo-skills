---
name: odoo-frontend-standards
description: >-
  Covers OWL and the Odoo web client: @odoo/owl components, services, templates, asset bundles
  and web.assets_* entrypoints, RPC from JS, i18n, performance under static/.
  Use when editing static/**/*.js or **/static/**/*.xml, __manifest__.py assets, or customizing the web UI.
  For Python ORM, security, controllers (.py), and server-side XML rules, use odoo-backend-standards.
globs:
  - "**/static/**/*.js"
  - "**/static/**/*.ts"
  - "**/static/**/*.xml"
alwaysApply: false
---

# Odoo OWL / Web Client Coding Standards

These standards apply to the Odoo **frontend**: OWL components, web client services, and assets under `static/`. They align with Odoo’s web framework patterns and OWL best practices.

Use together with **`odoo-backend-standards`** for Python, ORM, security, and server-side XML data rules.

## Module Layout & Assets

- **Layout:** Keep sources under `static/src/` (e.g. `js/`, `xml/` for OWL templates, optional `scss/`). Avoid dumping raw files into `static/` without a clear bundle entry.
- **Manifest:** Register bundles in `__manifest__.py` via the `assets` key (current Odoo versions). Only include what the feature needs; prefer extension bundles (`web.assets_backend`, etc.) over duplicating entire core bundles.
- **Lazy vs eager:** Put rarely used UI behind lazy-loaded assets when your Odoo version supports it; keep critical path bundles lean.
- **Imports:** Follow project ESLint/prettier rules. Prefer `@web/...` public imports from the web client over deep paths into `web/static/src` unless maintaining core itself.

## OWL Components

- **Base class:** Subclass `Component` from `@odoo/owl` (or the version-correct OWL entry point for your Odoo branch).
- **Template:** Use `static template = xml` with imported XML, or the pattern your codebase uses consistently—keep template and class in sync when renaming props or slots.
- **Props:** Define `static props` (and optional `defaultProps`) so templates fail fast and callers know the contract.
- **Lifecycle:** Use `onWillStart` for async initialization tied to mount; use `onWillUnmount` / `onWillDestroy` to remove listeners, timers, and subscriptions—avoid leaks when components mount/unmount often (e.g. dialogs, list cells).
- **State:** Prefer reactive local state (`useState`) and explicit updates; avoid mutating nested objects in ways OWL cannot observe unless using patterns meant for that.

```js
// ❌ BAD: listener never removed
setup() {
  window.addEventListener("resize", this.onResize);
}

// ✅ GOOD: pair add/remove in lifecycle
setup() {
  this.onResize = this.onResize.bind(this);
  onMounted(() => window.addEventListener("resize", this.onResize));
  onWillUnmount(() => window.removeEventListener("resize", this.onResize));
}
```

## Odoo Web Integration (Services & Env)

- **Services:** Use `useService("orm")`, `useService("notification")`, `useService("action")`, `useService("dialog")`, etc., instead of globals or raw `fetch` to Odoo JSON routes unless there is no service equivalent.
- **Environment:** Rely on `this.env` / component env for services and config; do not invent parallel singletons for the same concerns.
- **Registries:** When extending registries (fields, views, systray, actions), follow existing categories and naming; register once at module load, not per component instance.

## Templates (OWL XML)

- **Declarative UI:** Prefer `t-if`, `t-foreach`, `t-component`, and props over imperative DOM manipulation.
- **Lists:** Always use a stable `t-key` in `t-foreach` when the list can reorder or filter.
- **Slots:** Use named slots and props for composition instead of reaching into child DOM from parents.

```xml
<!-- ❌ BAD: no key on dynamic list -->
<li t-foreach="items" t-as="item"><t t-esc="item.name"/></li>

<!-- ✅ GOOD: stable key -->
<li t-foreach="items" t-as="item" t-key="item.id"><t t-esc="item.name"/></li>
```

## RPC & Data Loading

- **`orm` service:** Prefer `read`, `searchRead`, `webSearchRead`, `call`, etc., through the ORM service rather than hand-built RPC unless implementing a dedicated endpoint.
- **Batching:** Avoid N+1 patterns from the UI (e.g. looping records and awaiting `read` per id). Batch IDs, use `read` once, or use `search_read` with the fields you need.
- **Domains & fields:** Never trust client-built domains or field lists for security—treat them as UX hints only; enforce access and validation on the server ([`odoo-backend-standards`](../backend-standards/SKILL.md)).
- **JSON2 API:** Prefer using the JSON2 API over the legacy RPC API in Odoo version 19+.

## Internationalization

- **User-visible strings:** Wrap literals with `_t` / translation helpers from the web stack (`@web/core/l10n/translation` or project-standard import). Use static string literals so gettext-style extraction works.
- **Interpolation:** Mark the **format string** as translatable, then format (e.g. `sprintf` from `@web/core/utils/strings`) so extraction sees a static literal—avoid template literals around translated text.

```js
import { sprintf } from "@web/core/utils/strings";
import { _t } from "@web/core/l10n/translation";

// ❌ BAD: template literal is not a static extractable string
const msg = _t(`Hello ${name}`);

// ✅ GOOD: literal inside _t, then format
const msg = sprintf(_t("Hello %s"), name);
```

## Security Mindset

- **Client is untrusted:** Anything the browser sends can be forged. Hide fields with `groups`/ACLs on the server; use record rules; validate in models/wizards—never rely on OWL alone to “hide” dangerous actions.
- **XSS:** Prefer OWL text escaping (`t-esc`); use `t-out` / raw HTML only with sanitized server-side content.

## Performance & UX

- **Rendering:** Avoid huge `t-foreach` without paging/virtualization; defer heavy work off the critical path.
- **Handlers:** Debounce search-as-you-type and resize handlers when they trigger RPC or expensive work.
- **Subscriptions:** Unsubscribe from buses, listeners, and reactive bridges when components destroy.

## Testing

- **Tours:** Use Odoo **web tours** for integration flows where appropriate; keep steps resilient to translation (use hooks/data attributes over brittle label text when possible).
- **Unit tests:** Follow project conventions (e.g. QUnit for legacy web tests); keep tests deterministic—mock services/env when testing components in isolation if your setup supports it.

## Version-Specific Notes

- OWL and `@web` import paths evolve between Odoo versions. Prefer patterns and APIs from **your shipped branch**; when copying snippets from docs or older modules, verify modules and service names against that branch.
- Asset bundle keys and manifest schema (`assets` vs older `web.assets_*` only) depend on version—match your deployment.
