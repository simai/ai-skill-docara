# Upgrade, update, release, and publication

> Источник: simai/docara (https://github.com/simai/docara): CHANGELOG.md, docs/site/content/ru/build/{update,upgrade-3,publish,hosting,portal}.md · Проверено: 2026-10-28 · Версия: 3.9.1 (тег v3.9.1, 521b035f)

## Contents

- [Capability handshake](#capability-handshake)
- [Project-local upgrade](#project-local-upgrade)
- [Rollback](#rollback)
- [Move a 2.x project to 3.x](#move-a-2x-project-to-3x)
- [Legacy transition and low-level update](#legacy-transition-and-low-level-update)
- [Release and skill gate](#release-and-skill-gate)
- [Static publication](#static-publication)
- [State boundaries](#state-boundaries)

## Capability handshake

Find the exact project-local executable first:

```bash
php vendor/bin/docara capabilities --json
```

Use the returned command definitions, schemas, lifecycle flags and
`docara.ai_contract` version. One installed canonical skill may serve several
projects on different Docara releases (3.9.0 and 3.9.1 ship AI contract 1.2.0, inside
the supported `>=1.0.0 <2.0.0`); never replace this handshake with assumptions
about the newest package. If an old exact package has no `capabilities`, inspect its help,
README and update documentation and use the legacy path below.

## Project-local upgrade

For a project that owns `composer.json`, `composer.lock` and `vendor/`, the
normal compatible update is:

```bash
git status --short
php vendor/bin/docara upgrade
```

The explicit invocation may use Composer/network. It resolves only a stable
patch/minor version inside the current major and project constraint, installs
an independent candidate, runs candidate doctor, project validation, engine
sync, production build and static verification, then re-hashes every input.
Only after all checks pass may it promote dependencies, engine and the verified
build.

For reviewable automation:

```bash
php vendor/bin/docara upgrade --check --json
php vendor/bin/docara upgrade --to=3.9.1 --dry-run --json
php vendor/bin/docara upgrade --apply=<exact-plan-sha256> --json
```

`--to` must be an exact stable `X.Y.Z`. Reject a branch, moving reference,
prerelease, downgrade, constraint violation or different major. Changed
Composer files, content, examples, assets, settings, translations, Framework
lock, Smart/design source, engine or verified build make the plan stale.
A different major returns `MAJOR_UPGRADE_REQUIRED`; use the manual route
below.

Build, serve, discovery and authoring remain offline. Do not add a background
updater or call upgrade implicitly from those commands.

## Rollback

Both dependency states must exist locally before promotion. Restore with:

```bash
php vendor/bin/docara upgrade --rollback=latest
# or
php vendor/bin/docara upgrade --rollback=<exact-id>
```

Rollback must not need network. It validates the applied lock, vendor, engine
and build hashes before restoring the previous transaction. If a failure occurs
during apply, compensation restores those surfaces automatically and preserves
the last verified build.

## Move a 2.x project to 3.x

`upgrade` stays inside the current major. Move a 2.x project manually; Markdown,
settings, translations and assets are not rewritten:

1. Record `git status --short`, commit or save unfinished work, keep the last
   verified `build_production` for rollback.
2. `composer require simai/docara:^3.0 -W`.
3. `php vendor/bin/docara update --verify`, `--dry-run`, then `--apply` (with
   `--adopt` on the dry run when the project has no ownership manifest).
4. Read `runtime.pair_id` in `simai-framework.lock.json`. `ui-32c7a64e4575`,
   `ui-25beeb814ae8`, `ui-82fc0ba882ba` and the Docara 2.18 lock
   `ui-4f5e5067afeb-smart-7d932ed51408` build with the bundled release; any
   other value falls back to the legacy portable Smart copy, so replace the
   lock with `vendor/simai/docara/stubs/portable/simai-framework.lock.json`
   (a project-owned change: review it).
5. `grep -rn "sf-admin-menu\|sf-table" content`: both components left the free
   Framework release for a commercial SIMAI component package; rewrite those
   pages (for example, a Markdown table instead of `sf-table`).
6. Drop `DOCARA_SIMAI_UI_ROOT` / `SIMAI_UI_ROOT` from build and CI when they
   only pointed at a Framework checkout; make sure Node.js 20+ is present.
7. Complete build, `verify-static`, HTTP check of home, nested pages, search,
   theme and component pages.

Expect these 3.x output differences: shared assets once under
`<base_url>_docara/` (not per locale prefix), receipts in
`build_<environment>.docara/`, built-in page strings in English except on
Russian pages, and `_docara/component-catalog.json` entries with `release`
instead of `revision`/`author`/`changed_at`. Scripts that read
`build_<environment>/.docara/` must move to `build_<environment>.docara/`.
After that, compatible 3.x updates use `upgrade`.

## Pre-manifest transition and low-level update

An existing site without the package-owned `.docara/engine` manifest gets one
explicit project-local Composer runtime and one reviewed engine adoption. Do
not delete or move any older engine automatically:

```bash
cd /path/to/site
composer require simai/docara:^3.0
php vendor/bin/docara capabilities --json
php vendor/bin/docara update --dry-run --adopt --json
php vendor/bin/docara update --apply --json
php vendor/bin/docara update --verify --json
php vendor/bin/docara upgrade --check --json
```

Review the adoption plan before apply. It may create only package-owned
`.docara/engine`; it must not change Markdown, configuration, examples or
assets. If `upgrade` reports `UPGRADE_ENGINE_ADOPTION_REQUIRED`, follow this
same route and retry upgrade. Do not replace the explicit adoption with
`init`, a manual directory copy or an implicit write.

For an already selected exact package, retain the low-level engine-only flow:

```bash
php vendor/bin/docara update --verify --json
php vendor/bin/docara update --dry-run --json
php vendor/bin/docara update --apply --json
php vendor/bin/docara update --rollback=latest
```

Its plan may target only `.docara/engine`. `update` without an action runs
`--verify`. It does not run Composer or change the dependency lock. Unknown files, dirty ownership, conflicts, symlinks or a
stale plan fail before mutation. Never use `init --update`.

## Release and skill gate

Compare the new public AI contract with the previous release. Internal changes
may reuse the current skill. A changed command, safety sequence, ownership rule
or capability requires:

1. an incremented compatible `docara.ai_contract` identity;
2. synchronized canonical raw skill and skill graph;
3. `skill-change-detect`, `skill-graph-sync`, Federation verify and route check;
4. an exact canonical skill commit pinned by the Federation stable release
   lock.

Only then may the Docara package publication gate pass. Federation, not
`docara upgrade`, physically installs that exact skill revision during its own
atomic update. A project command never writes `~/.codex`.

Package preparation remains an exact clean-revision maintainer operation. The
candidate records source revision, planned version/tag, archive hash, file
ledger, dependency inventory and `published=false`. Build it independently in
two clean checkouts and compare byte-exact artifacts. Publication, tag, GitHub
Release and deploy require their own authorization.

## Static publication

Build with the correct `base_url`, run `verify-static`, copy only
`build_<environment>/` (never `build_<environment>.docara/`) to versioned
staging, compare digests, smoke routes/assets/search/themes, then switch
traffic atomically. Use `$ops` for hosting, access, backup, rollback and live
changes. Build success is not deployment authorization.

Publication surfaces added in 3.x (check the installed schema first):

- `site_url` (origin plus `base_url`) adds `sitemap.xml`, `robots.txt`,
  `llms.txt`, absolute canonical/hreflang and Open Graph tags;
  `branding.share_image` adds `og:image`. Content that already ships one of
  these files stops the build (`DISCOVERY_FILE_COLLISION`).
- The build writes `404.html` at the deployment base (off with
  `"not_found": {"enabled": false}`); the host must serve it: nginx
  `error_page 404 /404.html;`, Apache `ErrorDocument 404 /404.html`, GitHub
  Pages automatically.
- A `portal` section builds one path of a larger portal (needs `site_url`, one
  locale, `default_unprefixed` routing; no `sitemap.xml`, `robots.txt` or
  `404.html`). `docara portal-artifact` packs it into a reproducible
  `<product>-<version>-portal.tar.gz`, `.sha256` and receipt.
- Docara's own documentation is published on GitHub Pages
  (https://simai.github.io/docara/) by the product workflow; that is product
  release work for `$dev`, not a project task.

## State boundaries

Report separately:

- package capability contract read;
- compatible upgrade checked/planned/applied/compensated/rolled back;
- low-level engine update planned/applied/rolled back;
- site build succeeded and static output verified;
- browser/QA accepted;
- release package reproducible and skill gate synchronized;
- release authorized/published;
- site staged/deployed/smoke-tested.

Never promote one state into another without its own evidence.
