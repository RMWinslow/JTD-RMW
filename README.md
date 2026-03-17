
This is an extensively modified version of Patrick Marsceill's [Just the Docs](https://github.com/just-the-docs/just-the-docs) Jekyll theme.

I've heavily modified it, both to make it more suited for my purposes,
and as a learning experience.

## Header Arguments:

title
: The page title. Used in the browser tab, sidebar navigation, breadcrumbs, search index, and Open Graph meta tags. Rendered as an `<h1>` in the `post` layout.

parent: Title of Parent
: Nest this page under the page with the matching title in the sidebar navigation and breadcrumbs.

grand_parent
: The title of the grandparent page, for three-level nesting in the sidebar and breadcrumbs.

has_children: true
: Marks this page as a parent. Renders an expandable nav group and an automatic list of child pages at the bottom.

nav_order
: Controls sort position in the sidebar. Numbers sort first (by value), then strings (alphabetically). Defaults to the page's `title`.

has_toc: false
: Parent pages usually list their children after the main content. This disables that automatically generated list of children.

date and last_modified_date or modified
: Self explanatory. Use YYYY-MM-DD format.
: `last_modified_date` is the JTD default name. I added a check for `modified` in the post layout as well since I keep forgetting the full parameter name.

layout: post
: Use the layout with my additions, namely: ToC, subtitle, date in header.

toc: true
: My addition which adds a collapsible table of contents to the page, using [jekyll-toc](https://github.com/toshimaru/jekyll-toc). Only works with `layout: post`.

subtitle
: Displayed beneath the title in the `post` layout header and in parent-child TOC listings.

description
: Adds a manual description to SEO headers and some of my navigation menus.

author
: Displayed in the `post` layout's date line (e.g., "by Robert Winslow").

imagepreview
: A full URL to an image for social media previews. Sets the `og:image` meta property.

math
: Which math renderer to load: `"katex"` (default), `"mathjax"`, or `"none"` to disable.

nav_exclude
: renders the page but hides it from the sidebar and parent TOCs

search_exclude
: makes the page invisible to the search widget

table_wrappers: to toggle two alternate ways of wrapping the tables.
- `table_wrappers: false` disables the table wrappers entirely. Bare tables.
- `table_wrappers: wide` makes the table wrapper span the entire width of the page, and doesn't widen the tables inside. May play oddly with asides.
- Otherwise, the tablewrappers work as default in JTD.
