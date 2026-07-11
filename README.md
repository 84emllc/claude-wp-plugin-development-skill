# wp-plugin-development

[![CI](https://github.com/84emllc/claude-wp-plugin-development-skill/actions/workflows/ci.yml/badge.svg)](https://github.com/84emllc/claude-wp-plugin-development-skill/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)
[![Agent Skill](https://img.shields.io/badge/type-agent%20skill-8250df.svg)](https://agentskills.io/specification)

An agent skill for developing, reviewing, and refactoring WordPress plugins: architecture and hooks, activation/deactivation/uninstall lifecycle, the Settings API, data storage and cron, a security baseline (nonces, capabilities, sanitization/escaping, SQL safety), and a priority-ordered code-review checklist.

Built by [84EM](https://84em.com) for use with Claude Code and other agent harnesses that load [Agent Skills](https://agentskills.io/specification).

## What is in here

| Path | What it is | License |
|------|------------|---------|
| `SKILL.md` | The skill: architecture guidelines, the plugin lifecycle, Settings API usage, the security baseline, data/cron guidance, and the review-priority order | MIT |
| `references/structure.md` | Plugin file/namespace organization guidance | MIT |
| `references/lifecycle.md` | Activation/deactivation/uninstall hook guardrails | MIT |
| `references/settings-api.md` | Settings API registration and sanitization patterns | MIT |
| `references/security.md` | The security baseline in depth | MIT |
| `references/data-and-cron.md` | Data storage and cron-task guidance | MIT |
| `references/debugging.md` | Common failure modes and how to diagnose them | MIT |
| `scripts/detect_plugins.mjs` | Deterministic scan that detects plugin headers in a repo | MIT |

## Install

**Claude Code (personal skill):** clone and symlink into your skills directory.

```bash
git clone https://github.com/84emllc/claude-wp-plugin-development-skill.git
ln -sfn "$(pwd)/claude-wp-plugin-development-skill" ~/.claude/skills/wp-plugin-development
```

The skill is then discoverable by its `name` and `description` frontmatter. No build step is required.

## Usage

Ask the agent to build, extend, or review a WordPress plugin. It follows a fixed procedure: triage the repo, locate plugin entrypoints, then work through architecture, lifecycle hooks, settings, security, and data/cron guidance as relevant, pulling in the matching `references/*.md` file for depth. In review mode it works a fixed priority order: security first, then coding-standard compliance, performance, testability, error handling, and internationalization.

**Two references depend on skills not included in this repo.** `SKILL.md` calls out to a separate `wp-project-triage` skill's detection script for the initial repo-triage step, and defers test-writing guidance to a separate `wp-plugin-testing` skill. Neither is included here; either supply your own equivalents or treat those two references as pointers to write your own tooling. `references/structure.md` also documents an `EightyFourEM\` PSR-4 namespace convention specific to 84EM's own projects -- adapt the namespace to your own if you're not building for 84EM.

## Licensing

MIT. See [LICENSE](./LICENSE).
