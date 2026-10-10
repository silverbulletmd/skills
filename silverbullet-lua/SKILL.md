---
name: silverbullet-lua
description: Use when writing, inspecting, or debugging SilverBullet Space Lua functions, commands, live expressions, configuration, or views, or when discovering their APIs.
---

# Space Lua

When a task needs `sb`, check availability with `command -v sb`. If it is missing, install the edge CLI:

```sh
curl -fsSL https://silverbullet.md/install-sb.sh | sh -s -- --edge
```

Follow any PATH instructions printed by the installer, then verify with `sb --help` before continuing. Local file-only work does not require installation.

Before calling a local desktop runtime, determine whether the command environment can reach the host's loopback server without invoking `sb` as a probe. A blocked sandbox or VM can make `sb` repeatedly launch the app even when it is already open. In Codex, request outside-sandbox execution for each command that contacts the local space (`exec_command` with `sandbox_permissions: "require_escalated"`) or use an explicitly configured loopback permission. `sb --help` and `sb space ls` do not contact that server. In Cowork, use an available host-side connector; `127.0.0.1` inside its VM refers to the VM. If neither is available, keep local file work separate and report that live evaluation is unavailable. Do not run or retry runtime commands in the blocked environment.

Select the target with `sb space ls` and explicit `--space` (or a user-provided `--url`). Local file editing can work offline; Lua execution needs the selected space's runtime. Read the space's operating manual and the shared [silverbullet-markdown skill](../silverbullet-markdown/SKILL.md) when editing pages. Inspect `sb --help` before using unfamiliar commands.

## Discover before implementing

Use the live API documentation rather than guessing names or signatures. The "Explore your space" section of `sb --help` lists the discovery calls, including `system.listCommands()` for registered commands:

```sh
sb --space notes eval 'spacelua.listFunctions("editor")' --json
sb --space notes eval 'spacelua.describe("editor.getText")' --json
sb --space notes eval 'spacelua.renderApiDocumentation("editor")' --text
```

`describe` accepts a function or dotted function name; namespace discovery uses `listFunctions` or `renderApiDocumentation`. Inspect parameters, returns, examples, and `see` links. Runtime capabilities vary by version and installed libraries. If documentation is insufficient, follow [API and source discovery](references/discovery.md), including a last-resort clone of SilverBullet.md.

## Author and verify

Read [runtime and reload](references/runtime.md) before installing a `space-lua` block. For widgets, dashboards, custom block rendering, data blocks and styling, read [building interfaces](references/ui.md). Use a plain `lua` fence for an inert example. Prefer a small named namespace; preserve the space's existing design and APIs.

Use local file tools for an accessible folder, or `sb fs read/edit/write` for a remote-only space, retaining the inspected revision for conditional writes. `sb eval` evaluates one expression; `sb script --file script.lua` or stdin runs multiple statements. End scripts with `return` to obtain a result. Print output goes to runtime logs, not the returned value. Results are plain JSON: nil becomes `null`, a query collection such as `index.pages("book")` becomes an array of its rows (at most 1000, then a `"<truncated: …>"` entry), and functions appear as markers like `"<function>"`.

Verify new logic with narrow calls, inspect `sb --space notes logs -n 50`, and confirm saved content and expected results. Reboot saves any open unsaved buffer; check `spacelua.describe("system.reboot")` to see whether the running version merges an external edit of the open page first (see [runtime and reload](references/runtime.md)). A successful process, empty stdout, or a ready runtime does not establish that the user's script loaded correctly. Report syntax/runtime failures precisely and repair only the requested code.
