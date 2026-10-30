# Docara inside SIMAI Federation

This file only matters when the skill runs inside SIMAI Federation. Without it,
ignore this file and the folders `technologies/`, `quality/` and
`repository-recipes/`.

Use the repo-local Mirai Graph only to select capabilities, relations, gates,
and references. Raw skill sources remain authoritative for method and
judgement. Graph output never grants write, release, or deployment authority.

## Mirai Graph Project Technology

Use the Mirai 2 CLI `mirai technology verify . --significant-work` and
`mirai technology context . --task "..."` (`mirai-graph` is a compatible
alias; inside the Federation, `$simai technology verify`) for verification and
task-scoped context. The exact Mirai release is supplied by the Federation
release lock; raw Docara sources remain authoritative and unavailable context
fails closed.

Executable technology: [documentation-site-lifecycle.json](../technologies/documentation-site-lifecycle.json)
(scenarios: project change, engine upgrade, static publication).

## Companions

Read [skill-mesh-balance.md](./skill-mesh-balance.md) when the task
needs documentation, content, SEO, UX, QA, runtime, or release companions.

## Who owns what this skill does not do

In the Federation these belong to: `docs`, `content`, `marketing`, `ux`, `tester`,
`ops`, `dev` in the product repository, `framework`; graph structure to `graph`
and ordering skill data to `mirai`.

## Release gate for this skill

A changed command, safety sequence, ownership rule or capability of Docara also needs:

1. `skill-change-detect`, `skill-graph-sync`, Federation verify and route check;
2. an exact canonical skill commit pinned by the Federation stable release
   lock.
