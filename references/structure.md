# Plugin structure and loading

Use this file when introducing or refactoring a plugin architecture.

## Core concepts

- Main plugin file contains the plugin header and bootstraps the plugin.
- Prefer predictable init:
  - minimal boot file
  - a loader/class that registers hooks
  - admin-only code behind admin hooks

## Autoloading and class file naming (84EM convention)

- Use PSR-4 autoloading via Composer. Do not use the WordPress core
  `class-{classname}.php` file-naming convention.
- Name class files to match the class: `ClassName.php`, mapped to a namespace
  under the PSR-4 prefix in `composer.json`.
- Use the `EightyFourEM\` root namespace on 84EM projects, with sub-namespaces
  mirroring the directory structure.
- The Plugin Handbook examples use the `class-` prefix style; this convention
  overrides that. Apply PSR-4 even when upstream references show otherwise.

Upstream reference:

- https://developer.wordpress.org/plugins/plugin-basics/
