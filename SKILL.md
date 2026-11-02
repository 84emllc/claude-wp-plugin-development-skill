---
name: wp-plugin-development
description: "Use when developing, reviewing, or refactoring WordPress plugins: architecture and hooks, activation/deactivation/uninstall, admin UI and Settings API, data storage and custom tables, cron/tasks, REST/AJAX endpoints, security (nonces/capabilities/sanitization/escaping), performance, testability, i18n, release packaging, reviewing plugin code against WordPress coding standards, and the EightyFourEM\\ conventions used across 84EM projects."
compatibility: "Targets WordPress 7.0+ (PHP 7.4.0+). Filesystem-based agent with bash + node. Some workflows require WP-CLI."
---

# WP Plugin Development

## When to use

Use this skill for plugin work such as:

- creating or refactoring plugin structure (bootstrap, includes, namespaces/classes)
- adding hooks/actions/filters
- activation/deactivation/uninstall behavior and migrations
- adding settings pages / options / admin UI (Settings API)
- security fixes (nonces, capabilities, sanitization/escaping, SQL safety)
- packaging a release (build artifacts, readme, assets)

## Inputs required

- Repo root + target plugin(s) (path to plugin main file if known).
- Where this plugin runs: single site vs multisite; WP.com conventions if applicable.
- Target WordPress + PHP versions (affects available APIs and placeholder support in `$wpdb->prepare()`).

## Procedure

### 0) Triage and locate plugin entrypoints

1. Run triage:
   - `node ~/.claude/skills/wp-project-triage/scripts/detect_wp_project.mjs`
2. Detect plugin headers (deterministic scan):
   - `node ~/.claude/skills/wp-plugin-development/scripts/detect_plugins.mjs`

If this is a full site repo, pick the specific plugin under `wp-content/plugins/` or `mu-plugins/` before changing code.

### 1) Follow a predictable architecture

Guidelines:

- Match project conventions first: read CLAUDE.md / AGENTS.md and the existing plugin code, and adopt the established namespace, file organization, and patterns before applying any default. Project conventions win over generic advice when they conflict.
- Keep a single bootstrap (main plugin file with header).
- Avoid heavy side effects at file load time; load on hooks.
- Prefer a dedicated loader/class to register hooks.
- Keep admin-only code behind `is_admin()` (or admin hooks) to reduce frontend overhead.
- Use PSR-4 autoloading via Composer with the `EightyFourEM\` namespace; name class files `ClassName.php`, not the WordPress `class-{classname}.php` convention.

See:

- `references/structure.md`

### 2) Hooks and lifecycle (activation/deactivation/uninstall)

Activation hooks are fragile; follow guardrails:

- register activation/deactivation hooks at top-level, not inside other hooks
- flush rewrite rules only when needed and only after registering CPTs/rules
- uninstall should be explicit and safe (`uninstall.php` or `register_uninstall_hook`)

See:

- `references/lifecycle.md`

### 3) Settings and admin UI (Settings API)

Prefer Settings API for options:

- `register_setting()`, `add_settings_section()`, `add_settings_field()`
- sanitize via `sanitize_callback`

See:

- `references/settings-api.md`

### 4) Security baseline (always)

Before shipping:

- Validate/sanitize input early; escape output late.
- Use nonces to prevent CSRF *and* capability checks for authorization.
- Decide access only with `current_user_can()`. Never with the Referer, `wp_get_raw_referer()`, other headers, query arguments, `is_admin()` or `wp_doing_ajax()`; the client controls or can trigger all of them.
- Treat `wp_ajax_nopriv_*`, `?wc-ajax=` and REST routes as attacker-controlled; every REST route needs a capability-based `permission_callback`.
- Avoid directly trusting `$_POST` / `$_GET`; use `wp_unslash()` and specific keys.
- Use `$wpdb->prepare()` for SQL; avoid building SQL with string concatenation.

See:

- `references/security.md`

### 5) Data storage, cron, migrations (if needed)

- Prefer options for small config; custom tables only if necessary.
- For cron tasks, ensure idempotency and provide manual run paths (WP-CLI or admin).
- For schema changes, write upgrade routines and store schema version.

See:

- `references/data-and-cron.md`

### 6) Reviewing existing code

When the task is review rather than building, work in priority order:

1. Security vulnerabilities (highest priority)
2. WordPress coding-standard compliance
3. Performance implications (query count, caching, asset loading)
4. Testability
5. Error handling and user feedback
6. Internationalization coverage

Give specific, actionable feedback with code examples and explain why each issue matters. Flag vulnerabilities immediately and explain the exploit, not just the fix.

### 7) Tests

Test design and coverage are handled by the wp-plugin-testing skill: real WordPress
integration tests over mocks. Pair the two when adding a feature so it ships with
corresponding tests.

## Verification

- Plugin activates with no fatals/notices.
- Settings save and read correctly (capability + nonce enforced).
- Uninstall removes intended data (and nothing else).
- Run repo lint/tests (PHPUnit/PHPCS if present) and any JS build steps if the plugin ships assets.

## Failure modes / debugging

- Activation hook not firing:
  - hook registered incorrectly (not in main file scope), wrong main file path, or plugin is network-activated
- Settings not saving:
  - settings not registered, wrong option group, missing capability, nonce failure
- Security regressions:
  - nonce present but missing capability checks; or sanitized input not escaped on output

See:

- `references/debugging.md`

## Escalation

For canonical detail, consult the Plugin Handbook and security guidelines before inventing patterns.
