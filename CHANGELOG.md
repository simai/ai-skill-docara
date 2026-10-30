# Changelog

## 0.6.0 - 2026-10-30

- Open skill: Agent Skills format with `license` and `compatibility`, English
  throughout, Apache-2.0, works with or without SIMAI Federation; what only the
  Federation needs moved to `rules/federation.md`. Russian remains only in test
  requests under `quality/` and routing words, which check Russian requests.

## 0.5.0 - 2026-10-23

- Moved the skill from Docara 2 to Docara 3 (checked against `v3.8.0`):
  bundled SIMAI Framework release, Node.js 20+ build requirement, `init`
  templates, receipts in `build_<environment>.docara/`, discovery files,
  404 page, portal builds, site colour theme, hero effects, strict
  translation publishing and the extended front matter list.
- Added the manual 2.x -> 3.x migration route (Framework lock check, removed
  `sf-admin-menu`/`sf-table`, environment variables, output differences).
- Added freshness headers to every reference, a version re-check rule, the
  "Чем навык не занимается" block, and the Mirai 2 CLI names.
- Removed absolute local paths from README and AGENTS.

## Unreleased (before 0.5.0)

- Corrected the pre-manifest project transition: a missing `.docara/engine`
  now routes through reviewed `update --dry-run --adopt`, explicit apply and
  verification before the high-level upgrade is retried.

- Added the project-local capabilities handshake and compatibility range for
  serving multiple Docara 2.x projects with one canonical skill.
- Replaced package-selection assumptions with the verified high-level
  `upgrade` and offline rollback workflow while retaining low-level engine
  `update` guidance.
- Declared Federation as the physical installation owner; a project upgrade
  never writes the global Codex skill runtime.
- Aligned the operational references and capability graph with Docara 2.1
  reusable project examples and non-blocking translation tracking.
- Added the `translations status` and hash-bound `translations accept` agent
  workflow, including explicit AI and human review levels.
- Added layered Federation 3.0 recipes for standalone documentation sites,
  native Docara profiles, and embedded Docara workspaces while retaining the
  compatible legacy project recipe.
- Extended repository validation to cover both supported recipe schema
  generations and their fixture contracts.

- Rebuilt the skill from the current standalone Docara 2 architecture and CLI.
- Replaced legacy project, branding, and GitHub Pages guidance with eight
  outcome-oriented operational references.
- Added current settings inheritance, locale/navigation, component/Smart,
  build/verification, Developer/AI SDK, structured QA, transactional update,
  release, security, and troubleshooting workflows.
- Replaced the historical example graph and copied schema backlog with a small
  source-linked capability graph.
- Replaced date-bound graph checks with deterministic source and graph contract
  validation.
- Kept build, verification, browser acceptance, release, publication, and
  deployment as distinct evidence states.
