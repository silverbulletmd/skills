# Building interfaces

Widgets, dashboards, custom block rendering and styling. Check each API with `spacelua.describe` on the running version; this page covers the conventions the API listings don't.

## Pick the mechanism

* Plain Markdown and `${query[[...]]}` first: they keep links, hover previews and link decorations.
* Lists and tables of objects: a live widget (`widget.new { source = ..., presentation = { mode = "table" } }`). It refreshes itself, can filter, and makes `ref` columns clickable.
* Visuals: `widgets.chip`, `widgets.bars`, `widgets.stat` and `widgets.grid`, coloured with tones.
* Anything else: `widget.htmlBlock(dom.div { ... })` or `widget.html(...)`.
* Fenced data blocks: a fence whose language is `#tag` (for example ```` ```#contact ````) holds YAML; each document becomes a queryable object with that tag.
* Code widgets: `codeWidget.define { language = "#contact", render = function(body, pageName) ... end }` draws every fenced block of that language; `render` returns Markdown or a widget. With a data block of the same `#tag` language, each record stays queryable and renders as a card.
* Views (`view.define`) for panels outside the page.

## Lists and tables

```lua
board = board or {}

function board.tasks()
  return widget.new {
    title = "Tasks",
    refreshOn = { "index", "board:filter" },  -- re-run on index changes and on our event
    source = function()
      return query[[
        from t = index.tag "task"
        where not board.onlyBlocked or t.state == "blocked"
        order by t.page, t.pos
        select { text = t.name, state = t.state, ref = t.ref }
      ]]
    end,
    presentation = { mode = "table", columns = {
      { attribute = "text", label = "Task", type = "text" },
      { attribute = "ref", label = "Where", type = "ref" },
    } },
    filter = { inline = true },  -- a filter box in the title
  }
end

function board.toggle()
  board.onlyBlocked = not board.onlyBlocked
  event.dispatch("board:filter")
end
```

`${widgets.button("Only blocked", board.toggle)}` then `${board.tasks()}`. Segments and dropdowns only exist in panels; inline, use a button plus an event in `refreshOn`. Sort in the source: table headers don't sort.

## Visuals and tones

Tones are `success`, `warning`, `danger`, `info`, `neutral` and `accent`, with CSS variables `--tone-<name>` and `--tone-<name>-soft` defined for both themes. The helpers show text literally and accept lists or query results; `onClick` makes an element a button (for `bars`, it gets the clicked row):

```lua
app = app or {}
function app.progress(page)  -- done and total checklist items on a page
  local tasks = query[[from t = index.tag "task" where t.page == page]]
  local done = 0
  for _, t in ipairs(tasks) do if t.done then done = done + 1 end end
  return done, #tasks
end

widgets.chip("blocked", { tone = "danger",
  onClick = function() editor.navigate("Projects/Harbor") end })
widgets.grid(query[[from p = index.pages("project")]], function(p)
  local done, total = app.progress(p.name)
  return widgets.stat(p.name, done .. "/" .. total, { tone = "accent",
    sub = p.status, bar = total > 0 and done / total or 0,
    onClick = function() editor.navigate(p.name) end })
end, { min = "10rem" })
```

`widgets.bars(rows, { label = "name", value = fn, tone = fn, onClick = function(row) ... end })` draws one bar per row. Empty input renders "Nothing to show". For a colour no tone covers, pass `color = "#c0603a"` (any CSS colour) instead of `tone`; check it in both themes. In your own CSS, colour with `var(--tone-danger)` rather than fixed colours.

## Space Lua structure

* Each `space-lua` block is its own chunk: a `local` in one block is invisible to others, and blocks may load before the one that creates a shared table. Start every block that extends a namespace with `ns = ns or {}`, or put `-- priority: N` on the block that defines it.
* `query[[...]]` needs parentheses before indexing: `(query[[...]])[1]`.
* Query results are distinct: `select { status = t.status }` returns one row per status. Count with `group by` and `#group`, or over rows that keep `name` or `ref`.
* `CONFIG` loads first: `tag.define` (especially with `transform` or a schema) and configuration belong there; helpers can live on any page.
* `editor.getCurrentPage()` names the page being rendered.

## DOM builder

* `dom.tag { attr = "value", child, child }`: string keys are attributes, `on*` keys are event handlers. `class` also takes a list (`{ "card", urgent and "card-urgent" }`), `style` a table (`{ width = "40%" }`).
* String children render as Markdown and the whitespace between sibling children is dropped: write `"**12** checks"` as one string. `dom.text(s)` adds literal text; use it for names, titles and anything you didn't write.
* A list of children is added in order and `false` children are skipped: build rows in a loop, then `dom.ul { rows }`.
* Links: `dom.a { class = "wiki-link", ["data-ref"] = page, dom.text(name) }` looks and navigates like a SilverBullet link.

## Data in pages

* Keep records as pages with frontmatter or `#tag` data blocks, so the page stays readable as Markdown.
* Avoid tag names SilverBullet uses for its own objects: `page`, `task`, `item`, `paragraph`, `header`, `link`, `tag`, `data`, `anchor`, `table`.
* Quote YAML values that contain `: `, `#`, or start with `[`/`{`.

## Styling

Put CSS in a `space-style` block and style classes you attach:

* Tag-driven classes: `tag.define { name = "book", transform = function(o) o.pageDecoration = { cssClasses = { "book-page" } } return o end }` puts `book-page` on `<body>` and on links to those pages. Put it in `CONFIG`; elsewhere, pages indexed before it loads stay undecorated until `Space: Reindex`.
* Widget classes: give widget roots your own class and scope rules under it.
* Editor rules need SilverBullet's specificity: `body.book-page #sb-main .cm-editor { ... }`.

## Safety

Build markup with `dom.*` and `dom.text`, not by concatenating HTML strings (if you must, escape with `string.escapeHtml(s)`). To show user-, imported or agent-written Markdown, use `widget.markdown(text, { evaluate = false })`: `${...}`, code widgets and transclusions stay literal. Never `eval` page content you are only displaying.

## Check what you built

A reboot that returns cleanly does not mean the page works:

```sh
sb --space notes eval 'system.reboot()'
sb --space notes eval 'space.lint("Dashboard")'       # YAML, Lua and failing widgets; [] when clean
sb --space notes logs -n 50                           # load errors and "Lua widget error on …" lines
sb --space notes eval 'editor.navigate("Dashboard")'  # after a reboot, navigate again so widgets re-render
sb --space notes eval 'editor.awaitRender()'
sb --space notes screenshot --full-page dashboard.png
```

For dark mode, run `sb eval 'js.window.document.documentElement.setAttribute("data-theme", "dark")'`, screenshot, then set it back to `"light"`. `space.lint()` without a page checks every page; run it before reporting work as done.
