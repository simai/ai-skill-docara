# Architecture and project model

> Source: simai/docara (https://github.com/simai/docara): CHANGELOG.md, docs/site, resources/schemas, src · Checked: 2026-10-30 · Version: 3.9.1 (tag v3.9.1, 521b035f)

## Contents

- [Mental model](#mental-model)
- [Compiler pipeline](#compiler-pipeline)
- [Project files](#project-files)
- [Ownership](#ownership)
- [Initialize a project](#initialize-a-project)
- [Source discovery](#source-discovery)

## Mental model

Docara is a standalone PHP static-site compiler. It consumes Markdown,
validated JSON, admitted design definitions, exact-pinned Smart artifacts, and
project assets. It emits static HTML, CSS, JavaScript, assets, search data,
redirects, and machine-readable receipts. The published site has no database,
backend, daemon, or CDN dependency.

Docara 3 (`simai/docara:^3.0`, Apache-2.0) builds with PHP 8.2+ and the
extensions `ctype`, `dom`, `intl`, `json`, `libxml`, `mbstring`, Composer, and
Node.js 20+. Node.js runs the SIMAI Framework Composition Recipe runtime at
build time; set `DOCARA_NODE_BINARY` when `node` is not on `PATH` (otherwise
the build stops with `COMPOSITION_RECIPE_RUNTIME_REQUIRED`). The published site
stays static. Optional browser/visual QA may use additional tooling.

The package bundles a pinned public SIMAI Framework release (Core, Smart and
the Recipe runtime from one `simai/ui` commit): `ui-32c7a64e4575` since 3.6.0. No Framework
checkout is needed; `DOCARA_SIMAI_UI_ROOT` and `SIMAI_UI_ROOT` are only an
optional way to build with another distribution of the same Recipe
specification. Always read the exact pair from the project lock and the
installed package rather than from this list.

## Compiler pipeline

Use this architecture when diagnosing ownership or impact:

```text
Markdown source
-> typed in-memory Document IR
-> Document Renderer Registry
-> Smart Component Gateway when a Smart leaf is present
-> Layout / Region / Section / Block composition
-> publisher shell
-> HTML + assets + derived views + receipts
-> atomic build directory promotion
```

The same `PageBuilder` compiles a route in full and single-page builds. The
build modes differ by selected route set and accepted global context, not by a
second renderer.

Cross-page surfaces are derived from the same physical Markdown route set:

- canonical topology and visible navigation;
- breadcrumbs and previous/next links;
- outlines and heading fragments;
- backlinks and component indexes;
- locale alternates;
- search index;
- redirects and public route receipts.

Do not maintain a second public page catalogue or hand-edit derived output.

## Project files

```text
composer.json
composer.lock
vendor/
docara.json
docara.authoring.json                  # optional page profiles
redirects.json                         # optional when no explicit redirects exist
simai-framework.lock.json
contracts/generated/                   # created by init; commit it
assets/
examples/                              # optional shared HTML/CSS/JS demonstrations
translations.lock.json                 # optional accepted translation state
content/
  en/
    lang.json
    section.json                       # optional
    index.md
    404.md                             # optional; replaces the built-in 404 page
    guides.md
    guides/
      section.json                     # optional
      install.md
      install.page.json                # optional
smart/                                 # optional project-owned Smart artifacts
design/                                # optional project-owned design artifacts
.docara/
  engine/                              # package-owned state
build_local/                           # generated
build_production/                      # generated; publish this folder
build_production.docara/               # generated receipts; never publish
```

Use the flat section form by default: `guides.md` owns `/guides/`, while the
sibling `guides/` directory contains descendants. `guides/index.md` is also
supported, but both forms for one route are an error.

## Ownership

Project-owned authoring surfaces:

- `docara.json`, `redirects.json`, and the consumer-owned `composer.lock`;
- `content/**`, including Markdown, `section.json`, page sidecars, `lang.json`,
  and colocated content assets;
- root `assets/**`;
- reusable `examples/**` and a configured `translations.lock.json`;
- admitted project `smart/**` and `design/**` sources;
- the exact Framework lock selected by the project.

Package-owned or generated surfaces:

- `.docara/engine/**` and its ownership manifest;
- update plans and rollback packages;
- `build_*` and the receipts in `build_<environment>.docara/`;
- generated `_docara/**` assets, search data, and catalogues.
- generated `build_<environment>.docara/examples.json` and translation status.

Use product commands to update package-owned state. Never fix source behavior
by editing generated output.

## Initialize a project

Create the exact Composer runtime in the project itself, then run `init` in the
same directory:

```bash
mkdir /path/to/project && cd /path/to/project
composer require simai/docara:^3.0
php vendor/bin/docara init .            # --template=minimal|showcase --lang=en|ru
php vendor/bin/docara doctor --json
php vendor/bin/docara build production
php vendor/bin/docara verify-static build_production
```

The default starter is `--template=minimal --lang=en`: home, writing and
components pages, one locale, unprefixed routes. The 2.x starter (component
catalogue with project Smart demos) is `--template=showcase --lang=ru`. The
template is chosen only at the first `init`.

`init` accepts an empty directory or one containing only that verified
project-local Composer runtime; `.git/`, `.gitignore`, `.gitattributes`, `AGENTS.md` and
file-manager files (`.DS_Store`, `Thumbs.db`, `desktop.ini`) do not count, so
`git init` before `init` works. Anything else in the folder, such as `.claude/`, makes
`init` refuse: run it first, or in a new folder. If Composer offers only 2.x, clear its cache
and remove a stale `repositories` entry (see the product troubleshooting page).
`init --update` is disabled.

Commit everything `init` created, including `.docara/engine/` and
`contracts/generated/`; ignore `/vendor/`, `/build_*/` (this also covers
`build_*.docara/`), `/.docara-preview/`, `/.docara-qa/` and `/.docara/upgrades/`. Updating an
existing project uses the transactional `upgrade` workflow described in
[update-release-and-publication.md](./update-release-and-publication.md).

Do not treat `init` as an importer for a legacy static-site generator, an older
Docara project model, or an arbitrary documentation tree. Inventory and map
legacy sources first, then create a new portable project or an explicit
migration plan.

## Source discovery

Before a change:

1. Locate `docara.json` and record the project root.
2. Record Git status and the exact installed package/source revision.
3. Read declared locale roots and the Framework lock path.
4. Resolve the requested public route to its one physical Markdown owner.
5. Read every inherited `section.json` from locale root to the page and the
   optional page sidecar.
6. Use `docara inspect`, `schema`, or build receipts for exact effective
   contracts.

If local dependencies are unavailable, report the missing runtime instead of
silently switching to another checkout or stale global Docara installation.
