# CLAUDE.md — JTD-RMW

## Project Overview

This is an extensively modified version of the [Just the Docs](https://github.com/just-the-docs/just-the-docs) Jekyll theme, used as a custom GitHub Pages style. It is a remote theme consumed by multiple websites.

### Key Structure

- `_layouts/` — HTML layouts (default, post, home, remark_slides, table_wrappers, etc.)
- `_includes/` — Partials (nav, head, KaTeX, MathJax, TOC, etc.)
- `assets/css/` — Stylesheets: `compiled-jtd-style.css` (main JTD styles), `extrabits.css`, `kineticgraphs.css`, `colorscheme.css`
- `assets/js/` — JavaScript (search, etc.)
- `_config.yml` — Jekyll configuration (note: the `url` and `baseurl` here are from the original JTD and get overridden by consuming sites)

### Custom Features Beyond Vanilla JTD

- `layout: post` — adds TOC support, subtitle, and date in the header
- `toc: true` front matter — collapsible table of contents via jekyll-toc
- `table_wrappers` front matter — toggle between bare, wide, and default table wrapping
- KaTeX and MathJax includes for math rendering
- `kineticgraphs.css` — styling for [kgjs](https://github.com/cmakler/kgjs) interactive graphs
- Remark slides layout for presentations
- `parent_child_toc.html` — parent pages list their children automatically

## Writing Style Guidelines

- **Use natural language in all commit messages, PR descriptions, comments, and documentation.** Write in complete sentences. Do not drop articles ("a", "the", "an") or use telegraphic shorthand. For example, write "Add the missing KaTeX stylesheet to the post layout" rather than "add missing KaTeX stylesheet post layout."
- Keep commit messages concise but grammatically complete.

## Session Log

Keep this section updated with what was accomplished in each Claude session.

### 2026-03-16

- Created this CLAUDE.md file to establish project conventions and provide context for future sessions.
- Began indexing the repo structure and understanding the theme's custom features.

## TODOs

- [ ] Examine whether the `width: 0px` scrollbar trick (commented out in `kineticgraphs.css:14-17`) is used anywhere in the consuming sites, and whether it should be removed or restored.
- [ ] Index the websites that consume this theme as a remote theme. Candidate repos can be found at https://github.com/RMWinslow?tab=repositories — likely candidates include: posts, RMWinslow.github.io, bib, games, mynotes, cv, macronotes, 3102old, and others. Check each for `remote_theme` references to JTD-RMW in their `_config.yml`.

## Notes and Preferences

- Track all project context, session history, and preferences here in CLAUDE.md rather than in hidden memory files. This file should be the single source of truth for cross-session continuity.
