# AGENTS.md

Instructions for agents and human contributors working in this repository.

## What this repo is

A single agent skill (`SKILL.md`), six topic reference files under `references/`, and one detection script (`scripts/detect_plugins.mjs`) for WordPress plugin development.

## Hard rules

1. **`scripts/detect_plugins.mjs` output shape is a contract.** `SKILL.md` step 0 tells the agent to run this script and act on its output. If you change what the script detects or how it reports, update the corresponding guidance in `SKILL.md` step 0 in the same commit.

2. **Security guidance in `SKILL.md` and `references/security.md` must not contradict each other.** `SKILL.md`'s "Security baseline" section is the always-loaded summary; `references/security.md` is the detailed version. A change to one (e.g. a new required check) must be reflected in the other, even if only as a pointer.

3. **The review-priority order is fixed: security, coding standards, performance, testability, error handling, i18n.** This order is deliberate (security first, always). Don't reorder it without a documented reason in the commit message.

4. **This skill references two skills not included in this repo** (`wp-project-triage` for the initial triage step, `wp-plugin-testing` for test-writing guidance). Don't silently remove those references or pretend the functionality is self-contained -- if you fork or extend this skill to be fully standalone, say so explicitly in the README, don't just delete the pointers.

## Conventions

- Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`). Semantic Versioning. Keep a Changelog.
- GitHub Actions pinned to commit SHAs, not moving tags.
- Do not commit generated junk.
