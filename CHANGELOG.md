# Changelog

All notable changes to this project are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- `references/security.md`: an "Authorization: capabilities only, never request data" section (no access decisions from the Referer, `wp_get_raw_referer()`, headers, query arguments, `is_admin()` or `wp_doing_ajax()`), plus "AJAX handlers" and "REST routes" sections.
- `references/security.md`: an output escaping contexts table, including `esc_textarea()` and `wp_json_encode()` with the script APIs, and guidance to run third-party HTML through `wp_kses_post()` before saving.
- `SKILL.md` security baseline: the authorization rule and the attacker-controlled entry points, matching `references/security.md`.

### Changed

- Compatibility target raised to WordPress 7.0+ and PHP 7.4.0+, matching the upstream WordPress agent skill.

### Fixed

- `scripts/detect_plugins.mjs` now reads plugin header fields written inside a docblock (` * Version: 1.0.0`). Output shape is unchanged.

## [0.1.0] - 2026-07-11

### Added

- Initial `wp-plugin-development` agent skill (`SKILL.md`), extracted from 84EM's private skill collection: architecture guidelines, plugin lifecycle guardrails, Settings API usage, the security baseline, data/cron guidance, and a priority-ordered code-review checklist.
- `references/structure.md`, `references/lifecycle.md`, `references/settings-api.md`, `references/security.md`, `references/data-and-cron.md`, `references/debugging.md`: topic-depth reference files for each stage of plugin work.
- `scripts/detect_plugins.mjs`: deterministic scan that detects plugin headers in a repository.

[Unreleased]: https://github.com/84emllc/claude-wp-plugin-development-skill/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/84emllc/claude-wp-plugin-development-skill/releases/tag/v0.1.0
