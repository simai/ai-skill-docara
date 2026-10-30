# Docara Skill

The canonical English owner skill for building and operating standalone
Docara 3 sites (and moving legacy Docara 2.x projects to 3.x).

The skill understands Docara as a PHP static compiler: Markdown and validated
project configuration become typed document IR, renderer output, admitted
Smart components, composed layouts, static pages, derived indexes, and
verification receipts. It covers:

- project structure and ownership;
- content, locales, routes, navigation, redirects, and translation freshness;
- inherited site, section, page, reading, search, branding, and layout settings;
- inline and reusable examples plus native, typed, container, Framework,
  Docara, and project Smart components;
- full and single-page builds, preview, static verification, and browser QA;
- Developer/AI SDK discovery, scaffolding, validation, tests, structured QA,
  and MCP integration;
- generated project-local capability discovery, verified same-major upgrades,
  offline rollback, low-level engine synchronization, release readiness,
  publication, and troubleshooting.

The skill follows the open [Agent Skills](https://agentskills.io/specification)
format and is licensed under Apache-2.0. It works without any other tooling:
copy `skills/docara` into your agent's skills folder (for example
`~/.claude/skills/docara` or `~/.codex/skills/docara`). `rules/federation.md` and the
folders `technologies/`, `quality/` and `repository-recipes/` only serve SIMAI
Federation and can be ignored.

The installable entry point is [skills/docara/SKILL.md](skills/docara/SKILL.md).
The compact capability graph is [mirai/graph/specs/index.json](mirai/graph/specs/index.json).
The Docara product and the target project's exact installed package remain the
source of truth for schemas and executable behavior.

The skill supports `docara.ai_contract >=1.0.0 <2.0.0`; Docara 3.8.0 still
ships AI contract 1.1.1 in that range. It reads the exact project-local `capabilities --json` result and
does not copy a product-version command or schema catalogue. Older packages
without that command use their exact package documentation as a compatibility
fallback.

## Validation

```bash
python3 scripts/validate_skill_contract.py
bash tests/smoke.sh
```

This repository does not install, enable, release, or deploy itself.
