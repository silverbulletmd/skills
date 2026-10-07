# Runtime lifecycle

Space Lua is SilverBullet's custom Lua implementation, broadly compatible with Lua 5.4, with SLIQ and SilverBullet APIs. Standard system Lua is not a substitute for testing those extensions.

Fenced `space-lua` blocks are indexed from any Markdown page, not just `Library/`. Each block has its own local scope; global assignments and named global functions are available across the space. Indexing/loading must complete before new definitions are callable. A `-- priority: number` directive loads higher-priority blocks earlier; use it only for actual dependencies.

```sh
sb --space notes eval '1 + 1' --json
sb --space notes script --file /absolute/path/check.lua --json
```

The script path is a local input file, not a path in the remote space. On POSIX shells a quoted heredoc also works:

```sh
sb --space notes script --json <<'LUA'
local pages = query[[from p = index.pages() limit 3 select p.name]]
return {pages=pages}
LUA
```

Keep shell interpolation out of Lua. Use a temporary input file on shells without quoted heredocs. Explicit JSON is useful for results, but a nil result currently emits no bytes; logs always use text. CLI execution may target a headless client, so do not equate its current page/selection with a human's visible editor.

## Reload after changes

Inspect `spacelua.describe("system.reboot")` on the running version. It saves the current editor buffer, detects disk changes, drains the index queue, and reloads configuration/scripts/styles/client state. Current versions first pull an external edit of the runtime's currently open page into that buffer (an unmodified buffer reloads it; unsaved edits are merged with it), so editing files on disk and then rebooting is safe. If the description instead warns that such an edit can be overwritten, the running version saves first: before editing that page externally, handle outstanding changes through the supported editor APIs or navigate the runtime away and let it save, then re-read the file; if disk content has already changed while a buffer is unsaved, preserve both versions before navigation or reboot can save over the disk version, then reconcile the requested changes. Do not discard an unsaved buffer to force progress.

When needed, execute once:

```sh
sb --space notes --timeout 120 eval 'system.reboot()'
sb --space notes logs -n 50
```

Then evaluate the expected new function or a narrow query. An empty result is normal for a no-value operation. If context teardown interrupts the response, probe with a harmless `sb --space notes eval '1' --json` using bounded retries, inspect logs, and verify behavior. Do not blindly repeat reboot or turn an arbitrary error into assumed success. If indexing/API readiness remains unresolved, report it. Script load errors can appear in logs even after reboot returns successfully.

## Check the effect

After editing Space Lua, styles or config, reboot as above, then check the effect instead of trusting the reboot: render the page and look at it, and find a new command's name in `system.listCommands()`:

```sh
sb --space notes eval 'editor.navigate("Page")'
sb --space notes eval 'editor.awaitRender()'
sb --space notes screenshot
sb --space notes eval 'system.listCommands()'
```

A page-template command registers only when its frontmatter `command:` is valid YAML; quote a value containing `: `.
