# Building interfaces

Widgets, dashboards, custom block rendering and styling. Check each API with `spacelua.describe` on the running version; this page covers the conventions the API listings don't.

## Pick the mechanism

* `${expression}` in a page: returns Markdown text (links, tables, lists) or a widget. Use plain Markdown first; it keeps links, hover previews and link decorations.
* `widget.htmlBlock(dom.div { ... })` or `widget.html(...)`: interactive or laid-out output (grids, buttons, bars). `widget.new { markdown = ..., display = "block", cssClasses = {...} }` renders Markdown inside a styled container.
* Fenced data blocks: a fence whose language is `#tag` (for example ```` ```#contact ````) holds YAML; each document becomes a queryable object with that tag (`index.tag "contact"`).
* Code widgets: `codeWidget.define { language = "#contact", render = function(body, pageName) ... end }` draws every fenced block of that language; `render` returns Markdown or a widget. Combined with a data block of the same `#tag` language, each record stays a queryable object and renders as a card, so a page can hold structured records that read well.
* Views (`view.define`, see `spacelua.describe`) for panels and lists outside the page.

## Space Lua structure

* Each `space-lua` block is its own chunk: a `local` in one block is invisible to others, and blocks may load before the one that creates a shared table. Start every block that extends a namespace with `ns = ns or {}` then `local N = ns`, or put `-- priority: N` on the block that defines it.
* `query[[...]]` needs parentheses before indexing: `(query[[...]])[1]`.
* `CONFIG` loads first: tag definitions (`tag.define`, especially with `transform` or a schema) and configuration belong there; widget helpers and functions can live on any page.
* `editor.getCurrentPage()` names the page being rendered; use it to make one function serve every page of a kind.

## DOM builder

* `dom.tag { attr = "value", child, child }`: string keys are attributes, `on*` keys are event handlers (`onclick = function() editor.navigate("Page") end`).
* String children render as Markdown and the whitespace between sibling children is dropped: write `"**12** checks"` as one string rather than `dom.b{"12"}, " checks"`.
* For literal text that must not be interpreted (code, logs), build an HTML string with escaped `& < > "` and pass it to `widget.htmlBlock(html)`.

## Data in pages

* Keep records as pages with frontmatter, or as `#tag` data blocks; query them with SLIQ and render with widgets, so the page stays readable as Markdown.
* Avoid tag names SilverBullet already uses for its own objects: `page`, `task`, `item`, `paragraph`, `header`, `link`, `tag`, `data`, `anchor`, `table`. A page tagged `task` mixes with checkbox tasks in `index.tag "task"`.
* Quote YAML values that contain `: `, `#`, or start with `[`/`{`.

## Styling

Put CSS in a `space-style` block. Style through classes you attach, not SilverBullet's internal markup:

* Tag-driven classes: `tag.define { name = "book", transform = function(o) o.pageDecoration = { icon = "book", cssClasses = { "book-page" } } return o end }` puts `book-page` on `<body>` for those pages and on every link to them. Put every `tag.define` with a `transform` in `CONFIG`: it loads before pages are indexed. Elsewhere, pages indexed before that block loads keep no decoration until the space is reindexed (`Space: Reindex`).
* Widget classes: give widget roots your own class (`class = "my-board"`) and scope all rules under it.
* Colours: define your own variables on `html` and redefine them under `html[data-theme="dark"]`, so both themes work.
* Editor rules need SilverBullet's specificity: page-level overrides go under `#sb-main .cm-editor` (for example `body.book-page #sb-main .cm-editor { ... }`).

## Safety

Markdown widgets evaluate `${...}` in whatever text they render. When a widget shows user, imported or agent-written text as Markdown, break the sequence first (for example replace `${` with `$\u{200B}{`) or render it as escaped HTML. Never `eval` page content you are only displaying.

## Check what you built

A reboot that returns cleanly does not mean the page works. After changing Space Lua, styles or pages with widgets:

```sh
sb --space notes eval 'system.reboot()'
sb --space notes eval 'space.lint("Dashboard")'       # YAML, Lua and failing widgets on that page; [] when clean
sb --space notes logs -n 50                           # script load errors and "Lua widget error on …" lines
sb --space notes eval 'editor.navigate("Dashboard")'
sb --space notes eval 'editor.awaitRender()'
sb --space notes screenshot --full-page dashboard.png # the whole page, not just the first screen
```

If colours matter, look at both themes: `sb eval 'editor.invokeCommand("Editor: Toggle Dark Mode")'`, screenshot again, then toggle back. `space.lint()` without a page checks every page; run it before reporting work as done.
