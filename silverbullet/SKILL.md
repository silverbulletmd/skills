---
name: silverbullet
description: Use when working with a SilverBullet space, whether a local folder or a remote server: reading, searching, editing, querying, scripting or maintaining its pages. Installs and connects the sb CLI, then points to sb help for topic guidance.
---

# SilverBullet spaces

A SilverBullet space is a set of Markdown pages in a local folder or on a server. This skill gets you connected and safe; the space and `sb` teach the rest.

## Install and verify `sb`

Local file-only work does not need `sb`. When you need it, check `command -v sb`. If it is missing, install the edge CLI and follow any PATH instructions it prints:

```sh
curl -fsSL https://silverbullet.md/install-sb.sh | sh -s -- --edge
```

Verify with `sb --help`.

## Local files or remote

Choose by whether the space's folder is readable from your environment, not by hostname. A localhost URL does not imply accessible files.

* **Folder readable:** use ordinary file tools for reading and editing Markdown. Anchor paths to the confirmed space root, not a surrounding repository. Use `sb` only for what files cannot give you: indexed queries, Space Lua, and `sb fs read --rendered` (a page with its queries and templates evaluated).
* **Folder not readable:** use `sb fs` (`ls`, `read`, `edit`, `write`, `rm`). Paths are space-relative filenames including `.md`; wiki links omit it.

Run `sb space ls` (use `--json`) and pass an explicit `--space <name>` that matches the folder or server. Do not rely on the sole-space fallback; if more than one space could match, ask. To start a new local space use `sb space create <folder>`. To move one, run `sb space mv <name> <folder> --dry-run` first. `sb open <path>` opens a folder or file in Desktop only; register or open a folder only when the request calls for it, not as a prerequisite to editing text. For an exact URL the user gave you, use `--url`. If sign-in is needed, run `sb space login <name>` and let the user finish it in the browser.

## Reaching a local space from a sandbox

Commands that contact the space (`sb help`, `query`, `lua`, `eval`, `script`, `fs`) use HTTP, often on loopback. A sandbox or VM that cannot reach the host's loopback can make `sb` launch a second desktop process. `sb --help` and `sb space ls` never contact the space.

* **Codex:** request outside-sandbox execution for each command that contacts a local space (`sandbox_permissions: "require_escalated"`), or use a permission profile that allows `127.0.0.1`.
* **Cowork:** use a host-side connector. `127.0.0.1` inside its VM is the VM.
* **Otherwise:** if the space is unreachable, say so and use local files where possible. Do not retry blocked commands or treat the space as empty.

## Safety

* Keep secrets out of commands, examples and output. Prefer saved authentication.
* Read before you edit, and read back after you write.
* A lost response to a write is an uncertain outcome. Read the current state before retrying.
* Writes are conditional: `sb fs write` takes `--create` for a new page, or `--revision` (alias `--if-match`) with the revision you read. After a conflict, re-read and reassess. Never use `--overwrite` to get past one.
* Delete only within the requested scope.
* `sb lua`, `sb eval` and `sb script` run code in the space and can change it. Query first when you can.

## Then run `sb help`

Before querying, scripting or writing pages, run `sb help`, and `sb help --space <name>` once you know the space. It lists the space's skills and an outline of its pages. Load the skills it lists with `sb help <topic>` before you start. They are version-matched to the space and cover querying, Space Lua, Markdown conventions, mentions and more. If a reachable space has its own copy of a skill, that copy wins over the built-in one.

Read the space's own `AGENTS.md` or `CLAUDE.md` first, when it has one.
