# CLAUDE.md — JTD-RMW

## Project Overview

This is an extensively modified version of Patrick Marsceill's [Just the Docs](https://github.com/just-the-docs/just-the-docs) Jekyll theme, used as a custom GitHub Pages remote theme. Consuming sites reference it via `remote_theme: RMWinslow/JTD-RMW` in their `_config.yml`.

## Repo Structure

### Layouts (`_layouts/`)

The layout inheritance chain is: `vendor/compress` → `table_wrappers` → `default` → content layouts.

- **`vendor/compress.html`** — The outermost wrapper. A third-party HTML compression layout that minifies the final output, controlled by the `compress_html` setting in the consuming site's `_config.yml`.
- **`table_wrappers.html`** — Wraps `<table>` elements in `<div>` wrappers for responsive scrolling. Behavior is controlled by the `table_wrappers` frontmatter variable (see below). This is the parent of `default.html`.
- **`default.html`** — The main structural layout. Renders the sidebar navigation, search bar, breadcrumbs, heading anchors, the parent-child TOC, and the page footer (including "last edited" timestamps and "edit on GitHub" links). Almost all other layouts inherit from this.
- **`post.html`** — Extends `default`. Adds a title/subtitle header group, a date line (posted and last modified dates, author), and an optional collapsible table of contents. This is the layout to use for most content pages.
- **`page.html`** — Extends `default`. A bare passthrough — just renders `{{ content }}` with no additional chrome. Identical to `home.html` and `about.html`.
- **`home.html`** — Extends `default`. Identical to `page.html`.
- **`about.html`** — Extends `default`. Identical to `page.html`.
- **`remark_slides.html`** — A standalone layout (does **not** inherit from `default`). Renders the page content as a [Remark.js](https://remarkjs.com/) slideshow with KaTeX math support. Includes its own inline CSS for slide styling. Note: this layout replaces `\\` with `\\\\\\\\` in the content to make LaTeX escapes survive the Remark markdown engine, which can break filenames containing underscores.
- **`null.html`** — Renders raw content with no wrapping at all. Intended for including pre-built HTML pages directly.

### Includes (`_includes/`)

- **`head.html`** — The `<head>` element. Loads the compiled JTD stylesheet, sets the page `<title>`, includes Google Analytics (if configured), loads the math renderer (KaTeX by default, MathJax if requested, or neither if disabled), loads the Lunr search script, and includes the SEO and custom head partials.
- **`head_seo.html`** — Outputs Open Graph meta tags (`og:title`, `og:description`, `og:image`, `og:url`, canonical URL, and author). Uses `page.description`, `page.imagepreview`, `site.title`, `site.author`, `site.url`, and `site.baseurl`.
- **`head_custom.html`** — Empty by default. Consuming sites can override this to inject custom `<head>` content (additional stylesheets, scripts, etc.).
- **`header_custom.html`** — Empty by default. Injected into the main content area header, after the search bar.
- **`nav_details.html`** — The primary sidebar navigation. Uses `<details>`/`<summary>` elements for collapsible sections. Supports up to four levels of nesting (top-level → children → grandchildren → great-grandchildren via `parent`, `grand_parent`, and `great_grand_parent`). Respects `nav_order`, `nav_exclude`, and `has_children`. Also renders `site.nav_external_links`.
- **`nav.html`** — An older navigation partial using JavaScript-driven expand/collapse (chevron icons) instead of `<details>` elements. Supports only three levels of nesting. **This file is not currently included anywhere** — `default.html` uses `nav_details.html` instead.
- **`parent_child_toc.html`** — Automatically generates a "Pages in this group" list at the bottom of any parent page. Shows each child's title, dates, subtitle, and description. Respects `nav_exclude`. Controlled by the `has_toc` frontmatter variable.
- **`title.html`** — Renders either the site logo (if `site.logo` is set) or the site title text.
- **`toc.html`** — The [jekyll-toc](https://github.com/allejo/jekyll-toc) include (v1.2.0). Generates a table of contents from heading elements in the rendered HTML. Used by `post.html` when `toc: true` is set.
- **`katex.html`** — Loads KaTeX v0.16.22 from the jsDelivr CDN: the stylesheet, the main script (deferred), auto-render extension, and copy-tex extension. Configures delimiters (`$$...$$` for display, `$...$` for inline, plus `\(...\)` and `\[...\]`), enables `globalGroup` (for `\gdef`), and sets `throwOnError: false` to prevent one bad equation from breaking the entire page. Includes a small style block to make display math horizontally scrollable with a max width of 800px.
- **`mathjax.html`** — Loads MathJax v4.0.0-beta.4 from jsDelivr. Configures `$...$` for inline math. Note: this is a beta version of MathJax 4.
- **`fix_linenos.html`** — A workaround for a Jekyll/Rouge bug where highlighted code blocks with line numbers produce invalid nested `<pre>` tags. Used via `{% include fix_linenos.html code=some_var %}`.
- **`footer_custom.html`** — Renders `site.footer_content` if present.
- **`nav_header_custom.html`** — Empty. Can be overridden by consuming sites to add content above the navigation.
- **`nav_footer_custom.html`** — Empty. Can be overridden by consuming sites to add content below the navigation.

### CSS Includes (`_includes/css/`)

- **`just-the-docs.scss.liquid`** — The SCSS entry point. Imports support modules, the selected color scheme, all JTD modules, and the custom SCSS override file. This is how the theme's Sass gets compiled.
- **`custom.scss.liquid`** — Imports `custom/custom`, providing a hook for consuming sites to add their own Sass.

### Assets (`assets/`)

- **`css/compiled-jtd-style.css`** — The pre-compiled main stylesheet for JTD. Contains all the core layout, typography, navigation, and search styles. This is a compiled artifact, not a source file to edit directly.
- **`css/colorscheme.css`** — Defines CSS custom properties for the color scheme (solarized-inspired light theme with a dark mode `prefers-color-scheme` media query). Variables include `--basecolor`, `--boxcolor`, `--feedbackcolor`, `--accentcolor`, `--bordercolor`, `--textcolor`, and `--titlecolor`.
- **`css/extrabits.css`** — Empty placeholder for consuming sites to add custom CSS.
- **`css/kineticgraphs.css`** — Styles for [kgjs](https://github.com/cmakler/kgjs) interactive graph containers: text selection prevention on SVG text, KaTeX font sizing, game theory matrix tables, slider/checkbox styling. Contains a commented-out scrollbar-hiding trick (`width: 0px` on `::-webkit-scrollbar`).
- **`js/just-the-docs.js`** — The main JavaScript file handling search (Lunr integration), navigation toggle, and scroll behavior.
- **`js/zzzz-search-data.json`** — A Liquid template that generates the search index JSON. Iterates over all site pages (respecting `search_exclude`), splits content by headings, and creates searchable entries with titles, content snippets, and URLs. The `zzzz-` prefix ensures it sorts last in directory listings.
- **`js/vendor/lunr.min.js`** — The Lunr.js search library.

### Other Files

- **`_config.yml`** — The theme's default Jekyll configuration. The `url` and `baseurl` values here are vestiges of the original JTD and get overridden by each consuming site. Contains defaults for search, navigation sorting, footer, "edit on GitHub" links, Google Analytics, and kramdown settings.
- **`Gemfile`** — Points to the `just-the-docs` gemspec.
- **`just-the-docs.gemspec`** — The gem specification (v0.3.3). Lists Jekyll ≥ 3.8.5 as a runtime dependency.
- **`lib/tasks/search.rake`** — A Rake task (likely for building the search index).

## Page Frontmatter Reference

These are all the YAML frontmatter variables that JTD-RMW recognizes on individual pages. They can be set in the frontmatter block at the top of any page or post.

### Layout Selection

**`layout`** — Which layout template to use for this page.
- `post` — The recommended layout for most content pages. Displays the title, subtitle, dates, author, and an optional table of contents above the content.
- `default` — The base layout with sidebar navigation, search, breadcrumbs, and footer. No title header or date line — just the raw content.
- `page` / `home` / `about` — Identical passthrough layouts that extend `default`. They exist mainly for semantic naming and backward compatibility.
- `remark_slides` — Renders the page as a Remark.js slideshow instead of a normal page. Does not include the sidebar or any JTD chrome.
- `null` — Outputs raw content with absolutely no wrapping HTML. Useful for embedding pre-built HTML files.

### Title and Metadata

**`title`** — The page title. Used in the browser tab (`<title>` tag), the sidebar navigation, breadcrumbs, the parent-child TOC, the search index, and Open Graph meta tags. If `layout: post` is used, it is also rendered as an `<h1>` heading at the top of the page.

**`subtitle`** — A subtitle displayed beneath the title in the `post` layout's `<hgroup>` header. Also shown in the parent-child TOC listing for this page.

**`description`** — A short description of the page. Used in the HTML `<meta name="description">` tag and the Open Graph `og:description` property for SEO and social sharing. Also displayed in the parent-child TOC listing beneath the subtitle.

**`author`** — The author of the page. Displayed in the date header line of the `post` layout (e.g., "by Robert Winslow").

**`imagepreview`** — A URL to an image for social media previews. Sets the `og:image` Open Graph meta property. Should be a full URL.

### Dates

**`date`** — The original publication date of the page. Displayed in the `post` layout's date header and in the parent-child TOC. Use `YYYY-MM-DD` format.

**`last_modified_date`** — The date the page was last modified. Displayed in the `post` layout's date header, in the parent-child TOC, and in the footer's "Page last modified" timestamp (the footer uses the `site.last_edit_time_format` Ruby time format string). Use `YYYY-MM-DD` format.

**`modified`** — An alias for `last_modified_date`, added as a convenience because the full parameter name is easy to forget. The `post` layout checks for `last_modified_date` first and falls back to `modified`. Note: the footer in `default.html` only checks `page.last_modified_date`, not `modified`, so using `modified` alone will show the date in the post header but not in the footer.

### Navigation

**`parent`** — The title of the parent page in the navigation hierarchy. Setting this nests the page under the parent in the sidebar and generates a breadcrumb trail. The value must exactly match the `title` of the intended parent page.

**`grand_parent`** — The title of the grandparent page. Used for three-level nesting in the sidebar navigation and breadcrumbs. Must match the `title` of the grandparent page.

**`great_grand_parent`** — The title of the great-grandparent page. Only used in `nav_details.html` to determine whether to expand the navigation tree to show the current page. Does not appear in breadcrumbs (which only go two levels deep).

**`has_children`** — Set to `true` to indicate that this page has child pages. This controls whether the navigation renders an expandable `<details>` group for this page's children and whether the parent-child TOC appears at the bottom of the page.

**`has_toc`** — Set to `false` to suppress the automatically generated list of child pages at the bottom of a parent page. Only relevant when `has_children` is `true`. Defaults to `true`.

**`nav_order`** — Controls the sort position of this page in the sidebar navigation. Can be a number or a string. Numeric values are sorted first (by value), followed by string values (sorted alphabetically). If omitted, the page's `title` is used as the sort key.

**`nav_exclude`** — Set to `true` to hide this page from the sidebar navigation and from parent-child TOC listings. The page is still rendered and accessible by direct URL; it is just invisible in the navigation menus.

### Search

**`search_exclude`** — Set to `true` to exclude this page from the Lunr search index. The page will not appear in search results, though it remains accessible by URL and visible in the navigation (unless also `nav_exclude`d).

### Math Rendering

**`math`** — Controls which math rendering engine is loaded for this page. The resolution order is: page frontmatter → layout frontmatter → `site.math` in `_config.yml` → `"katex"` (the hardcoded default).
- `"katex"` (default) — Loads KaTeX with auto-render. Supports `$...$` (inline), `$$...$$` (display), `\(...\)` (inline), and `\[...\]` (display).
- `"mathjax"` — Loads MathJax 4 beta. Supports `$...$` (inline) and `\(...\)` (inline).
- `"none"` (or any other unrecognized value) — Disables math rendering entirely for the page. Useful for pages with content that conflicts with `$` delimiters.

### Table Wrapping

**`table_wrappers`** — Controls how `<table>` elements on the page are wrapped for responsive display. Processed by `table_wrappers.html`.
- *(default / omitted)* — Tables are wrapped in `<div class="table-wrapper">`, which provides horizontal scrolling on narrow screens and limits table width.
- `"wide"` — Tables are wrapped in `<div class="table-wrapper-wide">`, which spans the full page width without constraining the table.
- `false` — Tables are not wrapped at all. Bare `<table>` elements.

### Table of Contents (Post Layout)

**`toc`** — Set to `true` to display a collapsible table of contents at the top of the page in the `post` layout. The TOC is generated from heading elements in the rendered HTML using the `toc.html` include (jekyll-toc). Only works with `layout: post`.

## Site Configuration Reference (`_config.yml`)

These are the `_config.yml` settings that JTD-RMW's templates read. Consuming sites set these in their own `_config.yml`.

| Setting | Description |
|---------|-------------|
| `title` | The site title. Displayed in the sidebar header and browser tab. |
| `description` | Site description (not currently used in templates, but available to plugins). |
| `author` | Site author. Used in the `<meta property="author">` SEO tag. |
| `url` | The base URL of the site (e.g., `https://www.rmwinslow.com`). Used for canonical URLs and Open Graph. |
| `baseurl` | The subpath of the site (e.g., `/blog`). Used in URL construction. |
| `logo` | Path to a logo image. If set, the sidebar header displays this image instead of the site title text. |
| `search_enabled` | Set to `true` to enable the Lunr search bar. Defaults to `true`. |
| `search.heading_level` | The heading depth to split pages into searchable sections (1–6, default 2). |
| `search.previews` | Number of preview snippets per search result (default 3). |
| `search.preview_words_before` | Words to show before a search match in the preview (default 5). |
| `search.preview_words_after` | Words to show after a search match in the preview (default 10). |
| `search.tokenizer_separator` | Regex for splitting search tokens (default `/[\s\-/]+/`). |
| `search.rel_url` | Whether to display relative URLs in search results (default `true`). |
| `search.button` | If `true`, shows a floating search button in the bottom-right corner. |
| `heading_anchors` | If `true` (default), headings get clickable `¶` anchor links. |
| `nav_sort` | `"case_sensitive"` (default) or `"case_insensitive"` for sidebar nav sorting. |
| `nav_external_links` | A list of `{title, url}` objects for external links in the sidebar. |
| `back_to_top` | If `true`, shows a "Back to top" link in the page footer. |
| `back_to_top_text` | The text for the back-to-top link (default `"Back to top"`). |
| `footer_content` | HTML string displayed at the bottom of every page. |
| `last_edit_timestamp` | If `true`, shows "Page last modified: ..." in the footer. |
| `last_edit_time_format` | Ruby time format string for the last-modified timestamp. |
| `gh_edit_link` | If `true`, shows an "Edit this page on GitHub" link in the footer. |
| `gh_edit_link_text` | The text for the edit link. |
| `gh_edit_repository` | The GitHub repository URL for edit links. |
| `gh_edit_branch` | The branch name for edit links. |
| `gh_edit_source` | An optional subdirectory for the source files. |
| `gh_edit_view_mode` | `"tree"` or `"edit"` — whether the GitHub link opens the file viewer or the editor. |
| `color_scheme` | The color scheme name. Used by the SCSS pipeline to select a color scheme file. |
| `ga_tracking` | Google Analytics tracking ID (e.g., `UA-XXXXXXX-XX`). If set, includes the GA script. |
| `ga_tracking_anonymize_ip` | If `true`, enables GDPR-compliant IP anonymization in GA. |
| `math` | The default math renderer for the site: `"katex"`, `"mathjax"`, or `"none"`. Can be overridden per-page. |
| `webfontdirectory` | A URL to a directory of web fonts (used by some consuming sites, not directly by the theme templates). |
| `compress_html.ignore.envs` | Controls the HTML compression layout. Set to `all` to disable compression. |

## Potential Issues

These are things noticed during the code review. Tagged to distinguish them from user-created TODOs.

- **[claude's suggestion]** `nav.html` is never included anywhere — `default.html` uses `nav_details.html` instead. The file could be removed, or kept as an alternate navigation style that consuming sites could swap in via an override.
- **[claude's suggestion]** The `modified` alias only works in the `post` layout's date header. The footer timestamp in `default.html` (line 136) only checks `page.last_modified_date`, so a page using `modified:` alone will not show a "Page last modified" footer. Consider adding a fallback in the footer as well, or documenting this caveat prominently.
- **[claude's suggestion]** The `remark_slides.html` layout loads KaTeX v0.16.7, while `katex.html` uses the newer v0.16.22. These should probably be kept in sync.
- **[claude's suggestion]** The `mathjax.html` include loads MathJax v4.0.0-beta.4, which is a beta release. MathJax 4 has since been released. Consider updating to a stable version.
- **[claude's suggestion]** `_config.yml` still has the original JTD author's `url`, `baseurl`, `ga_tracking`, `footer_content`, and `gh_edit_repository` values. While consuming sites override these, having stale defaults could cause confusion if someone builds the theme repo directly (e.g., for testing).
- **[claude's suggestion]** The `great_grand_parent` frontmatter variable is checked in `nav_details.html` (line 63) to auto-expand the sidebar, but it is not used in the breadcrumb logic in `default.html`, which only handles two levels (`parent` and `grand_parent`). If a four-level hierarchy is intentional, breadcrumbs could be extended to match.
- **[claude's suggestion]** `head.html` sets the `<title>` to `{{ page.title }} - {{ site.title }}`. If a page has no `title`, this produces ` - Site Title` with a leading space and dash. A conditional fallback might be worth adding.

## Writing Style Guidelines

- **Use natural language in all commit messages, PR descriptions, comments, and documentation.** Write in complete sentences. Do not drop articles ("a", "the", "an") or use telegraphic shorthand. For example, write "Add the missing KaTeX stylesheet to the post layout" rather than "add missing KaTeX stylesheet post layout."
- Keep commit messages concise but grammatically complete.

## Session Log

Keep this section updated with what was accomplished in each Claude session.

### 2026-03-16

- Created this CLAUDE.md file to establish project conventions and provide context for future sessions.
- Began indexing the repo structure and understanding the theme's custom features.
- Indexed all public repos at https://github.com/RMWinslow for GitHub Pages usage and theme references.
- Thoroughly documented the entire repo: every layout, include, CSS file, JS file, and asset, with natural language descriptions.
- Documented all page frontmatter variables and site configuration settings with full descriptions and behavior details.
- Identified several potential issues (KaTeX version mismatch between layouts, unused `nav.html`, `modified` alias inconsistency, stale `_config.yml` defaults, etc.).

## TODOs

- [ ] Examine whether the `width: 0px` scrollbar trick (commented out in `kineticgraphs.css:14-17`) is used anywhere in the consuming sites, and whether it should be removed or restored.
- [x] Index the websites that consume this theme as a remote theme. (Done — see the inventory below.)
- [ ] Clean up or archive obsolete repos (e.g. `jtd-test-style`, `latexpagetest`, `just-the-docs-tweaked`, and any other repos that are no longer actively used).
- [ ] Index the differences between JTD-RMW and `just-the-docs-tweaked` to understand what would be involved in migrating the sites that still use the older fork (`3102old`, `tones`).
- [ ] Add `url: "https://www.rmwinslow.com"` to the **bib** site's `_config.yml`. Without it, the canonical URL and `og:url` meta tags use `http://` instead of `https://`, which is bad for SEO.
- [ ] Add `author: Robert Winslow` to the **bib** site's `_config.yml`. Currently the `<meta property="author">` tag renders empty.
- [ ] Audit the other consuming sites (`games`, `RMWinslow.github.io`) for the same `url`/`author` gaps.

## Consuming Sites Inventory

Indexed 2026-03-16 from https://github.com/RMWinslow?tab=repositories.

### Sites using `RMWinslow/JTD-RMW`

| Repo | Title | Notes |
|------|-------|-------|
| **posts** | RMW's Blogalike | Main blog. Has `CLAUDE.md` excluded from build. |
| **RMWinslow.github.io** | Robert Winslow | Root personal site. Uses MathJax. |
| **bib** | Bibliography | Minimal config. |
| **games** | Game Rules | Includes webfont directory reference. |
| **circe** | La Circe | Uses `_config.yaml` (not `.yml`). Missing `url` and `author`. Pulled locally. |
| **jtd-test-style** | JTD test | Test/sandbox site for the theme — candidate for archival. |

### Sites using a different theme

| Repo | Title | Theme | Notes |
|------|-------|-------|-------|
| **macronotes** | Macro Notes | `pmarsceill/just-the-docs` (original unmodified JTD) | Solarized color scheme. Could potentially migrate to JTD-RMW. |
| **3102old** | 3102 Instructor Notes | `RMWinslow/just-the-docs-tweaked` | Older fork, not JTD-RMW. |
| **tones** | Pure Tones | `RMWinslow/just-the-docs-tweaked` | Older fork, not JTD-RMW. Search disabled. |
| **mynotes** | Robbie's Notes | `minimal-mistakes` | Completely different theme. UMN email, kramdown, pagination. |
| **cv** | (CV) | `davewhipp` style | Simple markdown-to-CV theme. |
| **latexpagetest** | Testing a Theme | `Hammie217/LatexJekyll` | Had JTD-RMW commented out — candidate for archival. |
| **lists** | Lists of Links | No remote theme (default GitHub Pages) | Just markdown files of links. |
| **retlab** | — | Ben Balter config | Appears to be a fork, not an original RMW site. |

### Repos with no GitHub Pages config found

ytrss, scrapetest, webfonts, umn-git-thesis-demo, kgjs, emojitwo, twemoji-colr-tweaked, doodads, code, pui-data-repository, ModernSlides

(Checked for both `_config.yml` and `_config.yaml` on both `main` and `master` branches. None found.)

## Audit Ideas [claude's suggestions]

These are potential audits that could be run across the theme and consuming sites to improve consistency, cleanliness, and maintainability.

### Frontmatter hygiene (per-page, across all sites)

- **Missing titles.** Pages without a `title` produce a broken `<title>` tag (` - Site Title`) and are invisible to navigation and search. Scan for any untitled pages that aren't intentionally using `layout: null`.
- **Orphaned `parent` references.** Pages whose `parent` value doesn't match any existing page's `title` will silently fail to nest in the sidebar. This is easy to break when renaming a parent page.
- **Phantom `has_children`.** Pages marked `has_children: true` that don't actually have any children referencing them (empty nav groups), or the reverse — pages that have children pointing at them but aren't marked `has_children: true` (children won't render as a collapsible group).
- **`modified` vs `last_modified_date` usage.** Pages using the `modified` shorthand won't get the footer timestamp. Scan for pages using `modified` and flag them, since `last_modified_date` is the more complete option.
- **Stale dates.** Pages whose `last_modified_date` is older than their most recent git commit — the date may have been forgotten after an edit.
- **Missing `description`.** Pages that are likely to be shared (top-level, high-traffic) but have no `description` for social previews and SEO.
- **Unnecessary math loading.** Pages that don't contain any math notation but still load KaTeX (the default). Adding `math: none` to pages or sections that don't need it could reduce page weight, especially on sites like `games` where math is probably never used.
- **Double KaTeX conflict.** Some pages load interactive graphs via a third-party library (kgjs) that bundles its own KaTeX. When the theme also loads KaTeX, the page stutters and locks up because two instances of KaTeX are fighting over the same DOM. This needs to be fixed — likely by setting `math: none` on those pages, or by making the theme's KaTeX loading smarter about detecting an existing instance.

### Navigation structure

- **Depth consistency.** Check whether any site uses more than three levels of nesting, and whether the breadcrumb limitation (only two levels) causes confusion for deeply nested pages.
- **Single-child parents.** Parent pages with only one child — the grouping may not be adding value and could be flattened.
- **Nav ordering gaps.** Pages using `nav_order` with large gaps or inconsistent numbering that makes future insertions awkward.

### Cross-site consistency

- **Plugin standardization.** The sites use different plugin sets (some have `jekyll-sitemap`, some don't; some have `jekyll-redirect-from`, some don't). Consider whether all sites should have a standard plugin baseline.
- **`page_excerpts: true`** is set on some sites but the theme never reads it. Either remove it everywhere or add support for it in the theme.
- **`webfontdirectory`** is set on `posts` and `games` but not on `bib`, `circe`, or `RMWinslow.github.io`. If some pages on those sites reference web fonts, they may be loading from the wrong path.
- **`gh_edit_link: false`** is set inconsistently — present on most sites but missing from `bib`. Harmless in practice (the footer guards against incomplete config) but inconsistent.
- **`last_edit_timestamp: true`** is only set on `posts`. The other sites don't show "Page last modified" in the footer even when pages have `last_modified_date` set.

### Link and content integrity

- **Internal broken links.** Links between pages within a site that point to pages that have been moved, renamed, or deleted.
- **External link rot.** Links to external resources that have gone stale (404s, domain changes). Particularly relevant for `bib` which is full of paper references.
- **Missing or broken images.** Image references that don't resolve, especially in older posts.
- **Redirect chains.** Sites using `jekyll-redirect-from` may have accumulated stale redirects that could be cleaned up.

### Theme-level cleanup

- **Remove or repurpose `nav.html`.** It's unused — `default.html` uses `nav_details.html`. Either delete it or document it as an alternative that consuming sites can swap in.
- **Consolidate identical layouts.** `page.html`, `home.html`, and `about.html` are byte-for-byte identical. Consider whether they should all just be symlinks or whether the semantic distinction is worth maintaining.
- **Sync KaTeX versions.** `katex.html` uses v0.16.22 but `remark_slides.html` has v0.16.7 hardcoded inline.
- **Update MathJax.** `mathjax.html` loads a beta (v4.0.0-beta.4). MathJax 4 has since had stable releases.
- **Extend `modified` fallback to the footer.** The `post` layout handles `modified` as an alias for `last_modified_date`, but the footer in `default.html` doesn't. A one-line Liquid `assign` would fix this.
- **Guard the `<title>` tag.** Add a conditional so pages without a `title` get just the site title instead of ` - Site Title`.
- **Clean up `_config.yml` defaults.** The theme's own config still has the original JTD author's URLs, GA tracking ID, and footer content. These are overridden by consuming sites but are misleading if someone builds the theme repo directly for testing.

### Accessibility

- **Color contrast.** The `colorscheme.css` dark mode uses `--textcolor: #fee` on `--basecolor: #000` — likely fine, but the light mode's `--textcolor: #55525B` on `--basecolor: #fdf6e3` should be verified against WCAG AA standards.
- **Alt text in templates.** The SVG icons in `default.html` have `<title>` elements (good), but any images injected via the logo or content should be checked.
- **Keyboard navigation.** The `<details>`-based nav in `nav_details.html` should be keyboard-accessible by default, but worth verifying with a screen reader.

### Performance

- **KaTeX on every page.** The default math engine is KaTeX, which loads ~300KB of CSS + JS. On sites where most pages don't use math, a site-level `math: none` default with per-page opt-in (`math: katex`) would be lighter.
- **Search index size.** On large sites like `posts`, the Lunr search index (`search-data.json`) can get very large. Pages that don't need to be searchable could use `search_exclude: true` to trim it.
- **Unused CSS.** `extrabits.css` is loaded but empty. `kineticgraphs.css` is loaded on every site but only relevant to sites using kgjs. Consider making these opt-in via config rather than always-on.

## Notes and Preferences

- Track all project context, session history, and preferences here in CLAUDE.md rather than in hidden memory files. This file should be the single source of truth for cross-session continuity.
