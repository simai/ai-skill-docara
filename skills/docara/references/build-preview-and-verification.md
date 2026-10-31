# Build, preview, and verification

> Source: simai/docara (https://github.com/simai/docara): CHANGELOG.md, docs/site/content/ru/build/{php-only,verify,local-preview}.md · Checked: 2026-10-30 · Version: 3.9.1 (tag v3.9.1, 521b035f)

## Contents

- [Preflight](#preflight)
- [Complete build](#complete-build)
- [Single-page build](#single-page-build)
- [Atomic publication](#atomic-publication)
- [Static output](#static-output)
- [Static verification](#static-verification)
- [HTTP and preview](#http-and-preview)
- [Determinism](#determinism)

## Preflight

Run from the project root with the exact selected package. The build machine
needs PHP 8.2+ with its extensions, Composer and Node.js 20+
(`DOCARA_NODE_BINARY` when `node` is not on `PATH`):

```bash
git status --short
php vendor/bin/docara doctor --json
```

Resolve missing PHP/Composer/vendor state explicitly. Do not silently use a
different global Docara checkout. Build performs its own schema, ownership,
path, lock, registry, and input checks, but `doctor` gives an earlier actionable
diagnostic surface.

## Complete build

```bash
php vendor/bin/docara build production
```

The environment suffix controls `build_<environment>`; it does not select a
different content renderer.

At a high level, a complete build:

1. validates the project root, site config, schemas, lock, and providers;
2. discovers every declared locale and physical Markdown owner;
3. resolves inherited settings and provenance for every page;
4. compiles Markdown into typed Document IR and renders it through one registry;
5. derives outline, locale links, backlinks, catalogues, schemas, topology,
   menu, breadcrumbs, previous/next, search, redirects, shared-example assets,
   translation status, and asset plans;
6. composes layout regions and verifies main-document parity;
7. renders the publisher shell and writes all pages/assets/receipts into a
   candidate directory;
8. atomically promotes the candidate only after complete success.

Use a complete build after any structural, global, lock, registry, locale,
route, navigation, redirect, shared search/reading, or section-inheritance
change.

## Single-page build

```bash
php vendor/bin/docara build production --page=/en/guides/install/   # /guides/install/ when the default locale has no prefix
```

Use only when:

- a complete build already exists and has accepted receipts;
- the route already exists in that build;
- build purpose, engine tree, and dependency tuple match;
- topology, search, backlinks, atlas, schemas, locks, and registries remain
  valid for the accepted complete build;
- the page is not a draft;
- `theme.color` did not change (`PORTABLE_INCREMENTAL_THEME_CHANGED`).

After upgrading Docara, run one complete build before using `--page` again.

Docara copies the accepted output into an atomic candidate, recompiles the
selected route through the same `PageBuilder`, and preserves accepted global
projections. It does not make route creation, deletion, renaming, or a global
setting change safe.

## Atomic publication

Build never replaces the current result until the candidate is complete. It
temporarily moves the current output to a rollback directory, promotes the
candidate, and restores the previous output if promotion fails.

This is local build atomicity, not deployment authorization or a persistent
release rollback strategy.

## Static output

A production output normally contains (receipts live next to it, not inside):

```text
build_production/                      # publish this folder as is
├── index.html or locale root redirects
├── 404.html                           # unless not_found.enabled is false
├── <route>/index.html
├── sitemap.xml, robots.txt, llms.txt  # only with site_url
└── _docara/
    ├── search-index.json, search.js
    ├── docara-shell.<sha256>.css      # shared first-frame stylesheets
    ├── docara-theme.<sha256>.css      # only with theme.color
    ├── component-catalog.json
    ├── framework/
    └── vendor/                        # only the parts the pages use
build_production.docara/               # receipts; never publish
├── resolved-page-plans.json
├── redirects.json
├── locale-routes.json
├── performance.json
├── examples.json
├── translation-publication.json       # with translation_tracking publish-current
└── ...
```

Shared assets are published once under `<base_url>_docara/`, not per locale
prefix. Optional vendor parts (Mermaid, KaTeX, Rounded/Sharp icon faces, the
Framework `utility.full.css`) are published only when a built page needs them;
`verify-static` recomputes that choice (`@vendor-asset-selection`). A folder
built by 3.0.0 kept receipts in `build_production/.docara/`; `verify-static`
still reads it, takes `--receipts=DIR` for a deployed copy, and reports
`@build-receipts-in-output` when both exist.

Search files are emitted only when enabled. Assets are local and revisioned by
hash. The static site must not rely on runtime CDN fallback.

Translation issues are warnings in `report` mode and do not invalidate a
successful build. Report them separately from build/static readiness. A changed
reusable example invalidates isolated page rebuilding because every consumer
must move to the same receipt.

## Static verification

```bash
php vendor/bin/docara verify-static build_production
```

The verifier reads static bytes and receipts; it does not execute project PHP
or repair output. It checks:

- manifest/page/output equality and absence of extra HTML surface;
- local routes, links, assets, fragments, and safe fragment encoding;
- duplicate HTML IDs;
- search index/runtime and indexed-page contract;
- redirect receipts, pages, canonicals, and targets;
- component catalogue and Markdown ownership;
- exact Framework and Smart asset projection, shared shell stylesheets and
  the optional theme stylesheet;
- `sitemap.xml`, `robots.txt` and `llms.txt` when `site_url` is set; a portal
  build keeps `llms.txt` and has no `sitemap.xml` or `robots.txt`;
- the `404.html` page like any other page;
- missing, duplicate, symlinked, or hardlinked output.

Each verifier marker and diagnostic code has a cause and a fix in the product
documentation (Reference > Error codes); read it before guessing.

The product repository also has `scripts/verify-page-weight.php <build-dir>`
(page and image byte budgets). It is a maintainer script of the Docara
checkout, not a `vendor/bin/docara` command.

Never fix a verifier error inside `build_production`. Correct the owning
Markdown, JSON, asset, lock, registry, or engine source and rebuild.

## HTTP and preview

Serve the exact verified bytes:

```bash
php vendor/bin/docara serve production \
  --host=127.0.0.1 --port=8000 --no-build
```

Do not use `file://`; routes, query-revisioned assets, search, and fragments
must be checked through HTTP.

Test representative root and nested routes, navigation, search, assets,
fragments, light/dark/system themes, responsive regions, focus, keyboard, and
browser console/network behavior according to impact.

The SDK `preview` command is an isolated production-path artifact preview. It
does not create an accepted full-build receipt and does not replace
`verify-static` or site acceptance.

## Determinism

For release evidence, build from the same immutable source and dependency tuple
in independent clean destinations and compare canonical tree digests. A green
mutable-worktree build or a reused ignored output is not reproducibility
evidence.

Keep these states separate:

```text
build succeeded
!= static output verified
!= browser/QA accepted
!= release package ready
!= published
!= deployed and smoke-tested
```
