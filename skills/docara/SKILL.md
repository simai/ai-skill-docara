---
name: docara
description: "Docara 3: отдельный сайт документации и лендинг. docara.json, Markdown-контент, навигация, локали, layouts, компоненты, примеры, Developer/AI SDK, статическая сборка, обновление и переход с 2.x, откат, публикация."
---

# Docara

Own Docara-specific project mechanics. Treat Docara as a PHP compiler that
turns Markdown, validated JSON, admitted design artifacts, and exact-pinned
Smart components into a self-contained static site.

The current product line is Docara 3 (Apache-2.0, `simai/docara:^3.0`; the
references were checked against `v3.8.0`). It bundles a pinned SIMAI Framework
release and needs PHP 8.2+, Composer and Node.js 20+ on the build machine only.
Docara 2.x projects are legacy: move them to 3.x by the manual route in
[update-release-and-publication.md](./references/update-release-and-publication.md#move-a-2x-project-to-3x).

## Authority and freshness

Use this precedence:

1. The target project's files and exact installed `simai/docara` package.
2. The matching package schemas, CLI help, README, and documentation.
3. This skill as the operational map and safety contract.

When a command, schema field, component, or artifact contract may have changed,
inspect the exact installed package instead of guessing. If product sources and
this skill disagree, follow the product and report skill drift.

When this skill is updated, compare the `Версия:` header of every reference
with the latest `simai/docara` release tag and its `CHANGELOG.md`; a newer
minor or major release means the references must be re-checked before they are
re-stamped.

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

Executable technology: [documentation-site-lifecycle.json](./technologies/documentation-site-lifecycle.json)
(scenarios: project change, engine upgrade, static publication).

## Start every task

1. Locate the project root containing `docara.json`.
2. Inspect Git status and preserve unrelated or untracked work.
3. Read `docara.json`, declared locale roots, the nearest relevant
   `section.json`, an optional page sidecar, and the Framework lock reference.
4. Discover the executable from the project (`vendor/bin/docara`) or an exact
   package checkout; do not assume a global binary. Record the installed
   version (`php vendor/bin/docara --version`). A 2.x project goes to the
   2.x -> 3.x route first; `upgrade` never crosses a major version.
5. If the exact binary supports it, run `capabilities --json` and use its real
   commands, schemas, types and lifecycle flags. If it does not, use the
   documented legacy path for that installed package; never assume the latest
   surface.
6. Load only the reference that matches the task.

## Task router

| User outcome | Load |
| --- | --- |
| Create, adopt, or understand a project | [architecture-and-project-model.md](./references/architecture-and-project-model.md) |
| Change site, section, page, branding, layout, reading, search, or locale settings | [settings-and-inheritance.md](./references/settings-and-inheritance.md) |
| Write, restructure, translate, migrate, or track content; change routes or navigation | [content-locales-and-navigation.md](./references/content-locales-and-navigation.md) |
| Use inline or reusable examples; native, typed, container, Framework, Docara, or project Smart components | [components-smart-and-design.md](./references/components-smart-and-design.md) |
| Build one page or the full site, preview, serve, or verify output | [build-preview-and-verification.md](./references/build-preview-and-verification.md) |
| Inspect registries, scaffold artifacts, validate, test, run QA, or use MCP | [developer-sdk-and-qa.md](./references/developer-sdk-and-qa.md) |
| Update the engine, move a 2.x project to 3.x, prepare a package/release, or publish to a static host | [update-release-and-publication.md](./references/update-release-and-publication.md) |
| Diagnose errors, ownership, paths, locks, links, assets, or security boundaries | [security-and-troubleshooting.md](./references/security-and-troubleshooting.md) |
| Plan, write, restructure, audit, or accept a documentation set to the standard of the best public docs (Diátaxis map, page standards, benchmark, docs-only trial) | [documentation-quality-method.md](./references/documentation-quality-method.md) |

Read [rules/skill-mesh-balance.md](./rules/skill-mesh-balance.md) when the task
needs documentation, content, SEO, UX, QA, runtime, or release companions.

## Core execution loop

1. Translate the request into one primary outcome and the smallest complete
   project-owned change.
2. Identify the authoritative input: Markdown, front matter, `docara.json`, a
   `section.json`, a page sidecar, `lang.json`, `redirects.json`, `assets/`,
   reusable `examples/`, `translations.lock.json`, optional
   `docara.authoring.json`, `smart/`, or `design/`.
3. Validate IDs and fields against the exact schema, registry, or `inspect`
   result before editing.
4. Make the change without editing generated or package-owned state.
5. Run the cheapest relevant check, then the required build path.
6. Run `verify-static` on the exact output.
7. For visible behavior, serve the verified bytes over HTTP and test the
   affected routes, navigation, search, themes, responsiveness, and keyboard
   behavior as appropriate.
8. Report changed owners, commands, output path, evidence, and any separate
   publication or acceptance gate.

## Build selection

Use a single-page build only for an existing route after an accepted complete
build and only when the engine, dependency tuple, build purpose, topology, and
global registries remain unchanged:

```bash
php vendor/bin/docara build production --page=/public/route/
```

Use a complete build after adding, deleting, or renaming a route; changing a
reusable example, `docara.json`, locale copy, navigation topology, redirects,
Framework lock, shared registries, section inheritance, or other cross-page
state:

```bash
php vendor/bin/docara build production
php vendor/bin/docara verify-static build_production
```

Build receipts live next to the output in `build_production.docara/`; publish
only `build_production/`. Open exactly those verified bytes over HTTP:

```bash
php vendor/bin/docara serve production \
  --host=127.0.0.1 --port=8000 --no-build
```

## Non-negotiable boundaries

- Never edit `build_*`, build-local `.docara`, or package-owned
  `.docara/engine` as source.
- Never use `init --update`; it is a disabled compatibility guard. Prefer the
  project-local transactional `upgrade` workflow for compatible package
  updates. Use `update --verify -> --dry-run -> --apply` only for low-level
  synchronization of `.docara/engine` from an already selected exact package.
- When a pre-manifest project has no `.docara/engine`, do not start with
  `update --verify`: use the explicit hash-bound
  `update --dry-run --adopt -> update --apply -> update --verify` route, then
  retry `upgrade`.
- Never initialize a non-empty target or use `init` as a legacy converter.
- Never replace an exact Framework lock with a branch, moving URL, or `latest`.
  To move to the bundled release, copy
  `vendor/simai/docara/stubs/portable/simai-framework.lock.json`.
- Never admit a component from catalogue prose alone. Require its real
  manifest/provider/lock/registry contract.
- Never pass PHP classes, callbacks, executable hooks, arbitrary template
  paths, raw project CSS, or unsafe filesystem paths through content/config.
- Never weaken traversal, symlink, hardlink, ownership, namespace, schema,
  hash, slot, prop, or static-output checks to make a build pass.
- Keep secrets and private content out of source and static output.
- Treat `build`, `verify-static`, browser acceptance, release readiness,
  package publication, and deployment as separate states.
- Require `$ops`, backup, rollback, and an explicit live authorization before
  deployment or destructive runtime work.

## Чем навык не занимается

- Метод документации, аудитории, карта и редактура текстов — `docs`; большие
  объёмы текста и локализация — `content`.
- Решения о поисковой видимости — `marketing`; сценарии читателя и интерфейс — `ux`;
  вердикт о готовности — `tester`.
- Хостинг, доступы, бэкап и выкладка на живой сервер — `ops`.
- Код движка Docara (`src/`, схемы пакета, встроенные реестры, тесты
  продукта) — `dev` в репозитории продукта; навык даёт знание о продукте, а не
  владеет репозиторием.
- Сам SIMAI Framework и его компоненты — `framework`; структура графа —
  `graph`; упорядочивание данных навыка — `mirai`.

## Output contract

Return:

- project root and exact package/source revision when relevant;
- source owners changed and the effective scope (`site`, `section`, `page`,
  content, Smart, design, engine, or publication);
- checks and commands actually executed;
- build output and static-verification result;
- browser/QA coverage or a precise reason it was not required;
- unresolved diagnostics, owner review, release, or deployment boundary;
- the smallest safe next step.

Do not call a result released, published, or production-ready unless that
specific state has fresh evidence and authorization.
