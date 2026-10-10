---
name: silverbullet-remote
description: Use when reading or editing a SilverBullet space with no accessible local copy of its files, including a localhost URL accessed only through the CLI.
---

# Remote SilverBullet spaces

When a task needs `sb`, check availability with `command -v sb`. If it is missing, install the edge CLI:

```sh
curl -fsSL https://silverbullet.md/install-sb.sh | sh -s -- --edge
```

Follow any PATH instructions printed by the installer, then verify with `sb --help` before continuing. Local file-only work does not require installation.

Check where `sb` executes before connecting. A remote HTTPS space needs permitted network access; a localhost space needs access to the host's loopback server. Determine access without invoking a runtime-backed `sb` command as a probe: a blocked sandbox or VM can make it launch another desktop process. In Codex, request outside-sandbox execution for each command that contacts a local space (`exec_command` with `sandbox_permissions: "require_escalated"`) or use an explicitly configured loopback permission. `sb --help` and `sb space ls` do not contact that server. In Cowork, use an available host-side connector; `127.0.0.1` inside its VM refers to the VM. If the execution environment cannot reach the selected space, do not run or retry those commands there; report the limitation rather than treating it as an empty space.

Confirm commands with `sb --help` and `sb fs --help`. Inspect `sb space ls` and select an explicit `--space`; alternatively use the exact user-provided space URL with `--url`. A localhost URL does not imply accessible local files. Keep secrets out of examples and output; prefer saved authentication. Use `sb space login <name>` when reauthentication is needed and let the user complete browser sign-in.

Read the space's operating manual through `sb fs read` and the shared [silverbullet-markdown skill](../silverbullet-markdown/SKILL.md). Paths are space-relative filenames, including `.md`; wiki links omit that suffix.

## Read and edit

```sh
sb --space notes fs ls --recursive --glob '*.md' --limit 100 --json
sb --space notes fs read 'Projects/Orchard.md' --json
sb --space notes fs edit 'Projects/Orchard.md' --old 'Status: draft' --new 'Status: ready' --dry-run
```

Read context before editing. Retain the inspected revision and pass it unchanged to `--if-match` when applying the edit. For a whole-file replacement use `fs write --if-match`; for a new file use `fs write --create`. See [file operations](references/files.md) for exact inputs, conflicts, and error handling.

Read back after writing. A temporary downloaded file is a working copy, not a synchronized space. Do not silently upload it with `--overwrite` after a conflict. A lost mutation response has an uncertain outcome: read the server state before retrying.

## Search and runtime

`fs ls --glob` filters filenames; it does not search content. Read a bounded set of relevant files, or use `sb query` for indexed questions when the runtime is available. Queries return indexed objects rather than arbitrary full-file grep matches.

File operations require HTTP access but no Lua runtime. They operate on saved server bytes, not unsaved editor buffers, and do not decrypt client-encrypted spaces. Do not invent a local path or require native opening. If the server or authentication is unavailable, report that limitation instead of treating it as an empty space.
