# Documentation quality method

> Source: simai/docara (https://github.com/simai/docara): docs/site, src/Authoring · Checked: 2026-10-30 · Version: 3.9.1 (tag v3.9.1, 521b035f)

How to plan, write, audit and accept documentation built with Docara so that it
meets the standard of the best public documentation (MDN, Stripe, Tailwind,
Vue, Bootstrap, Laravel). The method is a Diátaxis coverage map, a reference page standard, a
benchmark and docs-only trials with graded solutions. Use it for
any Docara site, including Docara's own documentation.

Read this when the task is to write a documentation set, restructure one,
judge its quality, or prove that a reader can succeed with it. For Docara
mechanics (settings, components, builds) load the other references.

## 1. Principles

- **The reader's task is the unit.** Every page answers a question a named
  reader has: what is it, how do I do X, what exactly can I write, why does it
  work this way. A page that answers none is cut or merged.
- **Ground truth is the release, not memory.** Every class, field, command,
  default, event and path on a page is read from the exact release (schemas,
  CLI help, manifests, built CSS/JS). A fact that cannot be traced is not
  written.
- **Proof is a trial, not a review.** Documentation is good when a reader who
  has only the documentation finishes real tasks. Reviews find style issues;
  trials find the gaps that break work.
- **Generate what restates the release.** Parameter tables, field lists,
  error codes, component catalogues and API tables are generated from the
  release and checked with `--check`, never copied by hand.

## 2. Structure: Diátaxis

Every topic is mapped to four kinds of content, and each page is one kind.

| Kind | Reader's question | Form | Docara pattern |
| --- | --- | --- | --- |
| Tutorial | «Teach me» | One path from nothing to a working result, every step checked | `:::steps`, copyable commands, an expected result after each step |
| How-to | «Help me do X» | A task page: goal, prerequisites, steps, check, troubleshooting | Title starts with a verb; one task per page |
| Reference | «What exactly can I write?» | Complete, generated where possible, same layout on every page | `:::schema_reference`, generated tables, component pages |
| Explanation | «Why is it so?» | Concepts, decisions, trade-offs, links to reference | Prose with diagrams; no step lists |

Rules:

- Do not mix kinds on one page. A how-to links to the reference for fields;
  a reference page does not teach.
- The navigation shows the kinds: a "Getting started" path of tutorials, guide
  sections of how-to and explanation pages, a "Reference" group of reference pages.
  In Docara, list the sections as `header_navigation` items (`id`, `label`,
  `href`) and keep the sidebar to one section with `navigation.scope=section`
  (the default is `site`).
- First success in under 15 minutes: the quick start is its own page, starts
  from an empty folder, uses real versions (no placeholders the reader must
  look up elsewhere) and ends with a visible result.

## 3. Page standards

### Every page

- Front matter: `title`, `description` — one sentence on what the page does
  for the reader. The description is also the search snippet and the AI
  summary.
- `# Title`, then one or two sentences: what it is and when to choose it.
- The first example comes right after the introduction (a live `:::example`
  for components, a command block for tasks).
- Fixed H2 order per page type (below); detail goes into H3.
- `## See also` last, with links to the neighbouring kinds (the how-to links
  the reference, the reference links the how-to).

### Component page (Docara typed components, project components)

1. `## Example` — the component in its most common form, live.
2. `## Markup` — the minimal correct Markdown and the required parts.
3. `## Variants` — one H3 per parameter axis (`layout`, `variant`, `type`,
   `size` …); each H3 shows every value side by side in one live example.
4. `## States` — one H3 per state (open, disabled, loading, error …).
5. `## Accessibility` — names, focus, keys, what a screen reader announces.
6. `## Parameters` — generated from the component catalogue: name, type,
   default, allowed values, deprecated names.
7. `## See also`.

An axis is shown by values, not by prose. Every example whose layout depends
on the window can be opened in the device-width viewer (Docara offers it on
inline Markdown examples and sandboxed examples).

### How-to page

Goal (one sentence) → "What you need" → numbered steps with copyable
commands → "How to check" (an observable result) → "If it did not work"
(symptom → cause → fix, linking error codes) → "See also".

### Reference page

Generated tables first, hand-written notes after. Every field: name, type,
default, allowed values, inheritance level, an example. Error code pages:
code, what happened, why, how to fix, an example of the wrong and the right
input.

### Explanation page

Problem → the model Docara uses → consequences for the reader → links to the
how-to and reference that act on it. Diagrams for flows and layers.

## 4. Language

- Site copy, in the language of the site: plain words, present tense, active voice, the reader as
  "you". One idea per sentence; sentences under ~25 words (Docara's quality
  report flags longer ones).
- No history or plans in pages: no "previously", "now", "in the new version",
  "will", "coming soon", no versions in prose. History goes to the changelog,
  plans to the roadmap.
- One term per thing, the same everywhere; English identifiers stay in code
  spans. Keep a glossary of the product's terms.
- Examples are language-independent where possible (no screenshots with UI
  text; generated text-free illustrations), and every example is valid
  against the release.
- Name parts of the interface exactly as the interface names them.

## 5. Coverage map

Before writing or auditing, build the map and keep it with the project:

1. List the ground truth: every command, setting, component, parameter,
   diagnostic code, event, extension point and output the release has (from
   `capabilities --json`, schemas, the component catalogue, the diagnostic
   registry).
2. List the topics a reader meets, grouped by area (getting started, writing,
   components, settings, build and publish, extend, troubleshoot, AI).
3. For every topic fill four cells — tutorial, how-to, reference,
   explanation — with `yes` (where), `partly` (what is missing) or `no`.
4. Reconcile reference against the release in both directions: shipped
   without a page, and pages describing what does not ship (stale).
5. Score every area 0–10 with one line of justification, and rank the ten
   gaps with the highest reader impact.

## 6. Benchmark criteria

Score the site against the criteria the best documentation meets. Each is
`yes`, `partly` or `no`, with evidence.

| Criterion | What `yes` means |
| --- | --- |
| First success under 15 minutes | Own quick-start page, real versions, visible result, checks after steps |
| Search | Full-text search over every page, synonyms for product terms |
| Navigation by role and by task | Entry points per reader (author, site owner, developer, AI agent) and task-titled how-to pages |
| Copyable, valid examples | Every snippet copies, runs and is checked against the release |
| Live examples | Components shown live, with source, at device widths |
| Reference completeness | Every setting, component parameter, command, code and API listed, generated |
| Errors and troubleshooting | Symptom → cause → fix pages; every diagnostic code has a page |
| Migration and versioning | Upgrade guides per major/minor, deprecation calendar, support policy |
| Accessibility guidance | What the product guarantees and what the author must do |
| Performance and security | Page-weight budgets, CSP, trust boundaries, safe defaults |
| Integrations and hosting | Static hosts, CI, the products it serves |
| AI readiness | `llms.txt`, Markdown twins, machine-readable references, AGENTS guidance |
| Language versions | Parity tracking between locales |
| Freshness | Facts generated or checked against the release on every build |

## 7. Docs-only trial

The acceptance test of a documentation set.

1. **Snapshot** the built documentation (Markdown sources or the site). Name
   the target release.
2. **Write tasks by persona** — 10 to 20 real tasks for the readers the site
   serves (newcomer, author, designer, site owner, developer, AI agent, ops).
   Each task has an observable result.
3. **Solve with the docs only.** A solver (a person or an AI agent) may read
   only the snapshot: no repositories, no source code, no web search, no
   prior knowledge. What the docs do not say is not invented; it is recorded.
4. **Report per task:** result; steps with the pages used; gaps, each marked
   `missing | unclear | contradictory | suspected-wrong` with a quote and what
   was needed; assumptions; confidence (high / medium / low, why); effort
   (pages opened).
5. **Grade** every solution against the release:
   - static: every name, field, value, command and path exists;
   - runtime: build and serve the result, check behaviour, console, keyboard,
     both themes, phone and desktop widths;
   - verdict: works / works with fixes (cosmetic or local) / does not work;
   - compare the solver's confidence with the verdict.
6. **Classify every gap:** real docs gap, docs contradiction, docs error,
   solver error (the docs did answer it), product defect, task wording, not a
   gap. Also list the issues the solvers missed that broke their solutions;
   they are often the most important.
7. **Fix and repeat.** Rank the docs fixes by the tasks they would have saved,
   fix them, and run the next trial on the same and new tasks. Track the pass
   rate from trial to trial.

Acceptance: every task «works» or «works with fixes (cosmetic)», no docs
errors, no contradiction left open, and every product defect reported to its
owner.

## 8. Workflow in a Docara project

1. Build the coverage map (section 5) and the benchmark table (section 6);
   store them in the project (for example `docs/quality/`), not in the site.
2. Fix the structure first: navigation by Diátaxis kind, quick start, task
   titles. Use redirects for moved pages (`redirects.json`).
3. Bring pages to the page standards (section 3); generate reference tables.
4. Run Docara's own gates on every change: `docara qa report` (quality score,
   sentence length, missing descriptions), `verify-static`, page weight, and
   the generated-reference `--check` scripts.
5. Run a docs-only trial (section 7) before calling the set ready, and after
   each major restructure.
6. Record the scores and the trial results with the date and the release, so
   the next audit starts from evidence.

## 9. Report format

A quality report has: inputs and release; the coverage map; reference vs
release reconciliation; the benchmark table; scores per area; the ten
highest-impact gaps; trial results if a trial ran; method notes and limits.
Mark what was measured and what was inferred.
