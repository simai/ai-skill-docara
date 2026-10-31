# Markdown blocks and inline directives

> Source: simai/docara (https://github.com/simai/docara): docs/site/content/ru/components, docs/site/content/ru/authoring/markdown.md, docs/site/content/ru/reference/errors/content.md, src/PortableSite/PortableMarkdownRenderer.php, src/Markdown, src/Document, scripts/verify-static-build.php, resources/components, resources/component-catalog/typed · Checked: 2026-10-31 · Version: 3.9.1 (tag v3.9.1, 521b035f)

## Contents

- [Syntax rules](#syntax-rules)
- [Native Markdown additions](#native-markdown-additions)
- [Inline directives](#inline-directives)
- [Notices and disclosure](#notices-and-disclosure)
- [Sequences and alternatives](#sequences-and-alternatives)
- [Code, examples, and formulas](#code-examples-and-formulas)
- [Images, embeds, and files](#images-embeds-and-files)
- [Layout containers](#layout-containers)
- [Landing blocks](#landing-blocks)
- [Generated blocks](#generated-blocks)
- [Framework and project components](#framework-and-project-components)

Error codes below are what `docara build` reports at this version. The message names the source file.

## Syntax rules

A block starts at column 0 and closes with a line of the same number of colons:

```markdown
:::name {attr=value other="value with spaces"}
Markdown body.
:::
```

- Put one space before `{`. `:::alert{type=info}` is not a block: it stays visible text and the build passes.
- An unknown name, such as `:::note`, stops the build with `DOCUMENT_COMPONENT_ALIAS_UNKNOWN`.
- Attribute names are lowercase. Quote values with spaces. Empty values and duplicates are rejected.
- `tabs`, `html`, `features`, `showcase`, `promo`, `columns`, and `cta` have no attributes and silently ignore any you add. Every other block rejects unlisted attributes.
- Give the outer block more colons when nesting: `::::grid` around `:::card`. Equal counts close the outer block early.
- Only [layout containers](#layout-containers) contain other blocks. Inline directives work in Markdown bodies.
- A block in a list gives `MARKDOWN_BLOCK_INDENTATION_UNSUPPORTED`. A block in a quote stays visible text.
- A page holds at most 128 block openings (`DOCUMENT_DIRECTIVE_LIMIT_EXCEEDED`).
- Inside a fenced code block, directives stay literal. Use a longer outer fence to show a fence.
- Raw HTML in Markdown gives `MARKDOWN_RAW_HTML_FORBIDDEN`. For markup, use [`html`](#html) or [`example`](#example).
- A `javascript:` or `data:` link in ordinary text loses its address silently and the build passes. Block links with such schemes stop the build, as in `MARKDOWN_CTA_LINK_UNSAFE`.
- Inline form: `:name[content]{attr=value}`, with `{...}` optional. It stays literal inside backticks, after a backslash (`\:badge[...]`), or directly after a letter or digit.

| Code | Cause |
| --- | --- |
| `DOCUMENT_TYPED_DIRECTIVE_UNCLOSED`, `DOCUMENT_COMPONENT_BLOCK_UNCLOSED`, `DOCUMENT_SMART_UNCLOSED` | No closing line with the same colon count |
| `MARKDOWN_BLOCK_EMPTY`, `DOCUMENT_COMPONENT_BLOCK_CONTENT_REQUIRED`, `DOCUMENT_CONTAINER_CHILD_COUNT_MIN` | Empty body in a block that needs content |
| `DOCUMENT_TYPED_DIRECTIVE_NESTED_FORBIDDEN`, `DOCUMENT_CONTAINER_CHILD_FORBIDDEN`, `DOCUMENT_CONTAINER_SMART_CHILD_FORBIDDEN` | A block inside a block that does not admit it |
| `MARKDOWN_COMPONENT_ATTRIBUTES_INVALID` | Malformed `{...}`, such as an unclosed quote |
| `MARKDOWN_COMPONENT_ATTRIBUTE_UNKNOWN`, `MARKDOWN_COMPONENT_ATTRIBUTE_VALUE_INVALID`, `MARKDOWN_COMPONENT_ATTRIBUTE_DUPLICATE` | Bad attribute. `alert` reports `CONTENT_COMPONENT_PROP_UNKNOWN` and `CONTENT_COMPONENT_PROP_INVALID` for the first two. |

## Native Markdown additions

Headings, emphasis, lists, links, tables, footnotes (`[^id]`), and fenced code are standard. For front matter, see `content-locales-and-navigation.md`.

**Quote attribution.** Put the attributes on the line after the quote. `author` or `source` is required, and a line with neither stays visible text. `url` is optional.
```markdown
> Define the reader's task first, then explain the system.
{author="Article author" source="Publication name" url="https://example.com/source"}
```

**Image frame.** Put the attributes right after the image, in its own paragraph. `ratio`: `auto` (default), `1x1`, `4x3`, `3x2`, `16x9`, `21x9`; the slash form `16/9` also works. `fit`: `cover` (default), `contain`.
```markdown
![Build flow](../assets/flow.png){ratio=16x9 fit=contain}
```

**Fenced code.** The language after the fence sets the header and highlighting. Code never runs. To hide the wrap button, set `"code": {"wrap": false}` in site, section, or page settings. To include a file, see [`code`](#code).

## Inline directives

Empty content gives `MARKDOWN_INLINE_COMPONENT_LABEL_REQUIRED`. Bad attributes give `CONTENT_COMPONENT_PROP_UNKNOWN` or `CONTENT_COMPONENT_PROP_INVALID`.

### badge
Short status, version, or category label.
```markdown
:badge[New]{type=tonal scheme=primary size=1}
```
- `type`: `main`, `tonal` (default), `outline` `scheme`: `primary` (default), `secondary`, `tertiary`, `neutral`, `info`, `success`, `warning`, `danger`, `on-surface` `size`: `1/3` (default), `1/2`, `1`

### button
A link styled as a button. For an interface button, see [`ui.button`](#uibutton).
```markdown
:button[Open components]{href=/components/ type=outline scheme=on-surface size=1 icon=arrow_forward}
```
- `href`: required, safe URL. Missing or unsafe gives `CONTENT_COMPONENT_PROP_INVALID`. `type`: `default` (default), `tonal`, `outline`, `link`. `scheme`: `primary` (default), `secondary`, `on-surface`. `size`: `1/2`, `1` (default), `2`. `icon`: Material Symbols name. It is not checked: a typo shows as text.

### icon
A Material Symbols icon. The content is its name in lowercase Latin letters, digits, and `_`.
```markdown
:icon[check_circle]{size=3 family=rounded weight=medium filled=true label="Done"}
:icon[bolt]{size=1 container=circle variant=main scheme=success label="Fast"}
```
- `size`: `1/3`, `1/2` (default), `1`, `2`, `3`. `family`: `outlined` (default), `rounded`, `sharp`. `weight`: `light`, `regular` (default), `medium`. `filled`: `true`, `false` (default). `label`: screen reader text. Set it when the icon carries meaning alone. `container`: `none` (default), `square`, `circle`. `variant`: `auto` (default), `plain`, `main`, `tonal`, `outline`. `scheme`: same values as `badge`.
- Errors: `CONTENT_COMPONENT_ICON_UNKNOWN` for a name outside the set (`:icon[setings]`). `CONTENT_COMPONENT_SLOT_INVALID` for capitals or spaces. `CONTENT_COMPONENT_PROP_COMBINATION_INVALID` for `main`, `tonal`, or `outline` without a container, or `plain` with one.

### kbd
A keyboard key, with no attributes. Write each key separately.
```markdown
Press :kbd[Ctrl] + :kbd[C], then :kbd[Enter].
```

## Notices and disclosure

### alert
A notice next to the text. The body starts with a level 2-5 heading, with text after it.
```markdown
:::alert {type=warning variant=outlined}
#### Check the settings
Review the parameters before publishing.
:::
```
- `type`: `clear`, `info` (default), `success`, `warning`, `danger`. `variant`: `default` (default), `flat`, `outlined`.
- Errors: `CONTENT_COMPONENT_BLOCK_HEADING_REQUIRED`, `CONTENT_COMPONENT_BLOCK_CONTENT_REQUIRED`.

### banner
A short announcement for the whole page.
```markdown
:::banner {type=warning}
Save the current build before updating.
:::
```
- `type`: `info` (default), `success`, `warning`, `danger`. There is no `clear`, unlike `alert`.

### details
A collapsible explanation. The first heading, level 2-4, becomes the summary.
```markdown
:::details {view=lines open=true}
## What happens after the build?
Docara creates a static site that needs no server runtime.
:::
```
- `view`: `surface` (default), `lines`. `open`: `true`, `false` (default).
- Errors: `MARKDOWN_DETAILS_HEADING_REQUIRED`, `MARKDOWN_DETAILS_CONTENT_REQUIRED`.

## Sequences and alternatives

### steps
A short procedure. The body is exactly one numbered list (`MARKDOWN_STEPS_ORDERED_LIST_REQUIRED`).
```markdown
:::steps {view=timeline current=2}
1. Create the project.
2. Add pages.
3. Publish the site.
:::
```
- `view`: `timeline` (default), `list`. `current`: step number from 1, default 1, used by `timeline`. A non-number counts as 1.

### tabs
Equal alternatives. Each `###` heading is a tab label with content under it.
````markdown
:::tabs
### Linux
```bash
php vendor/bin/docara build production
```

### Result
The finished site appears in the build directory.
:::
````
- Errors: `MARKDOWN_TABS_ITEMS_REQUIRED` for fewer than two `###` tabs. `MARKDOWN_TABS_ITEM_INVALID` for a tab without content.

### tree
A file and folder tree. The body is one bulleted list (`MARKDOWN_TREE_LIST_REQUIRED`).
```markdown
:::tree {interactive=true}
- content
  - en
    - index.md
- docara.json
:::
```
- `interactive`: `true` (default, branches collapse), `false`

## Code, examples, and formulas

### code
Inserts part of a project file at build time. The body is empty.
```markdown
:::code {src="../../snippets/install.php" lang=php lines="1-8" title="install.php"}
:::
```
- `src`: required, relative to the page file's folder, inside the project. `lang`: lowercase letters, digits, `_`, `+`, `-`, up to 32 characters. Defaults to the file extension, or `text`. `lines`: `4` or `4-12`, from 1. An end past the last line is cut to the file end. `title`: defaults to the file name.
- Errors: `MARKDOWN_CODE_SOURCE_REQUIRED`. `MARKDOWN_CODE_SOURCE_UNSAFE` for an absolute path, outside the project, a symlink, or a missing file. `MARKDOWN_CODE_SOURCE_INVALID` for over 1 MiB or not UTF-8. `MARKDOWN_CODE_LINES_INVALID`. `MARKDOWN_CODE_LANGUAGE_INVALID`.

### example
A rendered result next to its source. The body is fenced code only.

Inline Markdown example, with exactly one `markdown` fence:
````markdown
:::example {label="Result"}
```markdown
:badge[Ready]{type=main scheme=success size=1/2}
```
:::
````
Inline interface example, with `html` required and `css` and `javascript` optional:
````markdown
:::example {label="Interactive" preview=sandbox}
```html
<button class="demo-button" type="button">Press</button>
```
```css
.demo-button { padding: .5rem 1rem; }
```
:::
````
Shared example, stored once at `examples/<id>/index.html` in the project, with optional `index.css`, `index.js`, and `assets/`. The body is empty:
```markdown
:::example {id="utilities/animation-duration" label="Result"}
:::
```
- `id`: path under `examples/`, lowercase. `label`: result tab name, default `Example`. `preview`: `auto` (default), `inline`, `sandbox`. `auto` inlines Markdown and safe HTML without CSS or JS, and isolates the rest in an iframe. `fullscreen`, `wrap`: `true` or `false`, showing or hiding the responsive-check and line-wrap buttons. Set defaults under `"examples"` in site, section, or page settings.
- Inlined `html` cannot use `on*`, `href`, `src`, `style`, or elements such as `a`, `img`, `script`, `iframe`. `auto` sends such markup to the iframe.
- Errors: `MARKDOWN_EXAMPLE_FENCED_SOURCE_REQUIRED`. `MARKDOWN_EXAMPLE_SOURCE_ONLY` for text outside fences. `MARKDOWN_EXAMPLE_HTML_REQUIRED` for CSS or JS without HTML. `MARKDOWN_EXAMPLE_SOURCE_COMBINATION_INVALID` for `markdown` mixed with `html`. `MARKDOWN_EXAMPLE_SOURCE_DUPLICATE`. `MARKDOWN_EXAMPLE_ID_BODY_CONFLICT`. `MARKDOWN_EXAMPLE_INLINE_NOT_ADMITTED` for `preview=inline` on content that needs isolation. `MARKDOWN_EXAMPLE_INLINE_ID_DUPLICATE` for a repeated HTML `id`.

### html
Trusted static HTML in a sandboxed iframe, with no scripts and no site styles. The body is exactly one `html` fence (`MARKDOWN_HTML_FENCE_REQUIRED`).
````markdown
:::html
```html
<p><strong>Isolated HTML</strong> with no access to the page.</p>
```
:::
````

### math
A TeX formula rendered by bundled KaTeX. Write it without `$`.
```markdown
:::math {display=block label="Area of a circle"}
S = \pi r^2
:::
```
- `display`: `block` (default), `inline`. `label`: screen reader text, default `Mathematical formula`.

### diagram
A Mermaid 11.16 diagram drawn in the browser. A Mermaid syntax error does not stop the build: the source text stays visible.
```markdown
:::diagram {engine=mermaid title="Publishing documentation"}
flowchart LR
  Author --> Build
  Build --> Verify
  Verify --> Publish
:::
```
- `engine`: `mermaid` (default, the only value). `title`: shown above the diagram and read by screen readers.
- Types in the docs: `flowchart`, `sequenceDiagram`, `stateDiagram-v2`, `journey`, `classDiagram`, `erDiagram`, `requirementDiagram`, `mindmap`, `treemap-beta`, `block-beta`, `packet-beta`, `C4Context`, `architecture-beta`, `gantt`, `timeline`, `gitGraph`, `kanban`, `pie`, `xychart-beta`, `quadrantChart`, `sankey-beta`, `radar-beta`. `click` and `href` in a diagram are ignored. `zenuml` is not bundled. The docs say `sankey-beta` needs Latin labels, and `architecture-beta` and `xychart-beta` need non-Latin labels in double quotes.

## Images, embeds, and files

Keep images in `content/<locale>/assets/` and link them relative to the page's public address: `../assets/flow.png` from `content/en/guide.md` (route `/guide/`), with one more `../` per subfolder. `hero` and `surface` backgrounds instead use `/assets/...` from the project's root `assets/` folder. A missing image in ordinary Markdown passes `build`, and `verify-static` reports it.

### figure
One image with an optional caption paragraph (`MARKDOWN_FIGURE_IMAGE_REQUIRED` otherwise). Alt text is not enforced, so write it yourself.
```markdown
:::figure {ratio=4/3 fit=cover}
![Build flow](../assets/flow.png)

A short caption adds to the image.
:::
```
- `ratio`: `auto` (default), `1/1`, `4/3`, `3/2`, `16/9`, `21/9`. `fit`: `cover` (default), `contain`.

### media
Short text beside one image, stacking on narrow screens. Without either, it gives `MARKDOWN_MEDIA_CONTENT_REQUIRED`.
```markdown
:::media {side=right ratio=4/3}
## A clear heading
A short explanation of how the text and image relate.

![Docara interface](../assets/screen.png)
:::
```
- `side`: `left`, `right` (default). `ratio`: `1/1`, `4/3`, `3/2`, `16/9` (default).

### embed
External material in a responsive frame. The body is one link, and its text labels the load button.
```markdown
:::embed {provider=map ratio=4/3 title="Interactive map"}
[Open the map](https://www.openstreetmap.org/export/embed.html)
:::
```
- `provider`: `generic` (default) takes a same-site path such as `/demo/widget/`. `video` takes `https` on `youtube.com`, `www.youtube.com`, `youtube-nocookie.com`, `www.youtube-nocookie.com`, `player.vimeo.com`. `map` takes `https` on `openstreetmap.org`, `www.openstreetmap.org`. `external` takes any `https`. `consent`: `required` (default), `none`. `none` loads at once and is allowed only for same-site addresses. `ratio`: `1/1`, `4/3`, `16/9` (default), `21/9`. `title`: iframe name, always set it. `id`: optional anchor, up to 80 letters, digits, `.`, `_`, `-`.
- Errors: `MARKDOWN_EMBED_ORIGIN_FORBIDDEN` for an address that does not fit the provider or consent. `MARKDOWN_EMBED_LINK_REQUIRED`, `MARKDOWN_EMBED_LABEL_REQUIRED`, `MARKDOWN_EMBED_URL_UNSAFE`, `MARKDOWN_EMBED_ID_INVALID`.

### download
A file download with its details visible first. The body is one link (`MARKDOWN_DOWNLOAD_LINK_REQUIRED`).
```markdown
:::download {name="archive.zip" format="ZIP" size="4.8 MB"}
[Download the archive](../assets/archive.zip)
:::
```
- `name`, `format`, `size`, `checksum`: optional display text, not checked against the file. `action`: `download` (default), `open`.

## Layout containers

### card
A heading and related text on a surface. The only block it may contain is `figure`.
```markdown
:::card {variant=default}
### Portable project
Keep Markdown and JSON next to the code.
:::
```
- `variant`: `default` (default), `plain`, which removes the border and padding.

### grid
Responsive columns of 1 to 12 cards, one column on phones. Children must be `card` (`MARKDOWN_GRID_CARD_REQUIRED`, `DOCUMENT_CONTAINER_CHILD_FORBIDDEN`).
```markdown
::::grid {columns=2 gap=2}
:::card
#### Content
The first block.
:::
:::card
#### Composition
The second block.
:::
::::
```
- `columns`: `1`, `2`, `3` (default), `4`. `gap`: `0`, `1`, `2` (default), `3`, `4`.

### surface
A styled band around Markdown. It may contain `grid`, `card`, `steps`, and project Smart components that declare `content.embeddable`, but not `ui.*`, `alert`, `figure`, `surface`, `hero`, `showcase`, or `promo`.
```markdown
:::surface {width=full content_width=container padding=lg tone=muted}
## One content rhythm
Text inside the full band aligns with the rest of the page.
:::
```
Nested form:
```markdown
:::::surface {width=content padding=md}
::::grid {columns=2}
:::card
#### First
Text.
:::
::::
:::::
```
- `width`: `content` (default), `full`, which reaches the edge of its layout region only. `content_width`: `container` (default), `full`. `padding`: `none`, `sm`, `md` (default), `lg`, `xl`. `tone`: `default` (default), `muted`, `accent`, `contrast`. `background_image`: decorative, local `/assets/...` path only. `background_fit`: `cover` (default), `contain`, `auto`. `background_x`: `left`, `center` (default), `right`. `background_y`: `top`, `center` (default), `bottom`. `overlay`: `none` (default), `light`, `dark`. `overlay_strength`: `soft`, `medium` (default), `strong`.
- Errors: `MARKDOWN_SURFACE_BACKGROUND_REQUIRED` for position attributes or an overlay without `background_image`. `MARKDOWN_SURFACE_OVERLAY_REQUIRED` for `overlay_strength` without an overlay. `MARKDOWN_SURFACE_BACKGROUND_UNSAFE` for a remote or missing file.

## Landing blocks

These do not nest in other blocks.

### hero
The first screen: one H1, one or two paragraphs, up to two action links, then an optional image.
```markdown
:::hero {variant=split media=side}
# Documentation that is easy to read
One meaningful screenshot stays part of the content.

[Open the components](/components/)

![Documentation interface](/assets/screen.png)
:::
```
Background image, with an empty alt:
```markdown
:::hero {media=background background_fit=cover overlay=dark overlay_strength=medium}
# Documentation without extra complexity
The content stays readable over the overlay.

![](/assets/screen.png)
:::
```
Animated effect:
```markdown
:::hero {variant=centered padding=2xl effect=constellation-net effect_theme=dark effect_settings='{"speed": 80}'}
# Documentation that is easy to read
The effect is drawn under the text.
:::
```
- `variant`: `split` (default), `centered`, `compact`. `padding`: `lg` (default for `compact`), `xl` (default otherwise), `2xl`. `media`: `auto` (default), `side` (needs `variant=split` and alt text), `background` (needs empty alt), `none`.
- With `media=background` only: `background_fit` `cover` (default), `contain`, `auto`. `background_x` `left`, `center` (default), `right`. `background_y` `top`, `center` (default), `bottom`. `overlay` `light`, `dark` (default). `overlay_strength` `soft`, `medium` (default), `strong`. `effect`: id from the catalogue at https://effects.simai.io/, or an `https` address of its `effect.json`. The id is not checked at build time. It needs `"effects": {"catalogue": "https://cdn.jsdelivr.net/gh/simai/ui-effects@<commit>/"}` in `docara.json`, or the build stops with `EFFECTS_CATALOGUE_REQUIRED`. `effect_theme`: `auto` (default), `dark`, `light`. `effect_settings`: JSON object in single quotes.
- Errors: `MARKDOWN_HERO_H1_REQUIRED`. `MARKDOWN_HERO_DESCRIPTION_REQUIRED` for not one or two paragraphs. `MARKDOWN_HERO_STRUCTURE_INVALID` for wrong order, lists, or a third link. `MARKDOWN_HERO_MEDIA_VARIANT_INCOMPATIBLE`, `MARKDOWN_HERO_SIDE_ALT_REQUIRED`, `MARKDOWN_HERO_BACKGROUND_ALT_FORBIDDEN`, `MARKDOWN_HERO_MEDIA_IMAGE_REQUIRED`, `MARKDOWN_HERO_MEDIA_IMAGE_FORBIDDEN`. `MARKDOWN_HERO_BACKGROUND_MODE_REQUIRED` for background attributes without `media=background`. `MARKDOWN_HERO_EFFECT_REQUIRED` for theme or settings without `effect`. `MARKDOWN_HERO_EFFECT_MEDIA_CONFLICT`, `MARKDOWN_HERO_EFFECT_SETTINGS_INVALID`.

### features
Cards from one flat bulleted list of 2 to 6 one-paragraph items. A leading icon or image sits above the text, and leading bold text becomes the title.
```markdown
:::features
- :icon[bolt]{size=2 container=square variant=tonal scheme=primary} **Fast.** The build outputs ready HTML, CSS, and JavaScript.
- :icon[search]{size=2 container=square variant=tonal scheme=primary} **Search.** The index runs in the reader's browser.
:::
```
- Errors: `MARKDOWN_FEATURES_ITEM_COUNT_INVALID`, `MARKDOWN_FEATURES_ITEM_CONTENT_INVALID`.

### showcase
A large result: an H2, one or two paragraphs, at most one link, and one image with alt text as the last line.
```markdown
:::showcase
## See the result before publishing
Docara builds the documentation interface from Markdown and JSON.

[Open the components](/components/)

![Docara documentation page](../assets/screen.png)
:::
```
- Errors: `MARKDOWN_SHOWCASE_IMAGE_REQUIRED`, `MARKDOWN_SHOWCASE_IMAGE_ALT_REQUIRED`, `MARKDOWN_SHOWCASE_STRUCTURE_INVALID`.

### promo
The closing call to action: an H2, one or two paragraphs, exactly one link, and an optional last-line image.
```markdown
:::promo
## Build your first site
The guide goes from an empty folder to a page in the browser.

[Open the components](/components/)
:::
```
- Errors: `MARKDOWN_PROMO_LINK_REQUIRED`, `MARKDOWN_PROMO_STRUCTURE_INVALID`.

### columns
Two to four parts of Markdown side by side, split by a `---` line with a blank line on each side.
```markdown
:::columns
### Markdown
Content stays readable source text.

---

### JSON
Settings are inherited and checked by schemas.
:::
```
- Errors: `MARKDOWN_COLUMNS_REGION_COUNT_INVALID`, also when the blank lines are missing. `MARKDOWN_COLUMNS_SEPARATOR_INVALID`, `MARKDOWN_COLUMNS_REGION_EMPTY`.

### cta
One link shown as a prominent button (`MARKDOWN_CTA_LINK_REQUIRED`, `MARKDOWN_CTA_LINK_UNSAFE`).
```markdown
:::cta
[Open the component catalog](/components/)
:::
```

### logos
A row from one flat bulleted list of 2 to 12 items. Each is text, a link, or an image with alt text.
```markdown
:::logos {tone=muted}
- [GitHub](https://github.com/)
- ![Docara](../assets/mark.svg)
- Plain text member
:::
```
- `tone`: `normal` (default), `muted`.
- Errors: `MARKDOWN_LOGOS_ITEM_COUNT_INVALID`, `MARKDOWN_LOGOS_ITEM_CONTENT_INVALID`.

## Generated blocks

Docara fills these from site data. Only `internal_preview` has a body.

### backlinks
Lists pages of this build that link to the current page. `limit`: 1 to 50, default 5 (`MARKDOWN_BACKLINKS_LIMIT_INVALID`).
```markdown
:::backlinks {limit=10}
:::
```

### component_index
Lists the catalog's components, with no attributes. Use it only on a page whose address ends in `/components/`, because `verify-static` fails elsewhere.
```markdown
:::component_index
:::
```

### atlas_index
Lists Design Atlas entries. Filters: `kind`, `authoring`, `support`, `status`, `namespace`, `owner`, `origin`, `provider`, `ids`, each a comma-separated list of lowercase ids (`MARKDOWN_ATLAS_FILTER_INVALID`). For values, run `php vendor/bin/docara atlas --json`.
```markdown
:::atlas_index {kind=smart owner=ui support=supported}
:::
```

### schema_reference
A settings table from a Docara JSON schema.
```markdown
:::schema_reference {schema=site scope=site fields=title,preset}
:::
```
- `schema`: required, `site`, `section`, `page`, `presentation`, `framework-lock`. `scope`: required, `site`, `section`, `page`, `shared`, `lock`. `fields`: top-level fields, comma-separated with no spaces (`MARKDOWN_SCHEMA_REFERENCE_FIELDS_INVALID`).

### internal_preview
Frames another page of the same site. The body is one link starting with `/`.
```markdown
:::internal_preview {size=compact title="Writing guide preview"}
[Open the page](/writing/)
:::
```
- `size`: `compact`, `normal` (default), `tall`. `title`: frame name for screen readers, with a default if omitted.
- Errors: `MARKDOWN_INTERNAL_PREVIEW_LINK_REQUIRED`, `MARKDOWN_INTERNAL_PREVIEW_URL_UNSAFE`.

## Framework and project components

Prefer the blocks above. Use `ui.*` only when the page needs the real Framework element. The body is one JSON object, and `{...}` on the opening line does not work.

Treat `manifests` in `simai-framework.lock.json` as the contract. A new project lists `ui.button` and `ui.alert`. Inspect yours with `php vendor/bin/docara atlas --json` and `php vendor/bin/docara inspect smart ui.button --json`.

### ui.button
An interface button, not a link. To navigate, use [`button`](#button) or [`cta`](#cta).
```markdown
:::ui.button
{"preset":"primary","text":"Continue","aria-label":"Continue setup"}
:::
```
- `preset`: `primary`, `secondary`, `tonal_on_surface`, `outline`, `outline_on_surface`, `link`, `link_on_surface`, `on_surface`. Or set `type` (`default`, `tonal`, `outline`, `link`) and `scheme` (`primary`, `secondary`, `on-surface`) directly. `text`, `aria-label`: up to 120 characters. `size`: `1/3`, `1/2`, `1`, `2`, `3`. `loading`, `disabled`: JSON booleans. `native-type`: `button`, `submit`, `reset`.

### ui.alert
```markdown
:::ui.alert
{"type":"info","variant":"default","title":"The build checks parameters","supporting-text":"An unknown parameter stops the build.","closable":false,"aria-label":"The build checks parameters"}
:::
```
- `type`: `clear`, `info`, `warning`, `danger`. `success` is blocked by the default lock. `variant`: `default`, `flat`, `outlined`. `title`: up to 160 characters. `supporting-text`: up to 500. `closable`: `false` only under the default lock. `aria-label`: screen reader text.

### Framework errors

| Code | Cause |
| --- | --- |
| `SMART_REGISTRY_COMPONENT_NOT_FOUND` | No such `ui.*` component |
| `DOCUMENT_SMART_PROPS_JSON_INVALID` | The body is not valid JSON |
| `DOCUMENT_SMART_PROPS_INVALID` | The body is JSON but not an object |
| `FRAMEWORK_PROP_UNKNOWN` | No such parameter, including `id`, which Docara sets itself |
| `FRAMEWORK_PRESET_UNKNOWN` | No such `preset` |
| `FRAMEWORK_PROP_UNSUPPORTED_IN_BOUNDED_RUNTIME` | The lock blocks this value on a static site |

### Project components
A project Smart component uses the same form, `:::project.<name>`, with a JSON body checked against its manifest. See `components-smart-and-design.md`.

### `simai-framework:` ids
These are not page syntax. `simai-framework:component.buttons` names an entry in the generated Framework contract registry for the Documentation Source tooling: `docara inspect source <source>:<key>` and `docara documentation status --source=<source>`. Without that source in the site's documentation tracking, the command returns `DOCUMENTATION_SOURCE_UNKNOWN`. Pages call Framework components only through `:::ui.*`.
